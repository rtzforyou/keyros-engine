# Report — EPIC 3 Resilience Layer (architecture)

Task: EPIC 3 — Resilience Layer (20 tarefas)
Agent: Claude / Fable
Status: BLOCKED (cross-agent conflict + major finding) — parado antes de
implementar/commitar. Nenhum código de produto alterado.
Repo: rtzforyou/easytattoo-crm (auditado main @ 39dbb7d + produção + working tree)
Branch (report): keyros-engine `claude/epic3-resilience-layer`

## ACHADO PRINCIPAL — a camada de resiliência já existe e está DEPLOYED

A EPIC 3 pede para "completar a fundação de resiliência". A auditoria mostra que
essa fundação **já está implementada e a correr em PRODUÇÃO**, mas o código-fonte
**não está commitado** no repositório — vive apenas no working tree local
(atualmente no branch `antenor/frontend-consistency-module-registry-alignment`,
ficheiros untracked/modified).

Evidência:
- **Migrações aplicadas em produção:** `circuit_breaker_resilience`,
  `automation_bulk_campaigns`, `rate_limiting`, `rate_limit_review_fixes`.
  Tabelas `circuit_breaker_state`, `system_health_logs`, `rate_limits` existem.
- **Edge Functions deployed e ACTIVE:** `automation-bulk-send` v3,
  `automation-campaign-control` v3, `whatsapp-send` v18 (atualizado recentemente),
  `whatsapp-webhook` v43.
- **Código local (untracked/modified, NÃO em main):**
  `supabase/functions/_shared/circuitBreaker.ts` (safeApiCall: timeout, threshold,
  half-open, log estruturado em system_health_logs), `_shared/rateLimiter.ts`,
  `lib/resilience/circuitBreaker.ts` (frontend), `automation-bulk-send/`,
  `automation-campaign-control/`, `whatsapp-send/index.ts` modificado (importa
  safeApiCall + checkRateLimit), `hooks/useSystemHealth.ts`,
  `components/settings/SystemHealthSettings.tsx`, `hooks/useCampaigns.ts`,
  migrações `20260705000000_*`, `20260706000000_*`.

**Conclusão:** reimplementar qualquer uma das 20 tarefas do zero seria (a)
duplicar código já deployed, (b) diverge do que está em produção, (c) conflito
cross-agent com o working tree do Antenor. O trabalho real da EPIC 3 é
**reconciliação repo↔prod** (commitar o que já está live), não implementação.

## Inventário — 20 tarefas vs realidade

| # | Tarefa | Estado |
|---|---|---|
| 1 | Rate limit em whatsapp-send | ✅ FEITO+DEPLOYED (whatsapp-send v18 importa `checkRateLimit`; substitui a contagem inline antiga). NÃO commitado. |
| 2 | Rate limit em automation-bulk-send | ✅ FEITO+DEPLOYED (v3, usa `checkRateLimit` config `bulk_campaign_launch`). NÃO commitado. |
| 3 | Rate limit em automation-campaign-control | ✅ FEITO+DEPLOYED (v3, `checkRateLimit` config `campaign_control`). NÃO commitado. |
| 4 | Usar o helper centralizado nos três | ✅ os três importam `_shared/rateLimiter.ts`. |
| 5 | Auditoria de criação de convite | ❌ GAP REAL — `useInvitations.sendInvitation` não escreve audit_logs. |
| 6 | Auditoria de reenvio de convite | ❌ GAP REAL — idem `resendInvitation`. |
| 7 | Auditoria de revogação de convite | ❌ GAP REAL — idem `revokeInvitation`. |
| 8 | Payload de auditoria padronizado | ⚠️ PARCIAL — taxonomia `dominio.evento` desenhada (backend-design-pack-2 §15); só `team.member_access_changed` implementado. |
| 9 | Timeout no webhook | ✅ `safeApiCall` tem `AbortController`+timeout (default 10s). Deployed. |
| 10 | Retry policy reutilizável | ⚠️ PARCIAL — half-open/reset do circuit breaker + `automation-retry` (v7) cobrem retry; não há um helper `withRetry` genérico separado. |
| 11 | Circuit Breaker v1 partilhado | ✅ FEITO+DEPLOYED (`_shared/circuitBreaker.ts` + frontend). NÃO commitado. |
| 12 | Health checks de integrações | ✅ `system_health_logs` + `circuit_breaker_state` + `useSystemHealth` + painel SystemHealthSettings. NÃO commitado. |
| 13 | Log de erro estruturado | ✅ `logAttempt` grava status/tempo/erro/estado em system_health_logs (JSON). |
| 14 | Metrics hooks | ⚠️ PARCIAL — system_health_logs serve de base de métricas; sem exportador de métricas dedicado. |
| 15 | Idempotência em fluxos outbound | ✅ `automation_executions` como fila (scheduled_for/status/idempotency_key); `automation-execute` bloqueia duplicados; webhook onConflict message_id. |
| 16 | Atualizar KB | ⚠️ a fazer — a KB (pós-EPIC 2) diz "circuit breaker Planned / não em main"; correto para MAIN mas impreciso para PRODUÇÃO. A corrigir para "deployed mas não commitado". |
| 17 | Atualizar CLAUDE.md | ✅ já correto — CLAUDE.md refere `lib/resilience/circuitBreaker.ts` e `_shared/circuitBreaker.ts`, que existem (deployed/uncommitted). Sem mudança de guidance. |
| 18 | Integration tests | ❌ GAP — sem runner de testes no repo (package.json só dev/build/preview). Planned. |
| 19 | Relatório de arquitetura | ✅ este documento. |
| 20 | PR draft | ✅ report + correção KB (sem código de produto). |

Resumo: **13/20 já feitas e DEPLOYED** (1-4, 9, 11-13, 15, e parciais 8/10/14),
**mas não commitadas**. Gaps reais: auditoria de convites (5-7-8), retry helper
genérico (10), metrics exporter (14), integration tests (18).

## Gate atingido (STOP)

Isto é simultaneamente **cross-agent conflict** e **major architecture decision**
(GATES.md). Não posso:
- Reimplementar (duplica/diverge do deployed).
- Commitar os ficheiros do working tree do Antenor (não é o meu checkout; é
  trabalho ativo de outro agente).

## Recomendação (para Victor/ChatGPT decidir)

1. **Reconciliação repo↔prod (prioridade máxima):** commitar a camada de
   resiliência JÁ DEPLOYED para `main` (circuit breaker edge+frontend,
   rateLimiter, whatsapp-send/bulk/campaign, migrações 20260705/06, system health).
   Decisão: quem commita (é backend = domínio Claude, mas está no checkout do
   Antenor). Recomendo: Claude commita os ficheiros backend/migrações; Antenor
   commita os componentes/hook de UI (SystemHealthSettings, useSystemHealth,
   useCampaigns, CampaignProgressPanel). Sem novo deploy (já está live) — só
   alinhar o repo com produção.
2. **Fechar gaps reais (slice Claude, após reconciliação):** auditoria de
   convites (create/resend/revoke) com payload padronizado; opcionalmente helper
   `withRetry` genérico. Auditoria de convites toca `useInvitations` (frontend);
   alternativa mais limpa: auditar no lado do servidor.
3. **Testes/metrics (Planned):** adicionar runner de testes e exportador de
   métricas são EPICs próprios.

## Riscos / não verificado
- Auditoria estática + estado de produção (MCP). Não corri o app.
- Não confirmei byte-a-byte que o deployed whatsapp-send v18 == ficheiro local
  modificado; a versão/timestamps e os imports indicam que sim (confirmar na
  reconciliação).
- Não commitei nem toquei em ficheiros do working tree do Antenor.

## Próximo passo
Aguardar decisão de gate sobre a reconciliação repo↔prod (quem commita o quê).
Nada de merge/deploy/migração feito.
