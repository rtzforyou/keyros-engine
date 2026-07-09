# ADR-0006 — Billing / Stripe (EPIC D)

Estado: **Proposto — design + scaffold de backend implementado. Aguarda decisões
de Victor (planos/preços/chaves) para ativar.**
Data: 2026-07-10 · Autor: Claude / Fable
Contexto: comercialização do CRM. Precisamos de subscrições pagas (Stripe) com
entitlements por plano, multi-tenant. Relacionado: EPIC D (ROADMAP), NEXT.md.

## Contexto
O CRM é multi-tenant. Cada organização (estúdio) deve poder subscrever um plano.
Já existe scaffolding frontend (`BillingSettings`, `StripeConfigDialog`,
`useBillingLimits`) mas sem um fluxo de checkout/subscrição real ligado ao Stripe.

## Decisões

**D1 — Modelo de dados.** Duas tabelas:
- `billing_plans` (global, config): code, name, price_cents, currency, interval,
  `stripe_price_id`, `features` (entitlements), seats_included, is_active.
- `organization_subscriptions` (por org): plan_id, stripe_customer_id,
  stripe_subscription_id, status, seats, current_period_end, cancel_at_period_end.
RLS: planos ativos legíveis por autenticados; subscrição legível só pela própria
org; **escrita só service_role** (o webhook do Stripe).

**D2 — Fluxo.** Stripe Checkout (hosted). `create-checkout-session` (verify_jwt=true)
resolve o plano → cria Checkout Session com `metadata.organization_id` (nunca do
payload; sempre do contexto autenticado). `stripe-webhook` (verify_jwt=false)
valida a assinatura `Stripe-Signature` (HMAC-SHA256, Web Crypto, sem SDK) e
atualiza `organization_subscriptions` em `checkout.session.completed` e
`customer.subscription.updated/deleted`.

**D3 — Segredos.** `STRIPE_SECRET_KEY` e `STRIPE_WEBHOOK_SECRET` como secrets de
Edge Function (nunca `VITE_*`, nunca no frontend). `stripe_price_id` é config
(não segredo) em `billing_plans`.

**D4 — Entitlements.** `billing_plans.features` (jsonb) define módulos/limites por
plano. Enforcement lê a subscrição ativa da org + features do plano. (Integra com
o Module Registry / `useBillingLimits` já existentes.)

**D5 — Sem SDK no edge.** Chamadas ao Stripe via REST (form-encoded) + verificação
de assinatura via Web Crypto — evita dependência pesada/instável no Deno edge.

## Decisões PENDENTES de Victor (bloqueiam a ativação)
1. **Planos + preços** (ex.: Starter / Pro / Studio) e o que cada um inclui.
2. **Mensal / anual** (ou ambos).
3. **Por-estúdio (flat) ou por-seat**.
4. **Chaves Stripe de TESTE** primeiro (`sk_test_...`, `whsec_...`) + criar os
   Products/Prices no Stripe e preencher `stripe_price_id` nos `billing_plans`.

## Estado de implementação (nesta ronda)
- Migração `20260715000000_billing_stripe.sql` (schema + RLS) — **draft, não aplicada**.
- `supabase/functions/stripe-webhook` — **código pronto, não deployed**.
- `supabase/functions/create-checkout-session` — **código pronto, não deployed**.
- Frontend `BillingSettings`/`useBillingLimits` — a ligar ao checkout (follow-up de UI).
- Nada ativado: `billing_plans` fica vazio/is_active=false até Victor confirmar.

## Segurança (obrigatório)
- Validar SEMPRE a assinatura do webhook (senão qualquer um forja eventos de
  subscrição e "ativa" planos de graça).
- `organization_id` do checkout vem do contexto autenticado, nunca do payload.
- Escrita das subscrições só por service_role (webhook); RLS só leitura por org.
- Chaves só em secrets do servidor.

## Gate
Migração + deploys + chaves = **gate de Victor**. Ativação depende das decisões D1–D4.
