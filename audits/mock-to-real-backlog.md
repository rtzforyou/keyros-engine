# Mock to Real Backlog

Purpose: list Keyros tabs and functions that still depend on mock/local/fallback behavior and must be converted into real product flows.

Source product repo inspected: `rtzforyou/easytattoo-crm`.

## Priority 0 — Rule for every future change

Every new feature must answer:

1. Does this change the Keyros Graph?
2. Does this change a business rule?
3. Does this change database, security, automation, integration or finance?
4. Does `CONTEXT_INDEX.md` need an update?
5. Does this require an ADR?

## Priority 1 — Must become real before commercial use

### Payments tab

Status: mostly fake/local.

Evidence:

- `PaymentsContent.tsx` imports `useMockData`.
- Payments, contacts, `addPayment` and `formatCurrency` come from `useMockData`.
- Payment success only calls `addPayment`, which creates local state with `pay-Date.now()`.
- Export CSV opens a not-implemented alert.
- Payout status is estimated as `stats.total * 0.95`.

Required implementation:

- Create real payments/invoices/transactions data model.
- Persist payment records in Supabase.
- Link payment to organization, contact, deal and optional appointment.
- Replace fake Stripe checkout simulation with real payment provider flow or mark as manual payment until provider is ready.
- Replace estimated payout with real payout/fee records or hide until integrated.
- Add audit logs for refunds and payment status changes.

### Forms settings / consent forms

Status: fake/local.

Evidence:

- `FormsSettings.tsx` imports `useMockData`.
- Form templates are initialized inside `useMockData` with a hardcoded generic consent form.
- Template CRUD is local state.
- File upload and signature are marked coming soon.

Required implementation:

- Create real `form_templates`, `form_template_versions`, and `form_submissions` flows.
- Store template versions immutably once used.
- Persist client submissions with organization and contact/deal links.
- Implement signature/file fields or keep disabled with explicit product status.
- Add permission rules for who can create/archive forms.

### Workflows tab

Status: fake/local.

Evidence:

- `WorkflowTabContent.tsx` imports `useMockData`.
- Workflows use local `addWorkflow`, `updateWorkflow`, `deleteWorkflow`, `toggleWorkflowStatus`.

Required implementation:

- Decide whether workflows are separate from automations or just multi-step automations.
- Create real database model if they remain separate.
- Add execution engine, status, logs, retries and failure handling.
- Connect workflows to automation event logs.

### Team members / permissions

Status: mixed; invitations are real, members/permission editing still fake/local.

Evidence:

- `TeamContent.tsx` uses real `useInvitations` for invitations.
- The same file uses `useMockData` for `teamMembers`, `updateTeamMember`, and `removeTeamMember`.
- `useInvitations.ts` persists invitations in Supabase.

Required implementation:

- Replace `teamMembers` from `useMockData` with real `users` / membership table.
- Persist role and permissions.
- Validate permissions server-side, not only in the frontend.
- Ensure invite acceptance updates real membership and cannot be forged with only a URL token.

## Priority 2 — Mixed real/fake; clean architecture required

### Calendar tab

Status: mostly real appointments, but still depends on mock data for contacts/tattooers in parts of the UI.

Evidence:

- `CalendarContent.tsx` uses real `useAppointments` and `useGoogleCalendar`.
- It also imports `useMockData` for `contacts` and `tattooers`.
- `useAppointments.ts` persists appointments in Supabase and triggers automation after creation.

Required implementation:

- Replace contacts dependency with `useContacts` or direct contact lookup.
- Replace tattooers dependency with `useTattooers` everywhere.
- Ensure Google Calendar sync is not described as two-way unless inbound sync is implemented.
- Confirm delete/update appointment flows are real, not view-only.

### Automations tab

Status: mostly real automations, but trigger options and workflows still depend on mock/local behavior.

Evidence:

- `AutomationsContent.tsx` uses real `useAutomations` for automation CRUD.
- It still imports `useMockData` for `automationTriggers`.
- `WorkflowTabContent.tsx` is local/mock.
- `useAutomations.ts` persists automations to Supabase and maps frontend fields to database fields.

Required implementation:

- Move trigger definitions to a real central registry or database-backed enum.
- Remove trigger list from `useMockData`.
- Connect workflows or delete the workflows tab until it is real.
- Confirm execution backend handles delay, idempotency, retry and logs.

### Dashboard tab

Status: mostly real, but legacy fallback exists in `useMockData` and must be removed from production-critical paths.

Evidence:

- `DashboardContent.tsx` uses `useDashboard`, not `useMockData`.
- `useDashboard.ts` fetches real Supabase data for stages, deals, financial transactions, appointments, contacts, lead submissions, WhatsApp messages and automation logs.
- `useDashboard.ts` still has fallback grouping for deals without `stage_id`.

Required implementation:

- Keep `useDashboard` as the canonical dashboard source.
- Remove old dashboard calculations from `useMockData` once no component depends on them.
- Migrate all deals to `stage_id` and stop relying on text `stage` fallback.
- Confirm all financial KPIs use `financial_transactions`, not legacy `expenses`.

## Priority 3 — Real but should be reviewed

### Contacts tab

Status: mostly real.

Evidence:

- `ContactsContent.tsx` uses `useContacts`.
- `useContacts.ts` fetches from Supabase, handles realtime, deduplicates by phone, links WhatsApp chats and persists create/update/delete.

Required review:

- Confirm import CSV is real and not only browser-local.
- Confirm delete contact should be hard delete or soft delete.
- Confirm all contact operations are organization-scoped and RLS protected.

### Appointments core

Status: mostly real.

Evidence:

- `useAppointments.ts` fetches appointments from Supabase by authenticated organization.
- It inserts appointments and triggers automation on appointment creation.

Required review:

- Implement real update/delete appointment actions if missing.
- Confirm conflict checks and business hours validation.
- Confirm appointment status transitions.

## Global technical debt

### `useMockData.ts`

Status: should be split and retired.

Problems:

- It centralizes too many unrelated domains.
- It mixes real Supabase reads with local fallback state.
- It still owns fake/local CRUD for payments, forms, workflows, team members, expenses, deals and parts of messaging.
- It creates IDs with `Date.now()` for several product entities.

Required plan:

1. Freeze `useMockData` — no new feature should depend on it.
2. Create one real hook per domain.
3. Remove domain by domain from `useMockData`.
4. Delete `useMockData` only after all imports are gone.

## Recommended implementation order

1. Payments real model and persistence.
2. Team members and permissions real model.
3. Forms real persistence and versioning.
4. Workflows decision: implement as real multi-step automations or remove tab.
5. Calendar cleanup: remove mock contacts/tattooers dependency.
6. Automations cleanup: real trigger registry and execution guarantees.
7. Remove legacy dashboard/fallback calculations from `useMockData`.
8. Retire `useMockData` completely.
