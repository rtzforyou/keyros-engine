# ADR-0003 — Edge Functions Shared Layer

Estado: **Proposto** (adoção nas funções é deploy-coupled → gate)
Data: 2026-07-09 · Autor: Claude / Fable
Relacionado: `audits/product-core-backend-audit.md`, ADR-0002

## Contexto
As edge functions de automação duplicam lógica pura:
- **Template engine** (`renderTemplate` + `buildContext` + `VARIABLE_ALLOWLIST` +
  `VARIABLE_ALIASES`) está inline e quase idêntico em `automation-scheduler` e
  `automation-retry`, com um terceiro allowlist em `automation-execute` e um
  `interpolate()` divergente em `automation-engine`.
- **Identidade WhatsApp** (`normalizeJid`, `getPhone`, `normalizePhone`) repetida em
  `whatsapp-webhook`, `sync-whatsapp-history`, `whatsapp-send`, `automation-engine`.
- **Logs** misturam `console.log(JSON.stringify({event...}))` com strings livres.

`_shared/circuitBreaker.ts` e `_shared/rateLimiter.ts` já provam o padrão de shared
layer (bundle relativo `../_shared/...`).

## Decisão
Criar módulos puros em `supabase/functions/_shared/`:
- `templateEngine.ts` — allowlist + aliases + `renderTemplate` + tipos de contexto.
- `whatsappIdentity.ts` — `normalizeJid` / `getPhone` / `normalizePhone`.
- `log.ts` — `logEvent(code, ctx)` com códigos estáveis (ver OBSERVABILITY).

Cada módulo é **puro e testável** (Deno test), sem I/O. As funções passam a importar
destes módulos em vez de manter cópias.

## Faseamento (crítico — drift)
1. **Aditivo, não-gate:** criar os módulos `_shared/*` + testes Deno. Não altera
   nenhuma função deployed → **sem drift**.
2. **Deploy-coupled, gate:** cada função adota o módulo (remove a cópia inline) e é
   **deployed no mesmo passo** — para não criar drift repo↔prod. Um domínio de cada vez
   (scheduler+retry primeiro, por serem idênticos), com validação vs deployed.

## Alternativas
- Manter duplicado: rejeitado (dívida cresce; correções divergem — ex.: um fix de
  template só num dos dois).
- Refatorar tudo de uma vez sem deploy: rejeitado (drift repo↔prod, o maior risco
  recorrente deste repo).

## Consequências
- (+) Uma fonte de verdade por helper; testável; correções num só sítio.
- (+) Testes cobrem lógica pura hoje sem cobertura.
- (−) Adoção exige deploys coordenados (gate) função a função.
- (−) Módulos ficam temporariamente sem uso entre a fase 1 e a adoção.
