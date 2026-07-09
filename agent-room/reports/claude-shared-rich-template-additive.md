# Report — `_shared/templateEngine.ts` variante rica (aditivo, ADR-0003 Opção C)

Autor: Claude / Fable · Data: 2026-07-09
PR: rtzforyou/easytattoo-crm **#44** · Branch: `claude/shared-rich-template`
Base: `main` @ `2f186f4` · Relacionado: proposta `claude-automation-execute-rich-template-architecture.md`, ADR-0003

## Objetivo
Passo 2 (aprovado: Opção C) — tornar o `_shared/templateEngine.ts` canónico para a
variante rica, **sem** adotar em nenhuma função e **sem** deploy.

## Alterações (só `_shared/templateEngine.ts` + teste)
- **+ `RenderResult`** — `rendered` + `detected`/`resolved`/`missing`/`warnings`.
- **+ `renderAutomationTemplate(...): RenderResult`** — núcleo rico (verbatim do `automation-execute`).
- **`renderTemplate(...): string`** → **wrapper** `renderAutomationTemplate(...).rendered`.
  Assinatura inalterada. UMA só implementação de render.
- Allowlist / aliases / `TemplateContext` inalterados.
- Diff: `templateEngine.ts` +64/−9 · `templateEngine.test.ts` +57.

## Critério de aceite — estado
- [x] **PR aditivo** (#44) — toca apenas os 2 ficheiros `_shared`.
- [x] **Testes verdes** — `deno test supabase/functions/_shared/` = **24 passed | 0 failed**;
  `deno check` OK.
- [x] **Compatibilidade preservada** — teste-invariante
  `renderTemplate === renderAutomationTemplate().rendered` sobre 12 inputs
  (allowlist, alias, case, desconhecido, valor vazio/`'0'`/só-espaços, anti-injeção,
  múltiplas variáveis) → passa.
- [x] **Relatório publicado** (este ficheiro).
- [x] Sem adoção em `automation-execute`.
- [x] Sem alterar `automation-retry` / `automation-scheduler` / `landing-lead`.
- [x] Sem deploy.

## Cobertura de testes acrescentada
- `renderAutomationTemplate` — detected/resolved/missing/warnings (com alias e
  desconhecido); anti-injeção `<>`; alias (detected=raw, resolved=canónica).
- Invariante de compatibilidade `renderTemplate === ...rendered`.

## Porque é seguro sem deploy / sem drift
Bundles deployed (`automation-retry` v8, `automation-scheduler` v24) são **snapshots
congelados** do `_shared` no momento do deploy; alterar o `_shared` em `main` **não**
afeta o que já corre. Só é re-bundlado no próximo deploy dessas funções — e aí o
output é **idêntico** (provado pelo invariante). Logo o merge deste PR é seguro sem
qualquer deploy.

## Próximo (aguarda gate)
Passo 3 — adoção #3 `automation-execute` (**deploy-coupled**): após o merge do #44,
remover a cópia inline (`RenderResult`/`renderAutomationTemplate`/allowlist/aliases/
`TemplateContext`) e importar do `_shared`; `buildTemplateContext` (I/O) fica local;
`deno check`/`deno test`; validar vs deployed; **parar antes do deploy** para aprovação.

## Gate
Nada adotado, nada deployed. **Parado antes de qualquer adoção ou deploy.**
