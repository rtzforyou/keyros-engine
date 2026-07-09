# Report — PR #37 Shared Layer Validation (ADR-0003 fase 1)

Autor: Claude / Fable · Data: 2026-07-09
PR: rtzforyou/easytattoo-crm #37 · Branch: `claude/backend-shared-templateengine`
Relacionado: ADR-0003 (keyros-engine PR #13)

## Objetivo
Deixar o PR #37 pronto para review/merge como **fase aditiva** do ADR-0003:
módulos partilhados puros + testes, **sem** adoção em nenhuma função (sem
deploy-coupled).

## Ações
1. **Rebase sobre `main`** (após merge do PR #36, main @ `74278e9`). Rebase limpo,
   sem conflitos (ficheiros disjuntos).
2. **Confirmado aditivo** — `git diff origin/main...HEAD` toca **apenas** 4 ficheiros,
   todos em `supabase/functions/_shared/`:
   - `templateEngine.ts` (novo)
   - `templateEngine.test.ts` (novo)
   - `whatsappIdentity.ts` (novo)
   - `whatsappIdentity.test.ts` (novo)
   - **nenhuma função deployed alterada** (verificado: 0 ficheiros fora de `_shared/`).
3. **Testes executados** com `deno test supabase/functions/_shared/`:

```
ok | 20 passed | 0 failed (23ms)
  templateEngine.test.ts .... 9 passed
  whatsappIdentity.test.ts .. 11 passed
```

4. **Type-check** com `deno check` nos módulos: OK (ambos).
5. **Sem correções necessárias** — testes verdes à primeira.
6. **`_shared` NÃO adotado** em nenhuma função (respeita a fronteira desta fase).
7. **Nenhuma alteração** a `automation-execute`/`scheduler`/`retry`/`landing-lead`.

## Ambiente de teste
- `deno 2.9.2` (instalado via brew nesta máquina — antes ausente).
- std de asserts fixado em `https://deno.land/std@0.168.0/testing/asserts.ts`
  (mesma versão dos imports das edge functions).

## Cobertura dos testes
- **templateEngine** (9): allowlist (double/single braces), resolução de alias,
  case-insensitive + trim, variável fora da allowlist → vazio, valor ausente →
  vazio, anti-injeção (remove `<`/`>` + trim), múltiplas variáveis, texto sem
  variáveis inalterado.
- **whatsappIdentity** (11): `normalizeJid` (device suffix / sem device / vazio),
  `getPhone` (privado com device / grupo), `normalizePhone` (BR 10/11 dígitos
  ganham 55, 12/13 com 55 mantêm, limpa não-dígitos, vazio).

## Critério de aceite — estado
- [x] Aditivo (só `_shared/`)
- [x] Sem deploy-coupled (nenhuma função adota `_shared` ainda)
- [x] Testes documentados **e executados** (20/20 verdes)
- [x] Rebased sobre `main` pós-#36
- [x] PR marcado ready-for-review (merge continua a ser gate)

## Próximo (fora deste PR)
Adoção função a função, **começando por `automation-retry`**, cada uma como PR
próprio **deploy-coupled** (o refactor é deployed no mesmo passo, validado contra
o deployed), conforme ADR-0003 faseamento.
