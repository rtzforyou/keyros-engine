# Auditoria — Edge Functions públicas (verify_jwt=false)

Autor: Claude / Fable · Data: 2026-07-09 · Contexto: Security Incident + hardening
Projeto Supabase: `xqifnqrvcnpxkxftycvf` (eu-west-1)

Fonte de verdade: `get_edge_function` (código deployed real), não o repo (há drift
deployed-only). 20 funções deployed; 15 com `verify_jwt=false`.

## `verify_jwt=true` (5) — não públicas (gateway exige JWT)
`send-whatsapp`, `twilio-webhook`, `automation-execute`, `whatsapp-sync`, `appointment-api`.

## `verify_jwt=false` (15) — classificação

Legenda: **own-auth** = a função valida o chamador por si mesma (JWT/token/segredo).

| # | Função | Classe | own-auth | service_role | muta prod | risco público | Veredicto / ação |
|---|--------|--------|:---:|:---:|---|---|---|
| 1 | **debug-whatsapp** | debug | ❌ | Evolution key | **DELETE instâncias** | CATASTRÓFICO | ✅ STUB deployed v11 → delete |
| 2 | **debug-api** | debug | ❌ | ✅ | insert messages | CRÍTICO | ✅ STUB deployed v16 → delete |
| 3 | **diagnostic** | debug | ❌ | ✅ | não (dump total) | CRÍTICO | ✅ STUB deployed v11 → delete |
| 4 | **lead-intake** | legacy | ❌ (org do body) | ✅ | contactos+deals+automações | **ALTO** | STUB PR #34 (⚠️ confirmar ext.) |
| 5 | **sync-whatsapp-history** | legacy | ❌ (instance do body) | ✅ | contactos+chats+**deals** | **ALTO** | STUB PR #34 → deploy+delete |
| 6 | automation-engine | legacy/dup | ❌ (sem caller-auth) | ✅ | envia WhatsApp, cria deals | MÉDIO | REVER: stub ou service-auth |
| 7 | automation-scheduler | required (cron) | ❌ (sem caller-auth) | ✅ | envia WhatsApp | MÉDIO | hardening: exigir service-auth |
| 8 | setup-org-storage | legacy | ❌ (org do body) | ✅ | pastas storage (.keep) | BAIXO | hardening: auth / verify_jwt=true |
| 9 | whatsapp-webhook | required | ❌ (design: Evolution) | ✅ | mensagens/contactos/automações | MÉDIO (spoofing) | PR #35 (segredo partilhado) |
| 10 | whatsapp-proxy | required | ✅ JWT→org | ✅ | instância da própria org | OK | — |
| 11 | whatsapp-send | required | ✅ JWT | ✅ | mensagens | OK | — |
| 12 | automation-retry | required | ✅ JWT + tenant | ✅ | execução da própria org | OK | — |
| 13 | landing-lead | required | ✅ form_id+public_token | ✅ | contactos/deals (scoped) | OK | — |
| 14 | automation-bulk-send | required | ✅ JWT (org≠payload) | ✅ | envios (rate-limited) | OK | — |
| 15 | automation-campaign-control | required | ✅ JWT (org≠payload) | ✅ | flips de campanha (rate-limited) | OK | — |

## Padrão que separa seguro de inseguro
- **Seguro** (10–15): validam o chamador (JWT/token) **e resolvem `organization_id`
  do contexto autenticado, nunca do payload**. landing-lead usa o modelo público
  correto (form_id + public_token + rate limit + honeypot).
- **Inseguro** (1–8): `verify_jwt=false` **sem** auth própria. Os piores aceitam
  `organization_id`/`instance_name` do body (lead-intake, sync-whatsapp-history,
  setup-org-storage) → escrita cross-tenant anónima. Vários correm com `service_role`
  (bypassa RLS).

## Capacidade de mutação de produção (tarefa 5)
Mutadores entre os públicos inseguros: debug-api (insert), debug-whatsapp (DELETE
instâncias Evolution), sync-whatsapp-history (contactos/chats/deals),
lead-intake (contactos/deals + automações), automation-engine (envios/deals),
automation-scheduler (envios), setup-org-storage (storage). diagnostic = read-only
(mas dump total cross-tenant). whatsapp-webhook muta por design (mensagens).

## Estado de remediação (2026-07-09)
- Debug trio (1–3): **STUBBED + DEPLOYED**, verificado HTTP 410. Incident report:
  keyros-engine PR #11. Repo reconciliado: produto PR #33.
- sync-whatsapp-history + lead-intake (4–5): stubs preparados no produto PR #34
  (NÃO deployed).
- whatsapp-webhook (9): segredo partilhado preparado no produto PR #35 (NÃO deployed).
- automation-engine / automation-scheduler / setup-org-storage (6–8): documentados
  no ADR-0002 para hardening faseado.
