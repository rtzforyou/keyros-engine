# Report — EPIC 2 Platform Consolidation (Claude queue audit)

Task: Auditoria de consolidação de plataforma (NEXT_ACTIONS Claude queue, 16 itens)
Agent: Claude / Fable
Status: needs_review — auditoria completa + correções de docs (seguras); tudo o
que muta backend/prod fica como finding (gate). Nenhum código de produto alterado.
Repo (docs): rtzforyou/easytattoo-crm, branch `claude/keyros-knowledge-base-v1`
Commit: 3ec8daf (correções de docs KB, sobre PR #19)
PR: docs em #19; sem PR de código novo (só findings)
Base auditada: `origin/main` @ 39dbb7d (clone raso).

## Método
Auditoria estática (grep/leitura) sobre main. Correções só onde é seguro (docs).
Todo fix que envolva deploy, migração, dados de produção ou fronteira de acesso
fica documentado como finding — respeitando os gates.

## Resultados por item da queue

1. **Ler roadmap/state/gates/role/docs** — feito.
2. **Module Registry keys canónicas** — ✅ CONSISTENTE. `lib/moduleRegistry.ts`
   tem exatamente as 11 keys aprovadas (dashboard, crm, calendar, messages,
   automations, team, billing, finance, reports, settings, agents).
3. **Aliases legados no backend/shared** — o registry trata via
   `LEGACY_MODULE_ALIASES` + `normalizeModuleIds` (contacts/deals/pipeline→crm,
   payments/expenses→finance, ai/aiAgent→agents). Backend: convites/permissões
   normalizam na escrita (`useTeamMembers`) e leitura (`Login`). Dados legados em
   `users.permissions`/`invitations.permissions` já foram normalizados pela
   migração `20260712000000` (em main + aplicada). ✅
4. **Permission checks usam keys do registry** — ⚠️ FINDING (frontend/Antenor):
   `App.tsx.hasPermission` reimplementa o mapa de aliases inline (em vez de
   importar `LEGACY_MODULE_ALIASES`); `Sidebar.tsx` usa `contacts/pipeline/
   payments/expenses` como keys de navegação/permissão; `App.tsx` e `Login.tsx`
   têm listas `ALL_MODULES` divergentes hardcoded (App=legado, Login=canónico).
   **Não é bug funcional** (o mapa converte; o Login normaliza as permissões
   guardadas), mas é drift de vocabulário. Correção pertence ao Antenor:
   importar do `moduleRegistry` e apagar o mapa/listas hardcoded.
5. **Entitlements centralizados** — ⚠️ FINDING/Planned: `lib/entitlements.ts`
   (`resolveOrgEntitlements`, `resolveEffectiveModules`, `hasModuleAccess`)
   existe mas **não tem consumidor** (não é importado). Sem tabela `plans`/
   `plan_id`. `useBillingLimits` usa `seatLimit` hardcoded (rascunho de UI). O
   enforcement de entitlements é **Planned** (Fase 4) — coerente com o roadmap.
6. **Admin-only protegido** — ✅ FORTE (backend, em main + aplicado):
   - Convites: escrita admin-only (`20260712000002`); leitura pública fechada.
   - `users`: guard `protect_sensitive_user_columns` (não-admin não auto-escala;
     last-admin protection) (`20260712000001`).
   - `audit_logs`: INSERT/SELECT admin-only.
   - Team page: admin-only na UI; billing core admin-only por design.
7. **Fluxos RLS-sensíveis** — revistos: `users`, `invitations`, `audit_logs`,
   `organizations`. Sem regressão. (Nota preventiva já registada: quando
   `organizations.plan_id` existir, a policy "Admins can update their own
   organization" precisa de proteger a coluna — ver backend-design-pack-2 §18.)
8. **Sem bypass frontend do backend** — ✅ `hasPermission` é UX; a proteção real
   é RLS + guard nas escritas. Permissão por membro é enforced no banco.
9. **Cobertura de auditoria** — ⚠️ FINDING/Planned: auditado
   `team.member_access_changed` (`useTeamMembers`) e um registo em `whatsapp-send`.
   **Gap:** criar/revogar/reenviar convite (`useInvitations`) NÃO é auditado. A
   taxonomia (team.member_invited/invite_revoked/…) está desenhada
   (backend-design-pack-2 §15) mas não implementada.
10. **Rate limit em Edge Functions abuso-prone** — ⚠️ FINDING/Planned: ativo só em
    `whatsapp-webhook` e `landing-lead`. `whatsapp-send`, `automation-bulk-send`,
    `automation-campaign-control` têm config em `RATE_LIMIT_CONFIG`
    (`whatsapp_send`, `bulk_campaign_launch`, `campaign_control`) mas **não
    chamam `checkRateLimit`** — os paths de envio em massa estão sem rate limit.
    Prioridade recomendada (envio em massa é o mais abuso-prone). Fix = deploy (gate).
11. **Idempotência/retry/circuit breaker** — idempotência ✅ (webhook `onConflict`
    em `message_id`; `automation-execute` `idempotency_key` →
    `duplicate_execution_blocked`). **Circuit breaker: NÃO existe em main**
    (`_shared/` só tem `rateLimiter.ts`; `lib/resilience/` ausente) apesar de
    CLAUDE.md e a KB o referirem → **doc divergence**. Retry existe em pontos
    específicos (reconexão silenciosa do webhook).
12. **Tests** — o repo não tem runner de testes configurado (package.json só
    dev/build/preview). Adicionar framework é scope creep/fora de gate — deixado
    como recomendação (Planned). Os testes SQL de RLS existem como scripts
    (`supabase/tests/team_invitations_rls_test.sql`).
13. **Correção de docs KB** — ✅ feito (commit 3ec8daf, sobre PR #19): docs
    03/12/14/16/19/26/28 corrigidos para marcar circuit breaker=Planned, rate
    limit parcial, auditoria parcial, entitlements não wired; findings agregados
    em `26-known-limitations.md`.
14. **Report** — este ficheiro.
15. **PR draft** — docs no PR #19 (já aberto); sem PR de código.
16. **Parar nos gates** — respeitado (nada de merge/deploy/migração/dados/auth).

## Findings priorizados (para decisão de gate)
| # | Severidade | Finding | Dono | Fix (gate) |
|---|---|---|---|---|
| A | Médio-Alto | Rate limit ausente em send/bulk/campaign | Claude | deploy |
| B | Médio | Convites não auditados | Claude | deploy (edge) |
| C | Médio | Circuit breaker inexistente (doc dizia que existia) | Claude | doc ✅ + impl Planned |
| D | Baixo-Médio | Vocabulário frontend legado (App/Sidebar/Login) | Antenor | frontend PR |
| E | Baixo | Entitlements não wired (esperado, Planned) | Claude | Fase 4 |
| F | Baixo | 20260712000004 aplicada mas ficheiro só no PR #17 | ChatGPT | merge #17 |

## Correções aplicadas (sem gate)
Só docs KB (não-gated), no branch do PR #19.

## Riscos / não verificado
- Auditoria estática; não corri o app. Não apliquei nada em backend/prod.
- CLAUDE.md (raiz do repo do produto) também refere o circuit breaker; não editei
  (ficheiro de instruções partilhado) — flag para ChatGPT alinhar.

## Próximo EPIC recomendado
Fechar os gaps de resiliência/auditoria como slice backend (rate limit em
send/bulk/campaign + auditoria de convites), sob gate de deploy; em paralelo,
Antenor alinha o vocabulário frontend com o Module Registry (finding D).
