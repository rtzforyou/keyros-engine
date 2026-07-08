# Report — Translation Architecture Audit (override excecional)

Task: Claude Translation Architecture Audit (NEXT_ACTIONS override)
Agent: Claude / Fable (autorização única — frontend/i18n only)
Status: needs_review — feito e verificado; parado antes de merge/deploy (gate)
Repo: rtzforyou/easytattoo-crm
Branch: claude/translation-architecture-audit
Commit SHA: aa062552d1342787b5b09667bc69c43e933ae198
PR: #27 (draft)

## Bug corrigido
Sidebar seguia o locale, mas os cards/labels do Dashboard ficavam hardcoded em PT.
Drift de i18n: alguns textos por ID, outros embebidos nos componentes.

## Ficheiros alterados
- lib/translations.ts — chaves namespaced nos 3 locales (en/pt/fr):
  dashboardMetrics.* / dashboardPeriods.* / dashboardMisc.* (pt/fr sobrepõem os
  defaults en porque fazem ...enStrings).
- components/dashboard/DashboardContent.tsx — 8 títulos de KPI, filtros de
  período, e textos de erro/config/empty/secção → t.dashboard*. Lógica de
  negócio inalterada (DashboardPeriod/ids).
- docs/knowledge/29-translation-architecture.md — regra de arquitetura de tradução.
- docs/knowledge/README.md — link para #29.
- scripts/i18n-audit.sh — heurística para achar texto hardcoded restante.

## Chaves de tradução adicionadas
dashboardMetrics: openOpportunities, newLeads, grossRevenue, expenses, netProfit,
wonDeals, conversionRate, averageTicket, dealsUnit, finalizedUnit, wonDealsUnit.
dashboardPeriods: today, week, month, lastMonth, year, custom.
dashboardMisc: errorLoading, configStagesTitle, configStagesBody, emptyFunnel,
whatsappAutomationsSection. (× 3 locales.)

## Strings hardcoded removidas
Todas as do DashboardContent.tsx (títulos, filtros, erro, aviso de config, funil
vazio, cabeçalho de secção). Confirmado por scripts/i18n-audit.sh (0 no Dashboard).

## Regra documentada
UI text = translation ID; business logic = stable id/code. Sistema flat +
namespacing por prefixo (dashboard.metrics.* → dashboardMetrics.*).

## Verificação
- tsc + build: verde para as minhas mudanças (confirmado com fix temporário dos
  2 erros pré-existentes, depois revertido — commit fica i18n-only).
- Base (main) está PARTIDA por drift não relacionado: MessagesContent importa
  useWhatsAppAvailability, hook nunca commitado (igual à quebra do PR #26). Não é
  desta task; o PR i18n fica verde quando a base for reconciliada.

## Teste manual
Trocar idioma (PT/EN/FR) em Definições → cards do Dashboard, filtros e cabeçalhos
seguem o idioma.

## Dívida i18n restante (próxima passagem)
Subcomponentes: WhatsAppAutomationStats, LeadSourceChart, RevenueExpenseChart,
ExpensesChart, UpcomingAppointments, RecentActivity, PipelineChart.

## Não tocado
backend, migrations, RLS, auth, billing, Stripe, dados de produção.

## Blockers / próximo passo
Merge do PR #27 (gate) depende de reconciliar a base partida (commitar
useWhatsAppAvailability — PR #26). Ownership volta: Claude→backend, Antenor→frontend.
