# 🚨 SECURITY INCIDENT — public unauthenticated Edge Functions

Agent: Claude / Fable · Date: 2026-07-09 · Mode: Security Incident
Status: CONTAINED IN PR (remediação preparada) — NÃO deployed. Aguarda aprovação.
Remediation PR (produto): #33 (410 kill-stubs). Nada deployed/desativado/mutado.

## Resumo
Existem Edge Functions **debug/legadas** ACTIVE em produção com `verify_jwt=false`
(públicas, sem auth do gateway) que usam `service_role`/API keys e expõem ou
mutam dados cross-tenant sem qualquer autenticação própria.

## Inventário — 20 funções deployed; 15 com verify_jwt=false
`verify_jwt=true` (não públicas): send-whatsapp, twilio-webhook, automation-execute,
whatsapp-sync, appointment-api.

### verify_jwt=false — classificação (15)
| Função | Classe | service_role | muta prod | own-auth | Severidade |
|---|---|---|---|---|---|
| **debug-whatsapp** | debug | — (usa Evolution key) | **SIM (DELETE instâncias)** | ❌ | **CATASTRÓFICO** |
| **debug-api** | debug | ✅ | SIM (insert messages) | ❌ | **CRÍTICO** |
| **diagnostic** | debug | ✅ | não (read-all) | ❌ | **CRÍTICO** |
| **sync-whatsapp-history** | legacy | ✅ | SIM (cria deals/contactos/chats) | ❌ | **ALTO** |
| whatsapp-webhook | required | ✅ | SIM (por design) | ❌ (webhook Evolution) | MÉDIO (spoofing de eventos) |
| whatsapp-send | required | ✅ | SIM | ✅ valida JWT do user | OK |
| landing-lead | required | ✅ | SIM | ✅ form_id+public_token | OK |
| whatsapp-proxy | required | ? | controla Evolution | ❓ não verificado | REVER |
| automation-scheduler | required | ✅ | SIM | ❓ (cron) | REVER |
| automation-retry | required | ✅ | SIM | ❓ | REVER |
| automation-bulk-send | required | ✅ | SIM (envia) | ❓ | REVER (envio em massa!) |
| automation-campaign-control | required | ✅ | SIM | ❓ | REVER |
| setup-org-storage | legacy/unknown | ❓ | ❓ | ❓ | REVER |
| automation-engine | legacy/unknown | ❓ | ❓ | ❓ | REVER |
| lead-intake | legacy/unknown | ❓ | ❓ | ❓ | REVER |

(«REVER» = fonte não lida nesta passagem; verificar auth própria; adicionar auth
ou pôr verify_jwt=true se em falta.)

## Endpoints afetados (confirmados perigosos)
1. `debug-whatsapp` — CATASTRÓFICO. Anónimo → apaga TODAS as instâncias WhatsApp
   de TODAS as orgs (exceto `studio_4c0e0fb6df8945e5`). Perda de serviço total.
2. `debug-api` — CRÍTICO. Anónimo → 20 messages + 20 webhook_logs cross-tenant
   (telefones/conteúdo) + insere "Simulated Insert" em `messages` a cada hit.
3. `diagnostic` — CRÍTICO. Anónimo → TODOS os contactos + TODAS as conversas
   cross-tenant.
4. `sync-whatsapp-history` — ALTO. Anónimo (com um instance_name válido) → força
   sync que cria contactos/chats/DEALS (viola "WhatsApp nunca cria deal").

## Risco
- Confidencialidade: exposição pública de PII multi-tenant (contactos, conversas,
  mensagens, telefones, payloads de webhook).
- Integridade: escrita pública (insert de mensagens/deals/contactos).
- Disponibilidade: destruição pública de todas as instâncias WhatsApp.
- Chaves: `WHATSAPP_GLOBAL_API_KEY` exercível publicamente via estes endpoints.

## Ordem de remediação recomendada (gate — deploy)
1. **debug-whatsapp PRIMEIRO** (destrutivo). Deploy do kill-stub 410 (PR #33) OU
   delete da função. Máxima urgência.
2. **diagnostic** + **debug-api** (kill-stub PR #33 / delete).
3. **sync-whatsapp-history** — kill-stub/delete OU exigir auth (fast-follow; não
   incluída no PR #33 por objetivo=«debug», mas confirmada perigosa).
4. **Rever** os «REVER»: whatsapp-proxy, automation-scheduler/retry/bulk/campaign,
   setup-org-storage, automation-engine, lead-intake — confirmar auth própria;
   pôr verify_jwt=true / adicionar guard onde faltar.
5. **whatsapp-webhook** (defesa em profundidade): validar um segredo partilhado
   da Evolution para evitar spoofing de eventos.
6. **Pós-remediação (data — gate):** procurar linhas "Simulated Insert" em
   `messages` (abuso de debug-api) e limpar; verificar `whatsapp_instances` por
   apagões inesperados (abuso de debug-whatsapp); **rodar WHATSAPP_GLOBAL_API_KEY**
   se houver suspeita de abuso.

## Rollback
- Os kill-stubs 410 são de risco ~nulo (estas funções não são usadas pelo produto).
  Se um stub afetasse algo legítimo (não afeta), rollback = redeploy — mas NUNCA
  restaurar a versão perigosa; a baseline segura é o stub committed no PR #33.
- Preferível a stub: **delete** da função em produção (remove o slug). Rollback =
  redeploy a partir do stub.
- Data cleanup (Simulated Insert): delete direcionado — reversível apenas por
  backup; são linhas de lixo, remoção segura.

## Gates respeitados
NÃO deployed, NÃO desativado em produção, NÃO mutado, NÃO chamadas as funções,
NÃO rodada nenhuma chave. Tudo preparado em PR draft #33. Aguarda aprovação.
