# 🚨 SECURITY (HIGH/CRITICAL) — public debug-api leaks cross-tenant PII + mutates prod

Agent: Claude / Fable · Date: 2026-07-09 · Status: ESCALATION — requires gated fix (deploy)
Descoberto durante trabalho autónomo (auditoria de segurança das edge functions legadas).

## Finding
`debug-api` (deployed v15, **ACTIVE**, **verify_jwt=false → PÚBLICA sem auth**) corre
com `service_role` (bypassa RLS). Em QUALQUER request anónimo a
`.../functions/v1/debug-api`:
1. `SELECT * FROM messages ORDER BY created_at DESC LIMIT 20` → devolve 20 mensagens
   REAIS de QUALQUER organização (conteúdo + `remote_jid` = telefones). Cross-tenant.
2. `SELECT * FROM webhook_logs ... LIMIT 20` → payloads crus de webhook (telefones,
   conteúdo de mensagens, metadados).
3. `INSERT INTO messages (... 'Simulated Insert' ...)` → **escreve em produção** a
   cada chamada (mutação de dados por endpoint público).
4. Chama a Evolution API com `WHATSAPP_GLOBAL_API_KEY` e devolve dados de instâncias.

## Impacto
- **Exposição pública de PII multi-tenant** (mensagens + telefones + webhook payloads)
  a qualquer pessoa que conheça/adivinhe o URL. Viola o princípio central de
  isolamento de tenant.
- **Integridade:** insere lixo em `messages` em cada hit (spam/poison).
- Severidade: **HIGH/CRITICAL**.

## Recomendação urgente (gate — Victor/ChatGPT)
1. **Desativar/eliminar `debug-api` de produção imediatamente** (deploy/delete = gate).
2. Rever igualmente `debug-whatsapp` e `diagnostic` (também ACTIVE, verify_jwt=false)
   — provável exposição semelhante; desativar por precaução.
3. Após remoção: verificar `webhook_logs`/`messages` por inserts "Simulated Insert"
   e limpar (data change = gate).
4. Rodar `WHATSAPP_GLOBAL_API_KEY` se houver suspeita de abuso (foi exercível
   publicamente via esta função).

## Não executado (respeitando gates)
- NÃO chamei a função (não disparei o insert nem exercitei a exposição).
- NÃO fiz deploy/remoção (gate). NÃO mutei dados.

## Nota
Isto reforça a prioridade #1 da proposta Legacy Edge/Migration Cleanup: as funções
`debug-*`/`diagnostic` não deviam estar ACTIVE/públicas em produção.
