# Keyros Graph Entry Point

This is the first file every agent should read.

Do not scan the full repository by default. Use this file to choose the shortest path to the right context.

## Product source

- Product repo: `rtzforyou/easytattoo-crm`
- Current business name: Keyros
- Legacy repo/product label: EasyTattoo CRM

## Core nodes

```text
Keyros
├── CRM
│   ├── Contacts
│   ├── Deals
│   └── Pipeline
├── Messaging
│   ├── WhatsApp
│   ├── Conversations
│   └── Quick Messages
├── Automations
│   ├── Triggers
│   └── Actions
├── Calendar
│   ├── Appointments
│   └── Tattooers
├── Finance
│   ├── Client Payments
│   ├── Deals Revenue
│   ├── Expenses
│   ├── Invoices
│   └── Profit
├── App Billing
│   ├── Plans
│   ├── Subscriptions
│   ├── Trials
│   └── Feature Limits
├── Dashboard
│   ├── KPIs
│   ├── Pipeline Funnel
│   └── Revenue vs Expenses
├── Team
│   ├── Invitations
│   ├── Roles
│   └── Permissions
├── Security
│   ├── Auth
│   ├── RLS
│   ├── Rate Limit
│   ├── Idempotency
│   └── Circuit Breaker
└── Infrastructure
    ├── Supabase
    ├── Edge Functions
    ├── Realtime
    └── External Integrations
```

## Non-core / removed from base app

```text
Removed from core
├── Forms
│   └── Reason: replaceable by Google Forms or external form tools for the current product scope
└── Workflows
    └── Reason: too complex for the standard user; may return later as premium/agency feature
```

## Fast routing

- CRM task: read `domains/crm.md` and `rules/business-rules.md`.
- WhatsApp task: read `domains/whatsapp.md`, `domains/messaging.md`, and `rules/security-rules.md`.
- Automation task: read `domains/automations.md` and `rules/business-rules.md`.
- Dashboard or finance task: read `domains/dashboard.md`, `domains/finance.md`, and `rules/business-rules.md`.
- Client payments task: read `domains/payments.md`.
- App billing or subscription task: read `domains/payments.md` and `domains/app-billing.md`.
- Database or schema task: read `architecture/database.md` and any relevant ADR.
- Security task: read `rules/security-rules.md` before touching code.
- New feature task: read `CHANGE_PROTOCOL.md` first.

## Graph rule

Every structural feature must update this graph or explicitly state why the graph is unchanged.
