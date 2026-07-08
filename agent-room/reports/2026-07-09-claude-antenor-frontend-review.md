# Report — Revisão/correção dos frontend PRs do Antenor

Task: Rever e corrigir os frontend improvements #18/#20/#21/#22/#23 + parte segura do #24
Agent: Claude / Fable (autorização única de Victor — Antenor sem créditos)
Status: needs_review — corrigido e a compilar; parado antes de merge/deploy (gate)
Repo: rtzforyou/easytattoo-crm
Branch: claude/antenor-frontend-review (base = loop4 = #18+#20+#21+#22+#23)
Commit SHA: 53387639d805f736a3a6dc1bce1e51d60b8369b8
PR: #26 (draft)

## Contexto
Os 6 PRs (#18,#20,#21,#22,#23,#24) estavam CLOSED e em stack acumulativo
(#20→#21→#22→#23→#24; #24 = superset). loop4 (#23) contém todo o frontend seguro
+ o normalizer de mensagens do #18. #24 acrescenta o alinhamento de vocabulário
MAS acopla a camada de resiliência/campanhas (deployed-uncommitted, backend).
Base escolhida: loop4 + só a parte segura do #24. Frontend-only.

## Achados e correções
1. Build partido (import): MessagesContent importava `useWhatsAppAvailability`
   (hook do #24) ausente em loop4. Adicionado o hook (read-only, seguro).
2. Build partido (órfão de main): ContactDetailsDrawer importava `./ContactFormsTab`
   (removido com a feature Forms). Aba/import órfãos removidos.
3. BUG FUNCIONAL: ConversationList comparava `chat.type` com 'group'/'direct'
   (minúsculas) enquanto o ConversationViewModel produz 'GROUP'/'DIRECT'. Os
   filtros Grupos/Privados e o badge de grupo nunca funcionavam. Corrigido
   (alinhado com ChatHeader).

## Verificação
- tsc: limpo. build: ✓.
- Normalizer em useRealMessages revisto: contactMap counterparty-only,
  group.subject — correto, sem regressão de identidade.

## Deixado de fora (parte não-segura do #24)
useCampaigns, useSystemHealth, SystemHealthSettings, CampaignProgressPanel,
useAutomations bulkConfig, circuit breaker em lib/whatsappService — camada
backend deployed-uncommitted (reconciliação EPIC 3). Alinhamento total de
vocabulário App/Sidebar → moduleRegistry: follow-up (loop4 é funcionalmente
correto).

## Gate
Parado antes de merge/deploy. Não toquei em backend/RLS/edge.

## Próximo passo
Merge do PR #26 (gate) após review. Depois: (a) reconciliação repo↔prod da
resiliência (EPIC 3); (b) alinhamento total de vocabulário frontend.
