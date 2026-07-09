# Auditoria — Product Core Backend (Fase 3)

Autor: Claude / Fable · Data: 2026-07-09 · Âmbito: arquitetura backend do Product Core
Método: leitura da fonte real (repo `origin/main` + deployed via `get_edge_function`)
e do estado de RLS/políticas em produção (read-only). Sem deploy, sem mutação.

Base: 14 edge functions no repo + `_shared/` (circuitBreaker, rateLimiter), 9 serviços
`lib/`, ~22 data hooks (repository layer), KB `docs/knowledge/`.

---

## Sumário executivo (prioridade)

| # | Achado | Sev. | Estado |
|---|--------|------|--------|
| 1 | Drift repo↔prod: `automation-execute`+`scheduler`+`retry` sem circuit breaker no repo + `landing-lead` (público) com rate limiter em memória vs DB-backed em prod | ALTO | **CORRIGIDO** — PR #36 (repo←prod, 4 funções) |
| 1b | `send-whatsapp` = função legada do provider antigo **Green API** (`GREENAPI_*`, formato `@c.us`), substituída por `whatsapp-send` (Evolution) — dead code | BAIXO | ADR-0002 cleanup |
| 2 | Defesa-em-profundidade: hooks fazem read/mutação sem `.eq('organization_id')` explícito | MÉDIO | documentado; fix precisa coordenação (Antenor) |
| 3 | Duplicação: template engine inline em scheduler+retry(+execute); `interpolate` próprio no engine | MÉDIO | ADR proposto (deploy-coupled) |
| 4 | Duplicação: helpers de identidade WhatsApp (normalizeJid/getPhone/normalizePhone) em ≥4 funções | MÉDIO | ADR proposto (deploy-coupled) |
| 5 | Funções legadas/perigosas (debug trio, sync-history, lead-intake, automation-engine, setup-org-storage) | ALTO→vário | ADR-0002 em curso (PRs #33/#34) |
| 6 | Sobreposição/naming: `whatsapp-send` vs `send-whatsapp`; `whatsapp-webhook` vs `evolution-webhook` vs `twilio-webhook` | BAIXO | classificar/renomear (cleanup) |
| 7 | Observabilidade inconsistente: logs estruturados nalgumas funções, strings livres noutras | BAIXO | helper `_shared/log.ts` proposto |
| 8 | Sem test runner no repo; cobertura de testes ~nula fora de SQL pontual | MÉDIO | plano de testes proposto |
| 9 | `usePipelineStages` não está em `origin/main` (só em branch) | MÉDIO | verificar fonte de stages em main |

---

## Por domínio

### Automation Engine
- **Executores múltiplos e sobrepostos:** `automation-execute` (729 linhas, verify_jwt=true,
  executor principal chamado pelo webhook/send/landing), `automation-scheduler` (cron/minuto),
  `automation-retry` (retry manual autenticado), `automation-engine` (fila pública — legado/
  duplicado, ver ADR-0002). **Template triplicado**: `renderTemplate` (scheduler/retry, idênticos) +
  `renderAutomationTemplate`→`RenderResult` (execute, variante **mais rica**: detected/resolved/
  missing/warnings) + `interpolate` (engine). **Alvo canónico da consolidação = a versão rica
  `RenderResult`.** `processAutomationTrigger` (em execute) é a função central madura (alias canónico,
  anti-loop, idempotência via unique 23505, debug logs padronizados) — boa.
- **Resiliência:** execute/scheduler/retry/send usam `safeApiCall` **em prod** (circuit breaker por-org,
  timeout via AbortController); o repo estava atrás nos 3 executores (ver #1, corrigido no PR #36).
  `automation-engine` **não** tem resiliência (mais um motivo para o stub).
- **Ação:** (a) consolidar template engine em `_shared/templateEngine.ts` (ADR + deploy-coupled);
  (b) decidir o destino do `automation-engine` (stub vs manter) — decisão de arquitetura.

### Repository Layer (data hooks)
- **Padrão correto** em vários hooks: resolvem `organization_id` do utilizador autenticado antes
  de escrever. Mas **defesa-em-profundidade em falta**: `useDeals`, `useFinances`, `useAutomations`,
  `useLandingIntegrations` fazem `SELECT`/`UPDATE`/`DELETE` **sem** `.eq('organization_id')`
  explícito (mutações por `.eq('id')` apenas), dependendo **só do RLS**.
- **Severidade correta = MÉDIO (não crítico):** o RLS está **ativo e org-scoped** em todas as
  tabelas (verificado: políticas `organization_id = get_user_org_id()` em deals/finances/automations,
  ALL). Logo **não há vulnerabilidade cross-tenant ativa** — é violação da regra de defesa-em-
  profundidade da CLAUDE.md ("filtro explícito obrigatório como segunda camada").
- **Ação:** adicionar `.eq('organization_id', organizationId)` a reads e mutações. **Coordenar com
  Antenor** (ficheiros partilhados, PRs #30/#31 ativos) — potencial ownership conflict → não alterar
  unilateralmente.

### CRM / Contacts / Deals
- `contacts` = fonte de verdade; dedup por remote_jid→phone no webhook (bom, invariante de
  identidade reforçada). `deals` nunca criados pelo WhatsApp no webhook (correto) — mas
  `sync-whatsapp-history`, `lead-intake` e `automation-engine` **criam deals** fora de ação do
  utilizador (ver #5/ADR-0002).
- Naming de stage: lógica deve usar `stage_id`/código, não texto. Confirmado uso de códigos nas
  automações; manter vigilância no dashboard/receita.

### Dashboard Providers / Reporting
- `useDashboard` já migrado para dados reais (bom, sem mock). Reporting concentrado no dashboard;
  sem camada de reporting dedicada — aceitável para o estágio.

### Calendar / Activities / Finance
- `useAppointments` aplica `.eq('organization_id')` (bom). `useFinances` **não** (ver #2).
- Finance é read/write simples sobre `financial_transactions` — candidato a hook com o padrão
  completo (loading/error/data + filtro explícito).

### Billing integration points
- `lib/entitlements.ts` (75 linhas) + `useBillingLimits` centralizam limites. Stripe **bloqueado**
  (gate Victor). Pontos de integração: webhook de subscrição atualiza estado; Keyros decide acesso;
  convites validam limites de lugar. Manter billing independente de Team (regra).

### Edge Functions / APIs
- 14 no repo; `_shared/` só com circuitBreaker+rateLimiter. Duplicação de template e de helpers de
  identidade (#3/#4). Sobreposição de nomes (#6). Endpoints públicos: ver auditoria dedicada
  `audits/public-edge-functions-audit.md` + ADR-0002.

### Event Bus / Background Jobs / Notifications
- **Não há event bus explícito** — o "barramento" é o encadeamento webhook→automation-execute e a
  fila `automation_executions` processada pelo cron `automation-scheduler`. Notificações = envios
  WhatsApp. Aceitável, mas **não documentado como tal**; recomendável um doc de arquitetura de
  eventos/fila (quem produz, quem consome, idempotência).

### Observability / Logging
- Bom: webhook e automações emitem `console.log(JSON.stringify({event, ...}))` com códigos estáveis;
  circuit breaker regista em `system_health_logs`; rate limiter em `system_events`.
- Inconsistente: muitas funções misturam logs estruturados com strings livres (`[whatsapp-proxy]
  ...`). **Ação:** helper `_shared/log.ts` (event code + contexto) e padronizar motivos.

### Performance
- Circuit breaker + timeout já protegem chamadas à Evolution. `webhook_logs`/`rate_limits`/
  `system_events` têm cleanup oportunista (bom). Riscos: N+1 em loops de sync (findContacts/findChats
  fazem upsert 1-a-1) — aceitável no volume atual; vigiar.

### Tests
- **Sem runner de testes no repo.** Só SQL pontual (ex.: `audit_invitation_changes_test.sql`).
  **Ação:** introduzir um runner mínimo (Deno test para edge helpers puros: templateEngine,
  whatsappIdentity, rateLimiter) — de baixo risco, alto valor, sem deploy.

---

## Backlog priorizado (não-gate primeiro)

1. **(feito)** Reconciliar scheduler/retry — PR #36.
2. **Docs/ADR** (não-gate): ADR shared-layer (templateEngine + whatsappIdentity + log); doc de
   arquitetura de eventos/fila; atualizar KB onde diverge.
3. **Testes de helpers puros** (não-gate): Deno test para templateEngine/rateLimiter/identity.
4. **Coordenar** com Antenor: hardening de defesa-em-profundidade nos hooks (#2).
5. **Deploy-coupled (gate)**: consolidação real do shared layer nas edge functions (após ADR).
6. **Decisão de arquitetura (gate)**: destino do `automation-engine`; cleanup de nomes (#6).

## Gates
Nada deployed/mutado nesta auditoria. Refactors reais de edge functions são **deploy-coupled**
(gate). Alterar hooks partilhados do frontend = **coordenação/ownership** (Antenor). Decidir
`automation-engine`/renomeações = **decisão de arquitetura**.
