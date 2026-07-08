# Dashboard real-data backend — plano (finance + CRM metrics)

Task: NEXT.md Claude tarefa 9
Agent: Claude / Fable
Status: needs_review (plano; nada implementado)

## Estado actual (verificado em hooks/useDashboard.ts @ main)

Já é 100% dados reais, org-scoped: pipeline_stages, deals (id, value,
stage_id, financial_status, closed_at, created_at, stage), financial_transactions,
appointments, contacts, lead_submissions, whatsapp_messages,
automation_event_logs. Agregação é toda CLIENT-SIDE: o hook puxa as linhas do
período e calcula KPIs no browser. Existe fallback por texto de stage para
deals antigos sem stage_id.

## Problemas a resolver

1. **Escala**: payload cresce linearmente com o volume (um estúdio com 10k
   deals/mensagens paga o custo no browser a cada mudança de período).
2. **Campos estruturados em falta** (CLAUDE.md §7, schema alvo): deals não têm
   `category_id`, `expected_revenue`, `gross_profit`, `expenses`, `net_profit`
   — margens/lucro não são calculáveis por deal.
3. **Fallback por texto de stage** ainda vivo — viola a regra IDs-não-texto
   para deals pré-stage_id.
4. **Funil CRM**: não há histórico de transições de stage — conversão
   stage→stage não é medível, só snapshots.

## Plano (fases incrementais, cada uma isolada)

### Fase 1 — agregação server-side (sem mudar schema)
- RPC `get_dashboard_kpis(period_start, period_end)` SECURITY INVOKER (RLS
  aplica; org resolvida por auth.uid(), NUNCA parâmetro org_id do cliente),
  devolvendo os agregados financeiros (receita por financial_status, despesas,
  contagens por stage_id) num único round-trip.
- Atenção à lição da 20260706154621: views/RPCs de leitura têm de respeitar
  RLS (security invoker) — nunca SECURITY DEFINER sem validação manual.
- Índices: `deals(organization_id, closed_at)`, `deals(organization_id, stage_id)`,
  `financial_transactions(organization_id, date)` (verificar existentes antes).
- Hook mantém shape actual; troca N fetches por 1 RPC + fetches leves (listas).

### Fase 2 — backfill de stage_id e fim do fallback por texto
- Migração de dados: resolver `deals.stage_id` NULL por match do texto `stage`
  contra `pipeline_stages.name` da MESMA org; relatório de não-resolvidos.
- Depois: remover o fallback textual do hook (uma task isolada).

### Fase 3 — campos financeiros estruturados em deals (schema alvo §7)
- Adicionar incrementalmente conforme necessidade real: `expected_revenue`,
  `expenses`, `net_profit` calculado ou coluna. `category_id` já existe via
  `pipeline_stages.category` — dashboard passa a agrupar por ela.

### Fase 4 — funil CRM
- Tabela `deal_stage_events` (deal_id, org_id, old_stage_id, new_stage_id,
  changed_by, created_at) alimentada no update de stage (mesmo ponto que já
  dispara automations `deal_stage_changed`). Dashboard ganha conversão e tempo
  médio por stage.

## Gates

Fases 1–4 têm migrações → cada apply é gate. Fase 1 pode começar como PR
draft imediatamente após autorização.
