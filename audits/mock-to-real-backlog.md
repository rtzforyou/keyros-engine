# Mock to Real Backlog

Purpose: list Keyros tabs and functions that still depend on mock/local/fallback behavior and must be converted, removed, or deferred.

Source product repo inspected: `rtzforyou/easytattoo-crm`.

## Priority 0 — Rule for every future change

Every new feature must answer:

1. Does this change the Keyros Graph?
2. Does this change a business rule?
3. Does this change database, security, automation, integration or finance?
4. Does `CONTEXT_INDEX.md` need an update?
5. Does this require an ADR?

## Priority 1 — Must become real before commercial use

### Payments tab — split into two different domains

Status: mostly fake/local today, but must be split before implementation.

There are two different payment domains and they must not be mixed:

1. Client payments inside the user's business
   - Money paid by the studio's clients to the studio.
   - Belongs to the user's business data.
   - Feeds revenue, invoices, receipts, deals, dashboard and finance.
   - Should work manually first: cash, bank transfer, card, external link, Twint/other future methods.

2. App billing / subscriptions
   - Money paid by Keyros customers to use the Keyros app.
   - Belongs to Keyros platform operations, not to the user's client finance.
   - Controls plans, subscription status, trials, limits and access.
   - Should be designed separately from client payments.

Stripe status: standby.

Do not implement Stripe now. The immediate goal is to design the database and product boundaries so Stripe can be added later without rewriting the finance model.

Evidence:

- `PaymentsContent.tsx` imports `useMockData`.
- Payments, contacts, `addPayment` and `formatCurrency` come from `useMockData`.
- Payment success only calls `addPayment`, which creates local state with `pay-Date.now()`.
- Export CSV opens a not-implemented alert.
- Payout status is estimated as `stats.total * 0.95`.

Required implementation for client payments:

- Create real client-side payment/invoice/transaction model in Supabase.
- Persist payment records by organization.
- Link payment to contact, deal and optional appointment.
- Support manual payment methods first.
- Mark external provider fields as nullable/future-ready.
- Hide Stripe checkout simulation until provider integration is active.
- Replace estimated payout with either real fee/payout data or hide it.
- Add audit logs for refunds and payment status changes.

Required implementation for app billing:

- Create separate platform billing model.
- Track organization plan, subscription status, trial status and limits.
- Do not mix app subscriptions with the user's `financial_transactions`.
- Add access control based on plan/status later.
- Keep provider integration abstract so Stripe can be added later.

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

## Priority 1B — Remove from core app instead of implementing now

### Forms settings / consent forms

Decision: remove from core app for now.

Reason:

- Not essential to the current Keyros core.
- Can be replaced by Google Forms or external form tools.
- Real implementation would require form templates, versioning, submissions, file uploads, signature handling and permission logic.
- Current value does not justify current complexity.

Current status:

- `FormsSettings.tsx` imports `useMockData`.
- Form templates are initialized inside `useMockData` with a hardcoded generic consent form.
- Template CRUD is local state.
- File upload and signature are marked coming soon.

Required action:

- Remove Forms tab from Settings.
- Remove or hide Forms-related UI from the base product.
- Remove Forms from core graph.
- Keep no active dependency on `useMockData` for forms.
- Optional future: allow external form URL integration only.

### Workflows tab

Decision: remove from core app for normal users.

Reason:

- Too complex for the standard Keyros user.
- Risks confusing users who mainly need simple automations.
- Current implementation is fake/local.

Premium possibility:

- Workflows may return later as an advanced premium or agency-only feature, especially for marketing companies.
- If implemented later, it should be positioned as advanced automation builder, not as a basic CRM feature.

Current status:

- `WorkflowTabContent.tsx` imports `useMockData`.
- Workflows use local `addWorkflow`, `updateWorkflow`, `deleteWorkflow`, `toggleWorkflowStatus`.

Required action now:

- Remove Workflows tab from Automations for base users.
- Remove Workflows from core graph.
- Keep simple automations as the core automation model.
- Do not create real workflow database model now.

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

Status: mostly real automations, but trigger options still depend on mock/local behavior.

Evidence:

- `AutomationsContent.tsx` uses real `useAutomations` for automation CRUD.
- It still imports `useMockData` for `automationTriggers`.
- `WorkflowTabContent.tsx` is local/mock and should be removed from base app.
- `useAutomations.ts` persists automations to Supabase and maps frontend fields to database fields.

Required implementation:

- Move trigger definitions to a real central registry or database-backed enum.
- Remove trigger list from `useMockData`.
- Remove Workflows tab from base UI.
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
3. Remove forms and workflows from the UI instead of implementing them now.
4. Remove domain by domain from `useMockData`.
5. Delete `useMockData` only after all imports are gone.

## Recommended implementation order

1. Split payments into client payments and app billing.
2. Implement client payments manually first, without Stripe.
3. Implement app billing model separately, with provider integration in standby.
4. Remove Forms from core app.
5. Remove Workflows from base app; keep as future premium/agency hypothesis.
6. Team members and permissions real model.
7. Calendar cleanup: remove mock contacts/tattooers dependency.
8. Automations cleanup: real trigger registry and execution guarantees.
9. Remove legacy dashboard/fallback calculations from `useMockData`.
10. Retire `useMockData` completely.
