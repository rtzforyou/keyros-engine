# Proposta — Canonical Module Registry (v1)

Task: EPIC Canonical Module Registry (UPDATE_TASK.md, tarefas 1–8)
Agent: Claude / Fable
Status: needs_review (proposta + planos; nada implementado, Stripe não tocado, Payments não tocado)
Repo alvo da implementação futura: rtzforyou/easytattoo-crm (`lib/moduleRegistry.ts`)

## 1. Princípios

- O id do módulo é ESTÁVEL, em inglês, e nunca muda depois de publicado. Toda a lógica (permissions, entitlements, feature flags) decide por id, nunca por label.
- Labels de display vivem em `lib/translations.ts` (`t.module_<id>`), separados dos ids — 3 idiomas.
- Uma única fonte: `lib/moduleRegistry.ts` no produto. Convites, edição de permissões, enforcement de login, sidebar e (futuro) entitlements de plano importam daqui. Proibido inventar ids ad hoc (regra do UPDATE_TASK.md).

## 2. Refinamentos ao draft do ChatGPT (reportados conforme pedido)

| Draft ChatGPT | Proposta Claude | Porquê |
|---|---|---|
| `crm` | **REMOVER** | Redundante: `contacts` + `deals` já cobrem o CRM. Um id deve gatear exactamente uma superfície; agrupamento visual é preocupação de UI, não de id. Dois ids a gatear a mesma coisa cria ambiguidade em permissions/entitlements. |
| (ausente) | **ADICIONAR `expenses`** | Existe hoje como módulo real (aba própria, dados reais via useFinances, checkbox próprio no convite). Removê-lo do registry quebraria convites existentes. Alternativa se ChatGPT preferir consolidar: fold em `payments` com migração de dados — decisão para Victor/ChatGPT. |
| `reports` | Manter como **`reserved`** | Não existe superfície "Reports" hoje (KPIs vivem no dashboard). Reservar o id já, mas não assignable até existir. |
| `ai` | Manter (rename de `aiAgent`) | Alias legado: `aiAgent` → `ai`. |
| `deals` | Manter (rename de `pipeline`) | Alias legado: `pipeline` → `deals`. Verificado em produção: 1 convite tem `pipeline` em `invitations.permissions` — precisa de migração de dados (secção 5). |

## 3. Registry canónico v1

| id | labelKey | status | teamAssignable | planControlled | dependsOn |
|---|---|---|---|---|---|
| `dashboard` | module_dashboard | active | sim | não (core) | — |
| `contacts` | module_contacts | active | sim | sim | — |
| `deals` | module_deals | active | sim | sim | contacts |
| `calendar` | module_calendar | active | sim | sim | — |
| `messages` | module_messages | active | sim | sim | contacts |
| `automations` | module_automations | active | sim | sim | messages |
| `ai` | module_ai | active | sim | sim | messages |
| `payments` | module_payments | active | sim | sim | contacts |
| `expenses` | module_expenses | active | sim | sim | — |
| `reports` | module_reports | **reserved** | não (ainda) | sim | dashboard |
| `settings` | module_settings | active | sim (com aviso) | não (core) | — |
| `team` | module_team | active | sim (com aviso) | sim (via seat limits) | — |

Notas:
- `dashboard` e `settings` são core: nunca desligados por plano (uma org paga sempre consegue entrar e configurar). `settings`/`team` como permission de membro staff exigem aviso na UI (dão poder de configuração; admin já tem tudo por role).
- `dependsOn` é soft: a UI avisa/auto-seleciona a dependência ao atribuir permission ou montar plano; o enforcement não bloqueia (um membro pode ter `deals` sem `contacts` — vê deals, não abre a lista de contactos).
- Admin: acesso total por ROLE, ignora a lista (invariante já garantida no DB pelo guard do PR #9).

### Shape TypeScript (implementação futura)

```ts
export type ModuleId = 'dashboard' | 'contacts' | 'deals' | 'calendar' | 'messages'
  | 'automations' | 'ai' | 'payments' | 'expenses' | 'reports' | 'settings' | 'team';

export interface ModuleDefinition {
  id: ModuleId;
  labelKey: keyof Translations;   // display separado do id
  status: 'active' | 'reserved';
  teamAssignable: boolean;
  planControlled: boolean;
  dependsOn?: ModuleId[];
}

export const MODULE_REGISTRY: ModuleDefinition[] = [ /* tabela acima */ ];

// Ids antigos aceites em leitura, nunca escritos de novo:
export const LEGACY_MODULE_ALIASES: Record<string, ModuleId> = {
  pipeline: 'deals',
  aiAgent: 'ai',
};
export const normalizeModuleIds = (ids: string[]): ModuleId[] => /* map + filter por registry */;
```

## 4. Representação em banco

- `users.permissions` e `invitations.permissions`: continuam `jsonb` array de ids canónicos (sem tabela nova — volume baixo, RLS já cobre).
- Futuro (Billing): tabela `plans` (`id`, `name`, `entitlements jsonb`: `{ "modules": ModuleId[], "limits": { "team_seats": number } }`) + `organizations.plan_id` FK nullable. `plan_id NULL` = plano default com todos os módulos = comportamento actual preservado (zero mudança de produção no dia 1).

## 5. Plano de implementação — Team Permissions Editing (slice seguinte)

1. **Migração de dados** (pequena): normalizar ids legados em `invitations.permissions` (`pipeline`→`deals`, `aiAgent`→`ai`). Verificado: 1 convite afectado. `users.permissions` está limpo (todos `[]`).
2. `lib/moduleRegistry.ts` + chaves `module_*` em translations (3 idiomas).
3. TeamContent: dialog de convite passa a usar o registry (substitui o array `MODULES` hardcoded); leitura de permissions com `normalizeModuleIds`.
4. `useTeamMembers.updateTeamMember(id, { role, permissions })` — validação de ids contra registry; visível só para admin (role vem do próprio perfil já resolvido pelo hook). Defesa em 3 camadas: UI + RLS admin-update + trigger guard (já em produção).
5. Restaurar o dialog "Editar Acesso" (JSX está no histórico git do PR #12) ligado ao hook.
6. **Auditoria**: INSERT em `public.audit_logs` (tabela existente, RLS admin-only já configurada) — action `team.permissions_changed` / `team.role_changed`, `target_id` = user id, `details` = `{old, new}`.
7. **Last-admin protection** (migração): estender `protect_sensitive_user_columns()` para bloquear despromoção do último admin de uma org (contagem dentro da função; SECURITY DEFINER já lê sem recursão RLS).
8. Testes: repetir bateria do guard + editar permissions de membro como admin via UI; multi-tenant; refresh.

Gates: passo 1 e 7 são migrações → apply exige autorização; resto é branch/PR draft dentro da minha autonomia.

## 6. Plano de implementação — Billing Entitlements + Team Seat Limits (sem Stripe)

1. Migração: tabela `plans` + `organizations.plan_id` (nullable, default NULL = tudo activo). Seed: `free`/`pro` provisórios com entitlements draft — valores de negócio são decisão de Victor.
2. `useOrgEntitlements` hook (org-scoped): resolve plano da org, devolve `{ modules, limits, isLoading, error }`; NULL → default-tudo.
3. **Resolução de acesso efectivo** (fórmula única, também usada no Login enforcement):
   - admin → `plan.modules` (tudo o que o plano dá)
   - não-admin → `user.permissions ∩ plan.modules`
4. **Seat limit enforcement** — ponto único no servidor: criação de convite valida `(membros activos + convites pending) < limits.team_seats`. DB-side (função/trigger em `invitations` INSERT) para não depender do frontend. UI mostra estados do Antenor (seat usage, invite blocked, upgrade CTA).
5. Downgrade de plano: módulos fora do novo plano ficam inacessíveis mas permissions não são apagadas (reversível ao re-upgrade). Nunca deletar dados por downgrade.
6. Stripe (futuro, fora deste plano): subscription webhook → actualiza `organizations.plan_id`. Nada mais muda — o enforcement já está todo atrás do `plan_id`.

Não implementado agora: nenhuma destas migrações/hoocks foi criada. Payments produção intocado.

## 7. Decisões pedidas a Victor/ChatGPT

1. Aprovar remoção de `crm` do draft (recomendado) ou definir o que `crm` gateia.
2. `expenses`: manter como módulo próprio (recomendado, é o estado actual) ou consolidar em `payments`.
3. Valores de negócio dos planos (nomes, módulos incluídos, team_seats) — para o seed do passo 6.1.
4. Autorização para iniciar a implementação da slice Team Permissions Editing (secção 5) como branch/PR draft.
