# ADR-0005 — WhatsApp Cloud API (Official Meta) Integration

Estado: **Proposto — arquitetura + decisões. Aguarda confirmação de Victor (D1–D5).**
Data: 2026-07-09 · Autor: Claude / Fable
Contexto: a app Meta foi verificada; podemos usar a **WhatsApp Cloud API oficial** em
vez da Evolution API (não-oficial, Baileys, risco de ban). Relacionado: EPIC C (ROADMAP).

## Contexto e diferenças que mudam a arquitetura
A Cloud API não é um "drop-in" da Evolution. Diferenças estruturais:
1. **Identidade do canal:** Evolution = `instance_name` + QR. Cloud = `phone_number_id`
   + `waba_id` + **access token permanente** (System User) por número. Sem QR.
2. **Envio:** `POST graph.facebook.com/v{ver}/{PHONE_NUMBER_ID}/messages` com
   `Authorization: Bearer <token>` (≠ Evolution `/message/sendText` + `apikey`).
3. **Janela de 24h (regra de negócio nova e crítica):** fora da janela de 24h desde a
   última mensagem do cliente, **só se pode enviar templates aprovados (HSM)** — não
   texto livre. Isto afeta diretamente as automações.
4. **Templates:** têm de ser criados e **aprovados na Meta** antes de uso proativo.
5. **Inbound:** a Meta envia para **um** callback verificado (GET com
   `hub.verify_token`/`hub.challenge`) e assina cada POST com **`X-Hub-Signature-256`**
   (HMAC-SHA256 com o App Secret) → validação obrigatória. Payload diferente.
6. **Status:** eventos `sent`/`delivered`/`read`/`failed` por mensagem.
7. **Multi-tenant:** cada estúdio tem o seu número/WABA.

## Modelo de dados atual (Evolution-centric)
`whatsapp_instances(instance_name, api_token, connection_status, phone_number, ...)`.
Sem `phone_number_id`/`waba_id`/`provider`/token encriptado/templates. Precisa de extensão.

## Decisões propostas (confirmar — D1–D5)

**D1 — Modelo de número multi-tenant.**
Recomendação: **BYON (Bring Your Own Number)** — cada estúdio liga o seu próprio número
WhatsApp Business. Faseado:
- Fase inicial: **ligação manual** (o estúdio cola `waba_id`, `phone_number_id` e um
  token permanente gerado no Meta Business Manager) — rápido de construir.
- Fase escala: **Embedded Signup** da Meta (self-serve, requer a app como Tech Provider
  com acesso avançado — agora possível com a verificação).
Alternativa rejeitada: plataforma fornece números (os estúdios querem o seu próprio número).

**D2 — Coexistência vs migração.**
Recomendação: **camada de provider** (`_shared/whatsappProvider.ts`) com interface única;
duas implementações — `EvolutionProvider` (existente) e `CloudApiProvider` (novo). Cloud
API passa a **default**; Evolution fica **legado/fallback por-org** durante a transição e
depois é **descontinuada**. Sem "big bang".

**D3 — Armazenamento de tokens (sensível).**
Recomendação: tokens de acesso por-org **encriptados em repouso** — Supabase **Vault /
pgsodium**, nunca em `VITE_*`, nunca no frontend. App Secret e verify token como secrets
de Edge Function.

**D4 — Estratégia de templates + janela de 24h.**
Recomendação: **registry de templates** (sync dos aprovados na Meta) + as automações
passam a distinguir: dentro da janela de 24h → texto livre; fora → **template** (com
mapeamento de variáveis para os componentes do template). O `automation-scheduler`/
`automation-execute` passam a verificar a janela.

**D5 — Número(s) para arranque.**
Precisamos do inventário Meta para começar: **App ID, App Secret, WABA ID,
Phone Number ID(s), System User token permanente, verify token, versão da Graph API.**

## Arquitetura alvo (resumo)
- **Canal:** novo `whatsapp_channels` (ou extensão de `whatsapp_instances`) com
  `provider ('evolution'|'cloud_api')`, `phone_number_id`, `waba_id`, `access_token`
  (encriptado), `verify_token`, `status`. `organization_id` + RLS.
- **Templates:** `whatsapp_templates(org, name, language, category, status, components)`.
- **Mensagens:** `whatsapp_messages` ganha `provider` + mapeamento de `status`.
- **Provider layer:** `_shared/whatsappProvider.ts` (interface: `sendText`, `sendTemplate`,
  `sendMedia`, `normalizeInbound`), `cloudApiProvider.ts`, `evolutionProvider.ts`.
- **Inbound:** nova edge function `whatsapp-cloud-webhook` — GET verify + POST com
  validação `X-Hub-Signature-256`; normaliza → mesmo downstream (contactos/chats/
  mensagens/automações) já existente.
- **Outbound:** `whatsapp-send`/`automation-*` passam pelo provider layer; regra da
  janela de 24h aplicada.
- **Onboarding UI (Antenor):** ligar canal (manual → Embedded Signup), estado, gestão de
  templates.

## Segurança (obrigatório)
- Validar **sempre** `X-Hub-Signature-256` no inbound (senão qualquer um forja eventos —
  o mesmo risco de spoofing do webhook Evolution, mas aqui a Meta dá-nos a assinatura).
- Tokens **nunca** no frontend; encriptados em repouso; RLS por org.
- Webhook responde 200 sempre; logs estruturados; nunca expor erro interno.
- Isolamento multi-tenant: um número/WABA pertence a uma org; validar sempre.

## Consequências
- (+) Canal **oficial** (sem risco de ban), status de entrega/leitura, templates,
  escalável, self-serve futuro.
- (+) A camada de provider isola a mudança e permite descontinuar a Evolution sem big-bang.
- (−) Nova regra de negócio (janela 24h + templates) muda automações e UX.
- (−) Onboarding Meta (verificação/tokens) é mais burocrático que "scan QR".

## Nota sobre trabalho Evolution pendente
Com esta direção, o **hardening do webhook Evolution (#35)** e a **rotação da chave
Evolution** passam a **baixa prioridade** (a Evolution vira legado). Sugiro **congelar**
esses gates até decidirmos o ritmo da migração.

## Gate
Decisões D1–D5 + migrações + deploys = **gates de Victor**. Nada implementado até
confirmação. Plano faseado de execução em ROADMAP → EPIC C.
