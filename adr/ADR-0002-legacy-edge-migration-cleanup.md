# ADR-0002 — Legacy Edge / Migration Cleanup

Estado: **Proposto** (aguarda decisão de arquitetura + gates de deploy)
Autor: Claude / Fable · Data: 2026-07-09
Relacionado: ADR-0001, `audits/public-edge-functions-audit.md`, incident report (PR #11)

## Contexto

Duas fontes de dívida na camada de plataforma, expostas pelo incidente de segurança
de 2026-07-09:

1. **Edge functions legadas/debug** públicas e perigosas (ver auditoria). Várias são
   **deployed-only** — existem em produção mas **nunca estiveram no repo** (drift):
   `setup-org-storage`, `sync-whatsapp-history`, `automation-engine`, `lead-intake`,
   `whatsapp-sync`, `appointment-api`. `debug-whatsapp` era deployed-only até ser
   trazido ao repo como stub (PR #33). `evolution-webhook` é repo-only (não deployed).

2. **Ledger de migrações desalinhado**: ~54 migrações aplicadas em produção vs ~34
   ficheiros no repo. O **schema é consistente**; o que diverge são nomes/histórico
   no ledger `supabase_migrations` (migrações aplicadas via `execute_sql`/MCP fora do
   fluxo de ficheiros). Não é um risco de dados — é um risco de *rastreabilidade*.

## Decisão

Limpeza faseada, cada fase atrás do seu gate. Nunca apagar em produção sem passar
antes por um stub neutralizado versionado (rollback = redeploy do stub).

### Fase 0 — Contenção do incidente (FEITO)
Debug trio (`debug-whatsapp`, `debug-api`, `diagnostic`) → stubs 410 **deployed** e
verificados. Reconciliados no repo (PR #33).

### Fase 1 — Neutralizar os writers públicos ALTO (gate: deploy)
- `sync-whatsapp-history` → deploy do stub 410 (PR #34); redundante com o
  whatsapp-webhook. Recomendado seguir com delete.
- `lead-intake` → **decisão necessária**: confirmar se alguma integração externa
  (Meta/IG/Zapier) o usa. Se não → deploy do stub (PR #34). Se sim → reconstruir com
  o modelo do `landing-lead` (form_id + public_token + rate limit + honeypot); nunca
  aceitar `organization_id` do payload.

### Fase 2 — Anti-spoofing do webhook (gate: deploy + env)
`whatsapp-webhook` + `whatsapp-proxy` → segredo partilhado `WHATSAPP_WEBHOOK_SECRET`
(PR #35). Rollout sem downtime: deploy → definir segredo → re-registar webhooks.

### Fase 3 — Hardening dos internos (gate: deploy)
- `automation-scheduler` (cron) e `automation-engine`: exigir auth de chamador
  (validar `Authorization: Bearer <service_role>` ou segredo dedicado); só devem ser
  chamados por cron/outras edge functions. `automation-engine` parece **duplicar** o
  `automation-scheduler`/`automation-execute` — avaliar stub/remoção.
- `setup-org-storage`: exigir auth ou `verify_jwt=true` (chamado no fluxo de criação
  de org); nunca aceitar `org_id` do payload sem validar o chamador.

### Fase 4 — Reconciliar deployed-only restantes (gate: deploy/decisão)
`whatsapp-sync`, `appointment-api`: trazer a fonte real ao repo (como em PR #29) ou
apagar se legado. Remover `evolution-webhook` do repo se não for para deployar.

### Fase 5 — Reconciliar ledger de migrações (sem alteração de schema)
Documentar o mapeamento aplicadas↔ficheiros; adotar doravante `apply_migration` (MCP)
para tudo que seja DDL, de forma a manter ledger==ficheiros. Sem DROP, sem reescrever
migrações antigas.

### Pós-limpeza (gate: dados / rotação de chave)
- Limpar 2 linhas `"Simulated Insert"` em `messages` (abuso do debug-api; 2026-06-19).
- **Rodar `WHATSAPP_GLOBAL_API_KEY`** — foi exercível publicamente via as funções debug.
- Delete definitivo das funções neutralizadas (debug trio, sync-whatsapp-history,
  lead-intake se confirmado).

## Rollback
Cada função neutralizada tem como baseline segura o **stub versionado** (410, sem
service_role, sem I/O). Rollback de qualquer deploy = redeploy do stub — **nunca**
restaurar a versão perigosa. Remoção do `WHATSAPP_WEBHOOK_SECRET` desativa a
validação de spoofing sem perder eventos. As alterações de schema são nulas (Fase 5
é só documentação/processo).

## Consequências
- (+) Superfície pública anónima reduzida a endpoints com auth própria.
- (+) Repo passa a refletir produção (fim do drift deployed-only).
- (+) Regra "endpoints públicos nunca aceitam organization_id do payload" reposta.
- (−) Requer coordenação de deploys + rotação de chave + re-registo de webhooks.
- (−) `lead-intake` exige uma decisão de produto (integrações externas).
