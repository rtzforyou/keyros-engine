# Report — Repository Stabilization (missão excecional)

Agent: Claude / Fable
Date: 2026-07-09
Status: completed — sem merge/deploy/apply (todos os gates respeitados)
Objetivo: deixar o repo estável para amanhã continuar a Fase 3.

## Repository Health

- **main NÃO compilava** (2 erros). Causa raiz + reparação mínima em **STEP 1**
  (PR #28). Com o PR #28 aplicado, `tsc` limpo e `npm run build` ✓.
- Causa raiz do build partido:
  1. **useWhatsAppAvailability — hook nunca committed.** O commit `15fe75a`
     adicionou o *import* em `MessagesContent` (chegou a main via merge do
     PR #15/`39dbb7d`), mas o ficheiro do hook ficou untracked no working tree.
  2. **ContactFormsTab — import órfão** em `ContactDetailsDrawer`, deixado quando
     os ficheiros da feature Forms foram removidos (`e2c4dea`).
- Reparação (PR #28, frontend mínimo): commitar o hook em falta (read-only, só
  lê `circuit_breaker_state` que existe em prod) + remover a aba/import órfãos.
- **Drift repo↔prod (importante):** vários artefactos estão APLICADOS/DEPLOYED em
  produção mas os ficheiros NÃO estão em main (untracked no working tree do
  Victor): migrações de resiliência (bulk campaigns, circuit breaker, rate
  limiting), o hook/circuit-breaker/edge functions da camada de resiliência, e
  os hooks/componentes de system health. Ver EPIC 3 report.

## Open PRs — veredictos (STEP 3)

Produto:
| PR | Título | Estado | Veredicto |
|---|---|---|---|
| #28 | fix(build) reconciliação de main | MERGEABLE/CLEAN | **READY — merge PRIMEIRO** (desbloqueia o repo) |
| #17 | group identity consistency | MERGEABLE/CLEAN | **READY** — commita migração 000004 (já aplicada em prod) + comentário em conversationIdentity.ts; isolado |
| #25 | invitation audit logging (DB-side) | MERGEABLE/UNSTABLE | **READY** (só ficheiros SQL/teste). Migração NÃO aplicada — apply é gate separado. UNSTABLE = check Cloudflare, não conflito |
| #27 | Translation Architecture (i18n) | UNKNOWN (base main partida) | **READY após #28** — sem conflito de conteúdo com main; fica verde quando #28 estiver em main |
| #26 | Antenor frontend review (#18/#20-23 + safe #24) | **CONFLICTING/DIRTY** | **BLOCKED** — base loop4 divergiu de main; sobrepõe #28 (useWhatsAppAvailability/ContactFormsTab) e #27 (DashboardContent). Rebase onto main pós-#28, remover as partes já cobertas, re-rever. Domínio Antenor |

Engine (só reports, MERGEABLE): #1, #3, #4, #5, #6, #7 — READY (documentação).

## Merge Order recomendada
1. **#28** (build fix) — primeiro, torna main saudável.
2. **#17** (group identity) — reconcilia migração 000004 já aplicada.
3. **#25** (invitation audit — ficheiros) — depois aplicar a migração (gate).
4. **#27** (i18n) — após #28 (build verde).
5. **#26** — só depois de rebase (BLOCKED até lá).
(#17/#25/#27 não conflitam entre si — ordem flexível após #28.)

## Migration Review + Order (STEP 4)

Estado em produção (verificado via MCP):
- **Aplicadas** (recentes): automation_bulk_campaigns, circuit_breaker_resilience,
  rate_limiting, rate_limit_review_fixes, secure_appointments_calendar_view,
  real_invite_acceptance_flow, team_users_columns_and_signup,
  normalize_legacy_module_permissions, team_permissions_editing_support,
  invitations_admin_only_write, repair_whatsapp_identity_poisoning,
  repair_group_identity_participant_names.
- **NÃO aplicada:** `20260713000000_audit_invitation_changes` (#25).

Migrações **pendentes de commit** (ficheiro fora de main):
- `20260712000004_repair_group_identity_participant_names` (#17) — JÁ APLICADA
  em prod (remoto `20260708205502`). Merge do #17 só regista o ficheiro. **Risco
  zero, sem apply.**
- Ficheiros de resiliência (bulk/circuit/rate) — JÁ APLICADOS em prod, ausentes
  de main. Reconciliar (commitar) sem apply.

Migração pendente de **apply** (única):
- `20260713000000_audit_invitation_changes` — trigger em `invitations` →
  `audit_logs`. Dependências: `audit_logs` + `invitations` (existem há muito).
  **Ordem:** independente; pode aplicar a qualquer momento após merge do #25.
  **Risco de produção:** baixo — não-destrutivo (CREATE FUNCTION + 2 triggers
  AFTER; SECURITY DEFINER). Não altera dados existentes.
  **Rollback:** `DROP TRIGGER audit_invitation_insert_trg/update_trg` +
  `DROP FUNCTION audit_invitation_change()` (via migração nova).
  Teste pós-apply: `supabase/tests/audit_invitation_changes_test.sql`.

**Ordem de execução recomendada (migrações):**
1. Reconciliar ficheiros já aplicados (commit dos 000004 + resiliência) — sem apply.
2. Aplicar `20260713000000` (gate) + correr o teste.

## Known Risks
- **Repo↔prod drift** (o maior): edge functions de resiliência (safeApiCall no
  whatsapp-send/bulk/campaign), circuit breaker e migrações estão DEPLOYED mas
  não committed. Um deploy futuro do webhook/send a partir de main PERDE-os.
  Reconciliar antes de qualquer deploy.
- **#26 conflitos**: precisa de rebase; risco de re-introduzir bugs se mal feito.
- Aplicar a migração de auditoria de convites é a única mutação de produção
  pendente (gate).

## Remaining Technical Debt
- Reconciliar repo↔prod (resiliência deployed-uncommitted).
- #26 rebase (componentes partilhados LoadingSpinner/EmptyState, fix do filtro de
  grupos por case em ConversationList — ainda valioso, fora do #28).
- Dashboard client-side aggregation (escala) — plano no roadmap.

## Remaining Architecture Debt
- Entitlements (`lib/entitlements.ts`) não wired (Planned, Fase 4/Billing).
- Vocabulário frontend App/Sidebar ainda com aliases legados (funcionalmente
  correto via mapa inline; alinhamento total = follow-up).
- Sem runner de testes no repo (só scripts SQL de RLS).

## Remaining Translation Debt
- Dashboard principal migrado (#27). Subcomponentes ainda hardcoded:
  WhatsAppAutomationStats, LeadSourceChart, RevenueExpenseChart, ExpensesChart,
  UpcomingAppointments, RecentActivity, PipelineChart. Usar `scripts/i18n-audit.sh`.

## Security Status
- **Forte no backend (em main + aplicado):** RLS por org, guard de auto-escalada
  + last-admin, invitations admin-only, audit_logs admin-only. Verificado por
  testes RLS (JWT simulado).
- **Gap conhecido (mitigável):** criar/revogar/reenviar convite ainda não
  auditado — fechado por #25 (pendente de merge+apply).
- Nada de RLS/auth/billing tocado nesta missão.

## Platform Status
- Module Registry consistente (11 keys canónicas). Rate limit / circuit breaker /
  idempotência: implementados e DEPLOYED, mas não committed (drift).
- main saudável assim que #28 for merged.

## Product Status
- Team real (read-only em prod) + permissões (migrações aplicadas). WhatsApp
  identity (privado + grupo) corrigido e aplicado. Dashboard real-data + i18n
  (pendente merge #27). Billing = UI draft / entitlements não wired (Fase 4).

## Recomendação para amanhã de manhã
1. **main está saudável?** Sim, assim que **#28** for merged (é o primeiro).
2. **Que PR merge primeiro?** **#28**. Depois #17, #25, #27 (após #28). #26 fica
   para rebase.
3. **Que migrações são seguras?** 000004 e as de resiliência = já aplicadas
   (só commit). `20260713000000` = segura para aplicar (não-destrutiva) sob gate.
4. **O que ainda bloqueia produção?** O drift repo↔prod da camada de resiliência
   (deployed-uncommitted) — reconciliar antes de qualquer deploy futuro.
5. **Próximo EPIC recomendado:** **Repo↔Prod Reconciliation** (commitar resiliência
   deployed + rebase do #26), e depois arrancar a Fase 3 (Product Core) com main
   limpo. Aplicar #25 (auditoria de convites) no mesmo ciclo de gate.

## Gates respeitados
Nenhum merge, deploy, apply de migração, mutação de produção, billing, stripe,
RLS ou auth. Todas as reparações em PR draft.
