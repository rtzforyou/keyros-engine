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
│   ├── Actions
│   └── Workflows
├── Calendar
│   ├── Appointments
│   └── Tattooers
├── Finance
│   ├── Deals Revenue
│   ├── Expenses
│   ├── Payments
│   └── Profit
├── Dashboard
│   ├── KPIs
│   ├── Pipeline Funnel
│   └── Revenue vs Expenses
├── Team
│   ├── Invitations
│   ├── Roles
│   └── Permissions
├── Forms
│   ├── Templates
│   └── Submissions
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

## Fast routing

- CRM task: read `domains/crm.md` and `rules/business-rules.md`.
- WhatsApp task: read `domains/whatsapp.md`, `domains/messaging.md`, and `rules/security-rules.md`.
- Automation task: read `domains/automations.md` and `rules/business-rules.md`.
- Dashboard or finance task: read `domains/dashboard.md`, `domains/finance.md`, and `rules/business-rules.md`.
- Database or schema task: read `architecture/database.md` and any relevant ADR.
- Security task: read `rules/security-rules.md` before touching code.
- New feature task: read `CHANGE_PROTOCOL.md` first.

## Graph rule

Every structural feature must update this graph or explicitly state why the graph is unchanged.
