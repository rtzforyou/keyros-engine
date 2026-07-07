# Keyros Context Index

Use this file after `GRAPH.md`.

It tells agents where to look without reading everything.

## Fast paths

| Task | Read first | Then read |
|---|---|---|
| New feature | `CHANGE_PROTOCOL.md` | affected domain file |
| CRM / contacts / deals | `domains/crm.md` | `rules/business-rules.md` |
| Pipeline stages | `domains/crm.md` | `architecture/database.md` |
| WhatsApp / messages | `domains/whatsapp.md` | `rules/security-rules.md` |
| Automations | `domains/automations.md` | `rules/business-rules.md` |
| Dashboard | `domains/dashboard.md` | `domains/finance.md` |
| Expenses / profit / payments | `domains/finance.md` | `rules/business-rules.md` |
| Calendar / appointments | `domains/calendar.md` | `domains/crm.md` |
| Team / permissions | `domains/team.md` | `rules/security-rules.md` |
| Database / schema | `architecture/database.md` | relevant ADR |
| Supabase / Edge Functions | `architecture/supabase.md` | `rules/security-rules.md` |
| Agent behavior | `AGENTS.md` | `CLAUDE.md` |

## Current confirmed product facts

- Frontend uses Vite, React, TypeScript, Supabase client and Recharts.
- Main app modules include dashboard, contacts, pipeline, calendar, messages, expenses, automations, payments, team and settings.
- Supabase is used through `lib/supabase.ts`.
- WhatsApp sending goes through Supabase Edge Function `whatsapp-send`.
- `stageId` should be treated as logical source of truth for deals; `stage` is for display.

## Unknowns to verify before major work

- Exact current database schema.
- Current deployed Edge Function versions.
- Current RLS policies.
- Current automation execution backend.
- Whether mock/fallback data still exists in production-critical flows.
