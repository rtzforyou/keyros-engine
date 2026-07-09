# Report — Adoção #1: automation-retry usa _shared/templateEngine (ADR-0003)

Autor: Claude / Fable · Data: 2026-07-09
PR: rtzforyou/easytattoo-crm **#42** · Branch: `claude/adopt-shared-templateengine-automation-retry`
Base: `main` @ `3cc6e8e` (após merge do #37) · Relacionado: ADR-0003, PR #37 (aditivo)

## Objetivo
Primeiro PR de adoção da shared layer, **pequeno e deploy-coupled**, alvo único
`automation-retry`.

## Regras cumpridas
- [x] Alvo **somente** `automation-retry`.
- [x] Removida **apenas** a cópia inline do template (`TemplateContext`,
  `VARIABLE_ALLOWLIST`, `VARIABLE_ALIASES`, `renderTemplate`).
- [x] Importa de `../_shared/templateEngine.ts`.
- [x] **Não toca** em `automation-execute`.
- [x] **Não toca** em `automation-scheduler`.
- [x] **Não toca** em `landing-lead`.
- [x] `deno test` executado.
- [x] Validado contra o deployed.
- [x] Relatório publicado (este ficheiro).
- [x] **Parado antes do deploy** — aguarda aprovação explícita do Victor.

## Diff
`supabase/functions/automation-retry/index.ts`: **+3 / −44** (um ficheiro).
`buildContext` (I/O de DB) mantém-se local — não é lógica pura.

## Garantia de zero mudança de comportamento
O bloco inline removido foi comparado **token-a-token** com
`_shared/templateEngine.ts` (removendo comentários e espaçamento): **idêntico**.
Único delta era formatação (espaço após vírgula). Logo o render é byte-idêntico
ao da versão deployed v7 para qualquer input.

## Validação executada
- `deno check supabase/functions/automation-retry/index.ts` → **OK** (o import
  `_shared` resolve e type-checa).
- `deno test supabase/functions/_shared/` → **20 passed | 0 failed**.
- Sem referências órfãs a `VARIABLE_ALLOWLIST`/`VARIABLE_ALIASES`.
- `git diff --name-only` = só `automation-retry/index.ts`.

## Validação contra deployed
Deployed `automation-retry` v7 tem o template **inline**; esta versão importa-o de
`_shared`. Como o conteúdo é token-idêntico, é **funcionalmente equivalente** ao
deployed — a única diferença é a origem do código (inline → módulo partilhado),
que o bundler resolve ao empacotar `../_shared/templateEngine.ts`.

## ⚠️ GATE — deploy-coupled
Ao mergear, `automation-retry` **tem de ser deployed no mesmo passo** (o
`_shared/templateEngine.ts` é bundlado com a função). Se mergear sem deploy, o
repo fica à frente do prod (drift). **NÃO deployar sem aprovação explícita do
Victor.** Parado aqui.

## Próximo (após este ser aprovado+deployed)
Adoção #2: `automation-scheduler` (mesmo template inline idêntico), mesmo padrão.
Depois `automation-execute` (usa a variante rica `renderAutomationTemplate` —
requer alinhar o alvo canónico primeiro) e os helpers `whatsappIdentity`.
