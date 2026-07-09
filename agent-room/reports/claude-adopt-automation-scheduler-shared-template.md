# Report — Adoção #2: automation-scheduler usa _shared/templateEngine (ADR-0003)

Autor: Claude / Fable · Data: 2026-07-09
PR: rtzforyou/easytattoo-crm **#43** · Branch: `claude/adopt-shared-templateengine-automation-scheduler`
Base: `main` @ `277360e` · Relacionado: ADR-0003, adoção #1 (#42, deployed v8)

## Objetivo
Segundo PR de adoção da shared layer, pequeno e deploy-coupled, alvo único
`automation-scheduler` (o cron a cada minuto).

## Regras cumpridas
- [x] PR separado (#43).
- [x] Alvo **somente** `automation-scheduler`.
- [x] Removida **apenas** a cópia inline do template (`TemplateContext`,
  `VARIABLE_ALLOWLIST`, `VARIABLE_ALIASES`, `renderTemplate`).
- [x] Importa de `../_shared/templateEngine.ts`.
- [x] **Não toca** em `automation-execute`.
- [x] **Não toca** em `automation-retry`.
- [x] **Não toca** em `landing-lead`.
- [x] `deno check` executado → OK.
- [x] `deno test` executado → 20 passed | 0 failed.
- [x] Validado contra o deployed (token-idêntico → equivalente a v23).
- [x] Relatório publicado (este ficheiro).
- [x] **Parado antes do deploy** — aguarda aprovação explícita do Victor.

## Diff
`supabase/functions/automation-scheduler/index.ts`: **+3 / −44** (um ficheiro).
Mantêm-se locais (não são template puro): `getTzOffsetMinutes`, `applySendWindow`
(janela de envio de campanhas), `buildContext` (I/O de DB + lógica de agendamento).

## Garantia de zero mudança de comportamento
Bloco inline removido comparado **token-a-token** (removendo comentários e
espaçamento) com `_shared/templateEngine.ts`: **idêntico** (37 linhas cada, diff
vazio). Único delta era formatação. Render byte-idêntico ao da versão deployed v23.

## Validação executada
- `deno check supabase/functions/automation-scheduler/index.ts` → **OK**.
- `deno test supabase/functions/_shared/` → **20 passed | 0 failed**.
- Sem referências órfãs a `VARIABLE_ALLOWLIST`/`VARIABLE_ALIASES`.
- `git diff --name-only` = só `automation-scheduler/index.ts`.

## Validação contra deployed
Deployed `automation-scheduler` v23 tem o template **inline**; esta versão
importa-o de `_shared`. Conteúdo token-idêntico → **funcionalmente equivalente** ao
deployed; única diferença = origem do código (inline → módulo partilhado), que o
bundler resolve ao empacotar `../_shared/templateEngine.ts`.

## ✅ DEPLOY EXECUTADO (Victor autorizou "aprovado deploy #43")
Fluxo deploy-coupled concluído em 2026-07-09:
1. **Merge** do PR #43 → `main` @ `2f186f4`.
2. **Deploy** via MCP (layout repo-path: `index.ts` + `_shared/circuitBreaker.ts`
   + `_shared/templateEngine.ts`, conteúdo exato de `main`; shas dos `_shared`
   confirmados iguais aos do #42). Resultado: **automation-scheduler v24**,
   ACTIVE, verify_jwt=false.
3. **Validação do deployed (com atenção — é cron):**
   - **OPTIONS smoke** → **HTTP 200 `ok`** (early-return, sem processar a fila) →
     o módulo **carrega** (imports `_shared` resolvem em runtime). GET/POST não
     usados (disparariam envios).
   - **Cron real:** os logs edge-function mostram `POST | 200` para
     `automation-scheduler` na **version 24** (deployment `..._24`, ~1390ms) logo
     após o deploy → o **cron a cada minuto corre limpo** com o template
     partilhado, **sem erros**.
   - `_shared/templateEngine.ts` + `circuitBreaker.ts` bundlados e resolvidos.
4. **Sem drift:** `main` (com adoção) == prod (v24 deployed).

## Estado
**PARADO antes da adoção #3**, conforme instruído. Não avanço para
`automation-execute` até nova autorização.

## Próximo (aguarda gate)
Adoção #3: `automation-execute` — usa a variante **rica** `renderAutomationTemplate`
(→ `RenderResult` com detected/resolved/missing/warnings), diferente do
`renderTemplate` simples. Requer **decisão de arquitetura** primeiro: alinhar o
alvo canónico (adicionar a variante rica ao `_shared/templateEngine.ts` ou manter
duas funções). Depois os helpers `whatsappIdentity`.
