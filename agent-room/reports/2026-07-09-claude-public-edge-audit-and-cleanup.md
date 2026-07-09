# Report — Auditoria de edge functions públicas + remediação + cleanup EPIC

Agent: Claude / Fable · Data: 2026-07-09 · Sem merge/deploy (tudo em PR, gates respeitados)

## Pedido (5 tarefas)
1. Reconciliar PR #33 dos kill-stubs.
2. Preparar remediação do sync-whatsapp-history.
3. Auditar todas as Edge Functions públicas.
4. Rever spoofing/segurança do whatsapp-webhook.
5. Documentar Legacy Edge / Migration Cleanup.

## Entregue
1. **PR #33 reconciliado** (produto): stubs do repo alinhados ao deployed+verificado
   (v11/v16/v11, HTTP 410). Bloco `serve` idêntico; comentário atualizado ao estado real.
2. **PR #34** (produto, NÃO deployed): stubs 410 para `sync-whatsapp-history` e
   `lead-intake` (ambos deployed-only, writers públicos que criavam deals). Trá-los ao
   repo neutralizados. lead-intake com CAVEAT (integrações externas).
3. **Auditoria completa**: `audits/public-edge-functions-audit.md`. 15 funções
   verify_jwt=false classificadas (own-auth / service_role / mutação / risco). Achados
   novos: **lead-intake ALTO** (aceita organization_id do body) e **automation-engine
   MÉDIO** (processador de fila público que envia WhatsApp). automation-retry, bulk-send,
   campaign-control, whatsapp-proxy confirmados SEGUROS (JWT + org do contexto).
4. **PR #35** (produto, NÃO deployed): anti-spoofing do `whatsapp-webhook` via segredo
   partilhado `?token=` (backward-compatible; rollout sem downtime documentado).
5. **ADR-0002** (`adr/ADR-0002-legacy-edge-migration-cleanup.md`): EPIC faseado
   (contenção → writers ALTO → webhook → internos → deployed-only → ledger → pós/rotação),
   com rollback por stub versionado.

## Riscos pendentes (gates)
- Deploy dos stubs PR #34 e do segredo PR #35 = gate.
- lead-intake exige decisão de produto (integrações externas) antes de stub definitivo.
- automation-engine/scheduler/setup-org-storage: hardening de caller-auth (Fase 3).
- Pós-incidente: limpar "Simulated Insert", **rodar WHATSAPP_GLOBAL_API_KEY**, delete
  definitivo das funções neutralizadas.

## Gates respeitados
Nada deployed nesta ronda, nada apagado em produção, nada mutado, nenhuma chave rodada.
Tudo em PRs (produto #33/#34/#35; engine este report + auditoria + ADR).
