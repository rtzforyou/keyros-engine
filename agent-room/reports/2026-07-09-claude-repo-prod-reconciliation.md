# Report — Repo↔Prod Reconciliation (camada de resiliência)

Agent: Claude / Fable
Date: 2026-07-09
Status: needs_review — reconciliação em PR draft; sem deploy/merge (gate)
Repo: rtzforyou/easytattoo-crm
Branch: claude/repo-prod-reconciliation · Commit: 6b1c1cb · PR: #29 (draft)

## Objetivo
Commitar a camada de resiliência JÁ DEPLOYED em produção para que um deploy
futuro de webhook/send a partir de main não a perca (regressão).

## Causa raiz do drift
Os ficheiros existiam committed em `7f0b59e^` e foram REMOVIDOS pelo commit
`7f0b59e` ("cleanup: remove backend and edge function files"). As funções/migrações
continuaram DEPLOYED/aplicadas em prod, mas a fonte saiu de main.

## Inventário (main antes vs produção)
- main JÁ tinha: `_shared/rateLimiter.ts`, `whatsapp-webhook` (== deployed v43,
  guards !fromMe + rate limiter), migrações `rate_limiting`/`rate_limit_review_fixes`.
- main FALTAVA (deployed em prod): `_shared/circuitBreaker.ts`,
  `automation-bulk-send` (v3), `automation-campaign-control` (v3), migrações
  `automation_bulk_campaigns`/`circuit_breaker_resilience`.
- main STALE: `whatsapp-send` (sem circuit breaker/rate limit) vs deployed v18
  (com). **Maior risco** — um deploy de send regrediria.

## Reconciliação (PR #29)
Recuperado de `7f0b59e^` (== deployed) e committed a main:
- `_shared/circuitBreaker.ts`, `automation-bulk-send/index.ts`,
  `automation-campaign-control/index.ts`, `whatsapp-send/index.ts` (atualizado),
  migrações `20260705000000`/`20260706000000` (já aplicadas; só histórico).
- docs/knowledge/26 atualizado (drift → resolvido).

## Validação contra produção (get_edge_function)
- whatsapp-send v18: anchors OK (org rate-limit key, evolution_api circuit,
  GATEWAY_UNREACHABLE, audit action) → recuperado == deployed.
- bulk/campaign: RATE_LIMIT_CONFIG.bulk_campaign_launch / campaign_control OK.
- circuitBreaker: safeApiCall + circuit_breaker_state + system_health_logs OK.
- whatsapp-webhook de main já == deployed v43 (sem alteração).
- Diferença cosmética: o bundle deployed tem comentários minificados; a fonte
  committed mantém os comentários. Comportamento idêntico.

## Fora de âmbito (WIP não deployed ao frontend de prod)
`lib/resilience/circuitBreaker.ts`, `useSystemHealth`, `SystemHealthSettings`,
`useCampaigns`, `CampaignProgressPanel` — o frontend de prod (main) não os tem;
são WIP do Antenor.

## Verificação
Edge functions são Deno (fora do vite). Frontend tsc limpo + build ✓.

## Riscos / não verificado
- Não redeployei nada (gate). Após merge do #29, main == prod na resiliência
  backend; deploys futuros seguros.
- Migrações 05/06 têm timestamp de ficheiro diferente do registo remoto
  (20260705095211/103118) — normal (MCP); já aplicadas, sem re-run.

## Próximo passo
Merge do #29 (gate). Depois: main==prod backend → arrancar Fase 3 com segurança.
Frontend system-health: Antenor commita quando quiser deployá-lo.
