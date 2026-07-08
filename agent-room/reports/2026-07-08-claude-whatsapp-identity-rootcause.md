# Report — WhatsApp Conversation Identity: causa raiz e correção

Task: Root-cause fix da corrupção de identidade de conversa (header ≠ mensagens)
Agent: Claude / Fable
Status: needs_review — código em PR draft; parado nos gates (deploy + data repair + merge)
Repo: rtzforyou/easytattoo-crm
Branch: claude/whatsapp-identity-rootfix
Commit SHA: 932ddc1dc46ab0d7a70627db836a6919dfc51aa1
PR: #16 (draft)

## Evidência (produção)

Chat `remote_jid = 41799020196@s.whatsapp.net`, `chat_type=private`:
`display_name = "Victor RTZ Tattoo"`, mas as mensagens são uma conversa 1:1
entre o dono (from_me=true, "Victor RTZ Tattoo") e o contacto real
(from_me=false, "Felipe - FLIP INK"). O contacto ligado chama-se "41799020196".

## Onde estava o bug — 1 ficheiro, 2 linhas

`supabase/functions/whatsapp-webhook/index.ts`, handler `MESSAGES_UPSERT`:
- `if (pushName && !isGroup) chatUpdates.display_name = pushName;`
- `if (!isGroup && pushName) await upsertWaContact(remoteJid, undefined, pushName);`

Ambas corriam **também para `fromMe = true`**.

## Porque acontecia

Um chat privado é identificado por `remote_jid` = o INTERLOCUTOR. O `pushName`
de uma mensagem `fromMe` é o DONO da instância. Gravar esse pushName no
`display_name` do chat e no `push_name` do `whatsapp_contacts` (ambos keyed pelo
interlocutor) faz o nome do dono aterrar na linha do contacto. Como a última
mensagem costuma ser do dono, o header mostrava o dono; as mensagens continuavam
a ser do contacto real. Fontes de "nome de dono": mensagens humanas ("Victor RTZ
Tattoo", "Você") e de automação ("Automação: boas vindas", etc.).

Os 3 commits recentes de "fix(messages)" (read layer, Antenor) tentavam corrigir
no ecrã — mas o dado persistido já estava corrompido, por isso não resolviam.

## Blast radius (verificado por SQL)

- 20 / 54 chats privados com `display_name` poluído.
- 22 `whatsapp_contacts` com `push_name` poluído.

## Auditoria completa (o que foi revisto)

- Webhook (`whatsapp-webhook`): único ponto que escrevia identidade do
  interlocutor a partir de mensagens. CHATS_UPSERT/CONTACTS_UPSERT/GROUPS_UPSERT
  usam dados do próprio interlocutor/grupo — corretos. `resolveContact` já era
  guardado por `!fromMe` (contacts.name / metadata.names.whatsapp intactos).
- `whatsapp-send`: NÃO escreve `display_name` a partir de dados do dono; o update
  pós-envio só toca last_message_*; sender_name da mensagem = 'Você'. Limpo.
- Identidade das tabelas: `whatsapp_chats.remote_jid` (Conversation Identity),
  `whatsapp_groups.remote_jid`+subject (Group Identity), `contacts`/`whatsapp_contacts`
  por phone/jid (Contact Identity). As chaves estão corretas — o bug era o VALOR
  do nome escrito na linha certa, não a chave.
- Sem duplicação de conversa: o upsert de chat é keyed por (org, instance,
  remote_jid); o read-hook ainda deduplica por remote_jid defensivamente.

## Correção (PR #16)

1. **Webhook (causa raiz):** `display_name` e `whatsapp_contacts.push_name` só
   são escritos a partir de mensagens recebidas (`!fromMe`). Invariante fixado
   com comentário. Grupos continuam a usar só `group.subject`.
2. **`lib/whatsapp/conversationIdentity.ts` (novo):** ETAPA ÚNICA de
   normalização. `toConversationViewModel(chat, ctx)` produz o
   `ConversationViewModel { conversationId, remoteJid, type, displayName, avatar,
   lastMessage, lastMessageAt, unreadCount, ... }`. Resolução de nome:
   - DIRECT: contacto → display_name do chat → telefone (nunca dono).
   - GROUP: só `group.subject`.
   Ficheiro novo e sem conflito. A ligação em `hooks/useRealMessages.ts` é o
   único ponto de coordenação com o Antenor (ver abaixo).
3. **Migração `20260712000003` (NÃO aplicada):** repara as linhas poluídas
   recomputando a identidade do último `sender_name` recebido; limpa para NULL
   quando não há nome recebido significativo. Só toca linhas poluídas.

## Dry-run da reparação (verificado, só leitura)

- `41799020196` → "Felipe - FLIP INK" ✅
- +16 nomes reais recuperados (Bruno Rosa, Evelyne, Lisete, Karina Colombo, …).
- 3 chats sem nome recebido → NULL (frontend cai para contacto/telefone).

## Fluxo antigo → novo

Antigo: Webhook (grava nome do remetente, incl. dono) → Persistência corrompida
→ read-hook lê push_name poluído → ViewModel improvisado no componente → header
trocado. Normalização espalhada por vários pontos.

Novo: Webhook (só interlocutor escreve identidade do interlocutor) →
Persistência limpa → 1 etapa de normalização (`conversationIdentity.ts`) →
ConversationViewModel → React renderiza. Nada além disto.

## Porque não pode voltar a acontecer

O invariante "identidade do interlocutor nunca vem de eco fromMe" está no único
ponto de escrita (webhook) e no único ponto de leitura (módulo de normalização).
Nenhum caminho de código grava o nome do dono na linha do contacto.

## Coordenação com Antenor (read layer)

`hooks/useRealMessages.ts` está no domínio frontend do Antenor e tem 3 commits
recentes (c3ba2a3/0ade5a5/c3ef7d6, "fix display name swaps"). Não editei esse
ficheiro (evitar cross-agent conflict — hard gate). Passo a ligação recomendada:
substituir o bloco `enriched = data.map(...)` por
`data.map(chat => toConversationViewModel(chat, { contactByPhone, groupByJid }))`
e remover a lógica defensiva de swap (deixa de ser necessária com o dado limpo).
`contactByPhone` deve ser construído SÓ de fontes do interlocutor (contacts +
whatsapp_contacts por phone) — já é o caso.

## Verificação

- Build: `npm run build` ✓. Typecheck: `npx tsc --noEmit` sem erros novos (3
  pré-existentes de main, não relacionados).
- Webhook Deno: fora do build vite — revisão estática.
- Reparação: dry-run SQL verificado (acima). Não aplicada.

Remote-only change: no (ainda). Migração committed no PR; apply pendente de gate.

## Gates (parado — pedir a Victor/ChatGPT)

1. Deploy do edge function `whatsapp-webhook` (production deploy).
2. Apply da migração `20260712000003` (production data change — 20 chats + 22 wa_contacts).
3. Merge do PR #16.
4. Coordenar a ligação do módulo de normalização em useRealMessages com Antenor.

## Rollback

- Webhook: redeploy da versão anterior (git).
- Reparação: os valores antigos eram o nome do dono (corrompidos), por isso não
  há "estado bom" anterior para restaurar; se preciso, recomputar de novo do
  último inbound (a migração é idempotente). Nenhuma perda de dados reais
  (mensagens intactas; só o campo de nome muda).

## APLICADO 2026-07-08 (autorização explícita de Victor)

- **Deploy:** whatsapp-webhook v43 ACTIVE (verify_jwt=false preservado). O fix
  foi deployed sobre a versão de PRODUÇÃO (v42), que já continha o rate-limiter
  do Antenor — preservado. Bundle usou `./rateLimiter.ts` (sibling) por
  limitação do bundler com `../_shared/`.
- **Reparação:** migração 20260712000003 aplicada. Verificação pós-apply:
  chat 41799020196 → display_name e push_name = "Felipe - FLIP INK";
  chats_still_poisoned = 0; wacontacts_still_poisoned = 0; 6 chats privados sem
  nome (fallback telefone/contacto no read layer).

## RISCO / DIVERGÊNCIA repo↔prod (ACÇÃO NECESSÁRIA)

O webhook em PRODUÇÃO tem o rate-limiter (`_shared/rateLimiter.ts` + bloco),
mas isso é trabalho NÃO COMMITADO do Antenor (working tree de
antenor/billing-ux-draft; ausente de origin/main). O meu PR #16 (baseado em
origin/main) NÃO contém o rate-limiter — só o fix de identidade de 2 linhas.

Consequência: se alguém fizer deploy do webhook a partir de main após o merge
do #16, PERDE o rate-limiter. Reconciliação necessária antes de qualquer deploy
futuro do webhook: committar o rate-limiter (`supabase/functions/_shared/
rateLimiter.ts` + o bloco no webhook) a main. É trabalho do Antenor — flag para
ChatGPT coordenar. O deploy que fiz agora está correto (prod tem tudo); o risco
é só em deploys futuros a partir de um main incompleto.
