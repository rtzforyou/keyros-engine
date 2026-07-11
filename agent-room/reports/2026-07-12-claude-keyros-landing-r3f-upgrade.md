# Claude — Keyros Landing R3F Upgrade — 2026-07-12

## EPIC
AVANÇAR — KEYROS LANDING R3F UPGRADE (comando de Victor, 2026-07-12). Arquitetura R3F/WebGL aprovada.

## Status
DONE — PR #61 (draft) atualizado, parado no gate. NO MERGE / NO DEPLOY / NO APPLY.

## Entrega
- Branch (produto): `claude/keyros-landing-continuous-animation`
- Commits: d10053e (POC Canvas 2D) → 387da030887250052c8a67656f7db161fd050c94 (R3F upgrade)
- PR: rtzforyou/easytattoo-crm#61 (draft, corpo atualizado com relatório de performance)
- Docs: `docs/keyros-experience/landing-continuous-animation.md` (reescrito)

## Resumo técnico
- Renderer principal R3F/WebGL (desktop com GPU capaz); Canvas 2D = fallback lite (mobile/sem WebGL/erro); static = reduced-motion. Nunca página vazia.
- Deps aprovadas three/fiber/drei APENAS no chunk lazy (989 kB / 267.5 kB gzip). CRM: +0.07 kB (sem regressão). GSAP não adicionado.
- 7 GLBs de Victor comprimidos Draco+WebP 1024: 267 MB → 13.3 MB; decoder Draco self-hosted; lazy por proximidade de cena. 2 GLBs alternativos não usados (cérebros seccionados). Globo procedural (exigência do EPIC).
- Ferrugem: dissolve por noise com discard complementar (2 chaves, mesma geometria); nunca vira dourado; reversível.
- Fechadura: chave entra→gira→mecanismo GLB abre→só depois areia. Mecanismo = malha única (limitação documentada, nada inventado).
- Areia→cérebro: morph real 4500 partículas GPU (MeshSurfaceSampler seeded + vertex shader, stagger); mesh final só com formação ≥0.72; intermédio Neural Summit; reversível (verificado frame-idêntico).
- Scroll congela ao parar (frameloop=demand); zero relógio nas cenas; cursor secundário.

## Testes
tsc ✅ build ✅ 9 cenas via scroll real ✅ reversibilidade ✅ mobile 375px sem three/GLB na rede ✅ fallback por perda de contexto WebGL comprovado ao vivo ✅ heap ~150 MB.

## Pendências (gate Victor/ChatGPT)
1. Revisão do PR #61 (visual + direção de câmera).
2. Opcional: GLB do mecanismo com nós separados se quiser engrenagens articuladas.
3. Copy final + i18n.
4. Teste em GPU fraca real / dispositivos físicos antes de considerar produção.

## Graph
GRAPH.md inalterado — landing continua camada pública pré-auth fora dos nós core (mesma justificação do report anterior).
