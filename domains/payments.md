# Payments Domain

Payments in Keyros are split into two different domains.

They must not share the same business logic, tables or dashboard meaning.

## 1. Client payments

Money paid by the studio's clients to the studio.

This belongs to the user's business data.

Examples:

- tattoo deposit;
- session payment;
- invoice paid by a client;
- manual bank transfer;
- cash payment;
- card payment recorded manually;
- future external provider payment.

Client payments feed:

- finance dashboard;
- gross revenue;
- outstanding invoices;
- deal financial status;
- contact history;
- appointment/deal records.

Initial implementation rule:

- Manual/Supabase-first.
- No Stripe dependency now.
- Provider fields can exist as nullable future-ready fields.
- Payment must always belong to an organization.
- Payment should link to contact and optionally deal/appointment.

## 2. App billing / subscriptions

Money paid by Keyros customers to use the Keyros app.

This belongs to Keyros platform operations, not to the user's client finance.

Examples:

- monthly subscription;
- annual subscription;
- trial;
- plan limits;
- app access status;
- failed subscription payment;
- future provider subscription id.

App billing controls:

- organization plan;
- trial status;
- subscription status;
- limits;
- access rules;
- feature availability.

Initial implementation rule:

- Keep separate from user financial transactions.
- Do not show app subscription revenue in the user's dashboard.
- Do not mix app invoices with client invoices.
- Provider integration is abstract and can be connected later.

## Stripe status

Stripe is in standby.

Do not implement Stripe now.

The current goal is to create correct boundaries and database structure so Stripe can be added later without rewriting finance.

## Naming rule

Use explicit names:

- `client_payments` or `financial_transactions` for user business payments.
- `billing_subscriptions` / `organization_subscriptions` for app subscriptions.

Avoid generic `payments` when the domain is ambiguous.
