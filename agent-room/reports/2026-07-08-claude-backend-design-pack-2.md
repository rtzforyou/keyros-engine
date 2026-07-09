# Backend design pack 2 — NEXT.md tarefas 11–20

Task: NEXT.md (dabe10b) Claude tarefas 11–20 (1–10 entregues em reports anteriores)
Agent: Claude / Fable
Status: needs_review (design/planos; nada implementado nem aplicado)

## 11. Plan tiers canónicos: Solo, Team, Business, Enterprise

SUPERSEDE a secção 1 de 2026-07-08-claude-billing-entitlements-architecture.md
(founder/solo/studio/pro): os tiers canónicos passam a ser os definidos pelo
ChatGPT no NEXT.md. O conceito "founder" desaparece — o seu papel é coberto
pelo legacy-default `plan_id NULL` (secção 12). Restante arquitectura desse
report (schema, trigger de seats, resolução) continua válida.

Keys estáveis minúsculas; nomes de display separados. Nota de namespace: plan
key `team` e module id `team` vivem em espaços distintos (plans.key vs
ModuleId) e nunca se comparam entre si.

| key | módulos | team_seats | notas |
|---|---|---|---|
| `solo` | dashboard, crm, calendar, messages, finance, settings, billing | 1 | individual |
| `team` | solo + team, automations | 5 | estúdio pequeno |
| `business` | team + agents (+ reports quando activo) | 15 | estúdio grande |
| `enterprise` | todos os active | null (custom) | limites negociados |

Core sempre presente em todos: dashboard, settings, billing. Números de seats
e preços = decisão de Victor; a estrutura é o que fica fixo.

## 12. Comportamento default com plan_id NULL

`organizations.plan_id IS NULL` = **legacy-default**: todos os módulos
`active` + seats ilimitados = exactamente o comportamento actual. Todas as
orgs existentes ficam NULL no apply → zero impacto no dia 1. Novos signups
pós-billing: atribuição explícita de tier inicial (recomendação: `solo` com
trial — decisão de negócio de Victor). NULL nunca é erro; é um estado com
semântica definida.

## 13. Resolução de entitlements da organização

SQL (fonte única; usável por RLS futura, edge functions e frontend via RPC):

```sql
-- SECURITY INVOKER: RLS aplica (organizations própria + plans legível).
CREATE FUNCTION public.get_org_entitlements()
RETURNS jsonb LANGUAGE sql STABLE AS $$
  SELECT COALESCE(
    (SELECT p.entitlements
     FROM public.organizations o JOIN public.plans p ON p.id = o.plan_id
     WHERE o.id = public.get_user_org_id()),
    '{"modules": "__all_active__", "limits": {"team_seats": null}}'::jsonb
  );
$$;
```

O sentinel `__all_active__` é resolvido pelo consumidor contra o Module
Registry (a lista de active vive no código; o DB não conhece o registry —
ver 17). Frontend: `useOrgEntitlements` chama a RPC e materializa a lista.

## 14. Resolução de acesso efectivo do membro

```
effective(user):
  ent = get_org_entitlements()                # 13
  plan_modules = resolve(ent.modules)         # sentinel -> registry actives
  if user.role == 'admin': return plan_modules ∪ {billing}
  return normalizeModuleIds(user.permissions) ∩ plan_modules
```

Uma única implementação TS partilhada (registry + entitlements) usada por
Login enforcement, sidebar e guards de navegação. Versão SQL
(`get_effective_modules(uuid)`) só quando per-module RLS precisar dela —
não criar antes (YAGNI). Falha de resolução → negar não-core, erro visível.

## 15. Modelo de eventos de auditoria (Team + planos)

Tabela existente `public.audit_logs` (org_id, user_id, action, target_id,
details, created_at). Taxonomia de actions (estável, inglês, `dominio.evento`):

| action | details | writer |
|---|---|---|
| `team.member_access_changed` | {old:{role,permissions}, new} | frontend admin (PR #13, já implementado) |
| `team.member_invited` | {email, role, permissions} | frontend admin (slice futura) |
| `team.invite_revoked` / `team.invite_resent` | {invitation_id, email} | frontend admin |
| `team.member_removed` / `team.member_reactivated` | {old_status,new_status} | slice soft-remove |
| `billing.plan_changed` | {old_plan_key, new_plan_key, source} | service_role (backoffice/webhook) |
| `billing.seat_limit_hit` | {attempted_email} | edge function/trigger via service_role |

Regras: nunca guardar tokens/segredos em details; email só quando é o próprio
objecto do evento; INSERT por admins (policy 20260712000001) ou service_role;
sem purge por agora (volume baixo — rever com provider de billing).

## 16. Rollback plan — Team Permissions Editing (PR #13)

- Frontend: `git revert` do merge commit do PR #13 (UI volta a read-only;
  nenhum efeito em DB).
- Migração 20260712000001: nova migração que (a) restaura
  `protect_sensitive_user_columns()` com o corpo da 20260711000000 (remove o
  last-admin check), (b) `DROP POLICY "Admins can insert audit logs for their org"`.
  Nunca editar as migrações antigas.
- Migração 20260712000000 (normalização de dados): one-way por natureza, mas
  rollback é DESNECESSÁRIO — os aliases continuam legíveis por qualquer versão
  do frontend (normalizeModuleIds), e a UI antiga apenas não pré-selecciona
  checkboxes de ids novos. Documentado como irreversível-mas-inócuo.
- Dados criados entre apply e rollback (permissions editadas): permanecem
  válidos; guard antigo continua a protegê-los.

## 17. Registry: constantes vs tabela — checklist de migração

Decisão proposta: **constantes TS** (`lib/moduleRegistry.ts`) agora; NÃO criar
tabela `modules`. O DB guarda apenas arrays de ids (jsonb) e nunca precisa de
join com o registry (o sentinel da secção 13 mantém essa fronteira).

Migrar para tabela SÓ SE (gatilhos): per-module RLS precisar de join; ou
backoffice tiver de activar módulos sem deploy; ou módulos por-tenant.
Checklist para essa eventual migração: criar tabela `modules(id text pk,
status, team_assignable, plan_controlled)` seed a partir do registry → CHECK
constraints/FKs opcionais em permissions → gerar TS a partir do DB (inverter a
fonte) → um só dono da verdade, nunca dois.

## 18. Review das RLS afectadas por Team permissions

| Tabela | Policy | Veredicto |
|---|---|---|
| users | SELECT org-scoped / UPDATE own / UPDATE admin | OK (com guard 20260711000000; last-admin na 20260712000001) |
| audit_logs | SELECT admin-only; INSERT admin (20260712000001 draft) | OK após apply |
| invitations | "Tenant isolation" **FOR ALL a qualquer authenticated da org** | **CRÍTICO** — ver abaixo |
| organizations | "Admins can update their own organization" (sem restrição de coluna) | **ALTO (preventivo)** — quando `plan_id` existir, admin auto-upgrada o próprio plano. A migração de plans TEM de proteger a coluna (guard trigger ou revogar coluna na policy) |
| storage avatars | UPDATE/DELETE para qualquer authenticated no bucket inteiro | MÉDIO — membro pode substituir avatar de outrem; task isolada futura |

**CRÍTICO (novo achado, verificado em 20260620000000 §6):** a policy de
`invitations` é FOR ALL para qualquer membro autenticado da org — um membro
NÃO-admin pode criar um convite com `role='admin'` e permissions arbitrárias
(escalada indireta: convida um cúmplice/segunda conta própria como admin; o
signup do convidado cria admin de pleno direito). O guard de users não cobre
este caminho (é INSERT noutro fluxo). Fix proposto (migração draft na próxima
slice, junto com o apply do PR #13): dividir a policy — SELECT para membros da
org, INSERT/UPDATE/DELETE apenas admin (mesmo padrão do audit INSERT). O
trigger de seats (secção 3 do billing-architecture) fica no mesmo INSERT.

## 19. Matriz de testes (admin / membro / convidado)

Personas: A1 owner-admin · A2 segundo admin · M tattooer/staff · I convidado
(email com convite pendente, sem conta) · X utilizador de outra org · N anónimo.

| Acção | A1 | A2 | M | I | X | N |
|---|---|---|---|---|---|---|
| Ver membros da org | ✅ | ✅ | ✅ | — | ❌ (0 linhas) | ❌ |
| Editar próprio full_name/avatar | ✅ | ✅ | ✅ | — | — | ❌ |
| Editar próprias permissions/role | ✅* | ✅* | ❌ guard | — | — | ❌ |
| Editar role/permissions de membro | ✅ | ✅ | ❌ guard+RLS | — | ❌ | ❌ |
| Despromover último admin | ❌ last-admin | n/a (há 2) | ❌ | — | — | — |
| Criar convite | ✅ | ✅ | HOJE ✅ → deve ser ❌ (achado 18) | — | ❌ | ❌ |
| Convite com role admin | ✅ | ✅ | HOJE ✅ → ❌ | — | ❌ | ❌ |
| Revogar/reenviar convite | ✅ | ✅ | HOJE ✅ → ❌ | — | ❌ | ❌ |
| Ler convite por token | ✅ RPC | ✅ | ✅ | ✅ RPC | ✅ RPC (com token) | ✅ RPC |
| Aceitar convite (accept_invitation) | — | — | — | ✅ 1x (2ª vez ❌) | ❌ token inválido | ❌ not authenticated |
| Signup com convite pendente | — | — | — | ✅ entra na org convidada | — | — |
| Ver audit_logs | ✅ | ✅ | ❌ | — | ❌ | ❌ |
| Inserir audit_log forjado (outra org/outro user_id) | ❌ policy | ❌ | ❌ | — | ❌ | ❌ |

\* admin muda o próprio role só se existir outro admin (last-admin).
Execução: SQL com JWT simulado (padrão já usado) + browser real com contas de
teste (pedir a Victor 1 não-admin real — produção só tem admins).

## 20. Roadmap — próximas 3 slices backend

1. **Team Editing live**: merge PR #13 → apply 20260712000000/1 → migração
   nova `invitations admin-only policy` (fix do achado crítico da 18) → correr
   matriz da 19 → verificação visual. Gates: merge + 3 applies.
2. **Billing foundation**: migração plans/plan_id (tiers da 11, protecção da
   coluna plan_id da 18) + seat trigger + `get_org_entitlements` +
   `useOrgEntitlements`. Estados UI já preparados pelo Antenor. Gates: apply.
3. **Login enforcement + soft-remove**: resolução efectiva (14) aplicada a
   sidebar/router; migração `users.status` + fluxo revoke/reactivate com
   auditoria (15). Gates: apply + mudança de fronteira de acesso (rever com
   ChatGPT antes).

Depois disto: Dashboard backend Fase 1 (plano de 2026-07-08) e Stripe (EPIC
próprio, só com autorização explícita).
