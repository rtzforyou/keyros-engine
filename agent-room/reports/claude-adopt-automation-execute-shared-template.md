# Report — Adoção #3: automation-execute usa o rich `_shared/templateEngine.ts` (ADR-0003)

Autor: Claude / Fable · Data: 2026-07-09
PR: rtzforyou/easytattoo-crm **#45** · Branch: `claude/adopt-shared-rich-automation-execute`
Base: `main` @ `0e2fec2` (após #44) · Relacionado: ADR-0003, adoções #1 (retry v8), #2 (scheduler v24)

## Objetivo
Terceiro e último PR de adoção da shared layer, deploy-coupled, alvo único
`automation-execute` (o executor principal — envio imediato disparado pelo webhook).

## Regras cumpridas
- [x] PR separado (#45).
- [x] Alvo **somente** `automation-execute`.
- [x] Removido **apenas** o renderer rico inline (`TemplateContext`, `VARIABLE_ALLOWLIST`,
  `VARIABLE_ALIASES`, `RenderResult`, `renderAutomationTemplate`).
- [x] Importa `renderAutomationTemplate` + `RenderResult` (+ `TemplateContext`) de
  `../_shared/templateEngine.ts`.
- [x] **Não toca** em `automation-retry`.
- [x] **Não toca** em `automation-scheduler`.
- [x] **Não toca** em `landing-lead`.
- [x] `deno check` executado (ver nota abaixo).
- [x] `deno test` executado → 24 passed | 0 failed.
- [x] Validada equivalência contra o deployed (código idêntico).
- [x] Relatório publicado (este ficheiro).
- [x] **Parado antes do deploy** — aguarda aprovação explícita do Victor.

## Diff
`supabase/functions/automation-execute/index.ts`: **+2 / −125** (um ficheiro).
`buildTemplateContext` (I/O de DB) fica local.

## Equivalência vs deployed
O bloco inline removido foi comparado com o `_shared` removendo **todos** os
comentários (inclusive de fim de linha): **código idêntico**. Allowlist e aliases
também token-idênticos (o `_shared` foi extraído verbatim do próprio `automation-execute`
no PR #44). Logo o render é **byte-idêntico** ao da versão deployed **v22** — a única
diferença é a origem do código (inline → módulo partilhado).

## Nota — `deno check` (1 erro PRÉ-EXISTENTE, fora de escopo)
`deno check` acusa **1 erro de type-check não relacionado** com esta mudança:
`result.error` (`string | undefined`) passado a
`updateAutomationStatus(..., error: string | null)` (linha ~375). **Confirmado
presente no `origin/main`** (versão original, sem a minha mudança); a adoção
**não introduz erros novos** (1 antes = 1 depois). Por regra ("remover apenas o
renderer rico inline"), **não corrigido aqui** (fora de escopo). Não bloqueia o
deploy — o v22 foi deployed com este mesmo código (o bundler de deploy não faz
type-check estrito). **Follow-up separado sugerido:** tipar `error?: string | null`.

## ⚠️ GATE — deploy-coupled
Ao mergear, `automation-execute` **tem de ser deployed no mesmo passo** (bundla o
`_shared/templateEngine.ts`). É o **executor principal** (envio imediato do webhook)
— no deploy validar com atenção (get_edge_function + smoke). **NÃO deployar sem
aprovação explícita do Victor.** Parado aqui.

## Estado global ADR-0003 (após aprovação+deploy do #45)
Shared layer completo: retry (v8), scheduler (v24), execute (#45) — todos a usar
`_shared/templateEngine.ts`. Duplicação do template engine eliminada. Restaria (fora
deste âmbito) o `automation-engine` legado (decisão stub vs manter) e os helpers
`whatsappIdentity`.
