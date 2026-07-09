# Proposta — variante rica de template no `_shared/templateEngine.ts` (ADR-0003 update)

Autor: Claude / Fable · Data: 2026-07-09 · **Proposta — aguarda aprovação (sem código)**
Contexto: adoção #3 (`automation-execute`) do shared layer. Relacionado: ADR-0003,
adoções #1 (retry v8) e #2 (scheduler v24), ambas deployed.

---

## 1. `automation-execute` atual (lido de `main`, 753 linhas)
Usa uma variante **rica** de render, `renderAutomationTemplate`, que devolve
`RenderResult` (usado para observabilidade: logs `template_render` e
`automation_event_logs` com detected/resolved/missing/warnings). Além disso tem
`buildTemplateContext` (I/O de DB — fica local, não é template puro).

## 2. Mapa exato da variante rica (verbatim de `main`)
```ts
interface RenderResult {
  rendered: string;
  detected: string[];   // toda variável encontrada (raw, lowercase)
  resolved: string[];   // canónicas resolvidas com valor não-vazio
  missing: string[];    // canónicas sem valor OU raw desconhecidos
  warnings: string[];   // `variável desconhecida: {{raw}}`
}

function renderAutomationTemplate(template, context): RenderResult {
  // regex: /\{\{([^}]+)\}\}|\{([^}]+)\}/g   (mesma do renderTemplate)
  // por match:
  //   raw = (double ?? single).trim().toLowerCase();  detected.push(raw)
  //   canonical = VARIABLE_ALIASES[raw] ?? raw
  //   se !VARIABLE_ALLOWLIST.has(canonical): warnings.push(...); missing.push(raw); return ''
  //   value = context[canonical]
  //   se value !== undefined && value !== '': resolved.push(canonical); return value.replace(/[<>]/g,'').trim()
  //   senão: missing.push(canonical); return ''
}
```
- **detected/resolved/missing/warnings:** ver acima.
- **anti-injeção:** `value.replace(/[<>]/g, '').trim()` — **idêntica** à do `renderTemplate` simples.
- **aliases:** `VARIABLE_ALIASES` — **token-idênticos** aos do `_shared` (verificado por diff).
- **allowlist:** `VARIABLE_ALLOWLIST` — **token-idênticos** aos do `_shared` (verificado por diff).

## 3. Comparação com o `_shared/templateEngine.ts` atual
`renderTemplate` (simples, no `_shared`):
```ts
// por match: raw; canonical; se !allowlist return '';
//   value; return value ? value.replace(/[<>]/g,'').trim() : ''
```

**Diferença = apenas metadados.** A string `rendered` é **idêntica** entre as duas
para qualquer input (provado caso a caso):

| input do valor | simples (`value ?`) | rica (`!==undefined && !==''`) | igual? |
|---|---|---|---|
| desconhecido na allowlist | `''` | `''` | ✅ |
| valor `undefined` | `''` | `''` (missing) | ✅ |
| valor `''` | `''` | `''` (missing) | ✅ |
| valor `'0'` (truthy) | `'0'` | `'0'` | ✅ |
| valor só espaços `' '` | `''` (após trim) | `''` (após trim) | ✅ |
| valor normal | sanitizado+trim | sanitizado+trim | ✅ |

→ `renderAutomationTemplate(t,c).rendered` **=== `renderTemplate(t,c)`** em todos os casos.
A rica é um **superconjunto**: mesmo render + diagnóstico.

## 4. Arquitetura proposta — **Opção C: adaptar o módulo atual com compatibilidade total**

Adicionar a variante rica ao **mesmo** módulo `_shared/templateEngine.ts`,
**unificando o núcleo de render** para não duplicar o loop:

```ts
// _shared/templateEngine.ts (desenho proposto)
export interface TemplateContext { /* inalterado */ }
export const VARIABLE_ALLOWLIST = /* inalterado */;
export const VARIABLE_ALIASES  = /* inalterado */;

export interface RenderResult {
  rendered: string; detected: string[]; resolved: string[];
  missing: string[]; warnings: string[];
}

// núcleo único (a versão rica)
export function renderAutomationTemplate(template: string, context: TemplateContext): RenderResult {
  /* exatamente a lógica rica do automation-execute */
}

// wrapper fino — mantém a assinatura string usada por retry/scheduler
export function renderTemplate(template: string, context: TemplateContext): string {
  return renderAutomationTemplate(template, context).rendered;
}
```

**Porquê C (e não A nem B):**
- **Rejeito B (novo módulo `_shared/automationTemplateEngine.ts`):** re-fragmenta o
  shared layer, **duplica** allowlist/aliases/regex em dois módulos — o oposto do
  objetivo do ADR-0003.
- **A (exportar as duas lado a lado sem unificar)** funciona, mas mantém **dois loops
  de render** quase iguais no mesmo ficheiro. C elimina essa duplicação com o wrapper.
- **C** dá: **um único núcleo de render**, `renderTemplate` com assinatura inalterada
  (string) → **retry/scheduler não mudam**, e `renderAutomationTemplate`+`RenderResult`
  para o execute. Compatibilidade total.

## 5. Compatibilidade / risco
- **retry (v8) e scheduler (v24):** importam `renderTemplate` → continua a devolver
  `string`, output **idêntico** (é `.rendered` da rica, provado). Os bundles deployed
  são snapshots congelados — mudar o `_shared` **não** afeta o que já corre; só é
  bundlado de novo no próximo deploy deles (produzindo o mesmo output). **Sem drift.**
- **execute:** passa a importar `renderAutomationTemplate` + `RenderResult` + `TemplateContext`
  e remove as cópias inline; `buildTemplateContext` (I/O) fica local.
- **Testes:** adicionar Deno tests para `renderAutomationTemplate` (detected/resolved/
  missing/warnings + anti-injeção) e um teste de invariante
  `renderTemplate(t,c) === renderAutomationTemplate(t,c).rendered`.

## 6. Sequência proposta (cada passo com o seu gate)
1. **(este doc)** Proposta/ADR update — **sem código**. ← aguarda aprovação.
2. **Após aprovação — PR aditivo ao `_shared`** (como o #37): adiciona
   `renderAutomationTemplate` + `RenderResult` + wrapper + testes Deno. **Aditivo, sem
   deploy** (módulo; só toma efeito quando uma função é deployed). `deno check`+`deno test`.
3. **Após merge do aditivo — PR de adoção #3 (`automation-execute`), deploy-coupled:**
   remove inline → importa do `_shared`; `deno check`/`deno test`; validar vs deployed;
   **parar antes do deploy** para aprovação (mesmo fluxo dos #42/#43).

## Recomendação
**Opção C** — adaptar `_shared/templateEngine.ts` com `renderAutomationTemplate` +
`RenderResult` e `renderTemplate` como wrapper do núcleo rico. É a única que remove
duplicação **e** mantém compatibilidade total com os usos já deployed.

## Gate
Nada mexido no produto. Sem PR de adoção, sem deploy. **Parado no gate de aprovação.**
Se aprovares a Opção C, avanço para o passo 2 (PR aditivo ao `_shared` + testes) e
paro antes de qualquer deploy.
