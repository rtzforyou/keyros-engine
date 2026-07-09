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

## ✅ DEPLOY EXECUTADO (Victor autorizou "aprovado deploy #42")
Fluxo deploy-coupled concluído em 2026-07-09:
1. **Merge** do PR #42 → `main` @ `277360e`.
2. **Deploy** via MCP `deploy_edge_function` (layout repo-path: `index.ts` +
   `_shared/circuitBreaker.ts` + `_shared/templateEngine.ts`, conteúdo exato de
   `main`). Resultado: **automation-retry v8**, ACTIVE, verify_jwt=false.
3. **Validação do deployed** (`get_edge_function` v8):
   - `index.ts` importa `renderTemplate`/`TemplateContext` de
     `../_shared/templateEngine.ts`; **sem allowlist inline**.
   - `_shared/templateEngine.ts` + `_shared/circuitBreaker.ts` **bundlados** e
     resolvidos (o deploy não falharia se os imports não resolvessem).
   - **Smoke test runtime:** `POST` sem auth → **HTTP 401 `{"error":"Não
     autenticado"}`** → a função **arranca e corre** (imports resolvem em
     runtime), auth-guard ativo, **sem efeitos colaterais**.
4. **Sem drift:** `main` (com a adoção) == prod (v8 deployed).

## Estado
**PARADO antes da adoção #2**, conforme instruído. Não avanço para
`automation-scheduler` até nova autorização.

## Próximo (aguarda gate)
Adoção #2: `automation-scheduler` (mesmo template inline idêntico), mesmo padrão
pequeno + deploy-coupled. Depois `automation-execute` (usa a variante rica
`renderAutomationTemplate` — requer decidir o alvo canónico primeiro) e os
helpers `whatsappIdentity`.
