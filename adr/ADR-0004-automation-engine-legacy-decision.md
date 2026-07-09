# ADR-0004 — `automation-engine` legacy decision (stub vs keep)

Estado: **Proposto — recomenda NEUTRALIZAR (410 stub) + eliminar. Aguarda decisão de Victor (gate de arquitetura + deploy).**
Data: 2026-07-09 · Autor: Claude / Fable
Relacionado: EPIC B (ROADMAP), ADR-0002 (legacy edge cleanup), auditoria de funções públicas, PR #34.

## Contexto
`supabase/functions/automation-engine` (deployed v7) é um processador de fila de
`automation_executions` que **executa ações incluindo envio de WhatsApp** (via Evolution).
Características:
- `verify_jwt=false` (**público**), **sem auth própria** (qualquer request anónimo pode
  disparar o batch `{batch_size}` e drenar a fila).
- corre com `service_role` (bypassa RLS).
- **sem circuit breaker / timeout** (usa `fetch` direto), ao contrário de
  `automation-scheduler`/`automation-retry`/`automation-execute`, que já adotaram
  `safeApiCall` (`_shared/circuitBreaker.ts`).
- **template próprio** (`interpolate`), divergente do `_shared/templateEngine.ts` canónico.

## Evidência de que está morto/redundante
- **Nenhum cron o invoca.** `cron.job` tem apenas `automation-scheduler-every-minute`
  (`* * * * *`, ativo) → chama `automation-scheduler`. Não há job para `automation-engine`.
- **Único invocador = `lead-intake`** (`fetch(.../automation-engine)`), que é ele próprio
  um endpoint público órfão a ser neutralizado com 410 no **PR #34**.
- **Redundante:** o processamento de fila em produção é feito pelo cron `automation-scheduler`
  (v24); os envios imediatos são feitos por `automation-execute` (v23). `automation-engine`
  é um terceiro processador sobreposto.
- **Não está no repo** (`origin/main`) — é deployed-only (drift legado).

## Decisão proposta
**Neutralizar `automation-engine` com um 410 kill-stub** (como o trio debug), acoplado à
neutralização do `lead-intake` (PR #34) — assim que `lead-intake` deixa de o chamar, fica
sem qualquer caller. Depois, **eliminar** a função de produção.

Racional: é a única opção que remove simultaneamente (a) um risco de segurança (sender
WhatsApp público sem auth), (b) duplicação de lógica, e (c) drift deployed-only.

## Alternativas
- **Manter + adicionar auth + resiliência + adotar `_shared`:** rejeitado — investir num
  terceiro processador de fila redundante, sem caller e sem cron, não se justifica.
- **Deixar como está:** rejeitado — mantém um endpoint público que envia WhatsApp sem auth.

## Sequência (deploy-coupled / gate)
1. (gate, com PR #34) neutralizar `lead-intake` → `automation-engine` fica sem caller.
2. (gate) deployar 410 stub para `automation-engine` (mesmo padrão do trio debug).
3. (gate) validar (POST anónimo → 410; sem envios) e, depois, eliminar a função.

## Consequências
- (+) menos superfície pública; menos um sender WhatsApp sem auth; menos duplicação e drift.
- (−) se algum integrador externo desconhecido chamar `automation-engine` diretamente
  (não há evidência disso), perderia esse caminho — mitigado por confirmar logs antes do delete.

## Gate
Decisão de arquitetura + deploy = **gate de Victor**. Nada neutralizado/deployed até aprovação.
Registado na "AWAITING VICTOR — gate queue" (NEXT_ACTIONS do orquestra).
