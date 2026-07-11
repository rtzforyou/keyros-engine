# Claude — Keyros Landing: animação contínua (POC) — 2026-07-11

## EPIC
Keyros Landing Page — fase inicial: animação contínua única controlada por scroll (comando de Victor, 2026-07-11).

## Status
DONE (prova de conceito) — PR draft aberto, parado no gate. NO MERGE / NO DEPLOY / NO APPLY.

## Entrega
- Branch (produto): `claude/keyros-landing-continuous-animation`
- Commit: d10053ec5812bb38b242de1fce6e25a300e61b90
- PR: rtzforyou/easytattoo-crm#61 (draft)
- Docs: `docs/keyros-experience/landing-continuous-animation.md`

## Resumo técnico
- Página pública antes do login (lazy chunk 14.25 kB / 5.49 kB gzip; bundle CRM inalterado; 0 deps novas).
- Timeline: progresso de scroll normalizado 0..1; 9 cenas em intervalos ajustáveis (`timeline.ts`).
- Render Canvas 2D procedural, determinístico em (progress, time) → scroll reversível por construção. Sem WebGL.
- Sequência exata aprovada: chave enferrujada → restaurada → ampulheta → areia → cérebro → luz → caminho → globo.
- Fallbacks: `lite` (mobile) e `static` (prefers-reduced-motion / hardware fraco). Pausa fora do ecrã; cleanup no unmount.
- Placeholders procedurais em todas as cenas; manifesto + loader progressivo prontos para os assets finais (`assets.ts`).
- Dashboard autenticado, convites (?token=), Project Memory, PR #54, KCC e Partner Platform: não tocados.

## Testes
Build Vite ✅ · tsc --noEmit ✅ · 9 cenas validadas via scroll real no dev server (desktop + 527px lite).

## Pendências (gate Victor/ChatGPT)
1. Revisão do PR #61 (draft).
2. Exportação dos assets finais por cena (recortes alpha / sequências) a partir das folhas aprovadas (Assets 1, 2, 3, 4, 6).
3. Copy final + i18n (labels de cena e "Entrar" são placeholders de dev).
4. Decisão futura: manter Canvas 2D ou subir para WebGL quando os assets finais existirem (com medição de bundle).

## Graph
GRAPH.md inalterado: a landing é camada pública de marketing/onboarding pré-auth, fora dos nós core do produto; se a landing evoluir para domínio permanente (onboarding + billing público), propor nó novo em PR próprio.
