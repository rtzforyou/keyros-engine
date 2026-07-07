# Priority 2 Strong Code Review

Purpose: define the required deep code review for mixed real/fake areas before continuing feature expansion.

## Scope

Priority 2 areas need a strong review because they are partially real and partially dependent on mock/local/fallback behavior.

Areas:

1. Calendar
2. Automations
3. Dashboard cleanup
4. Remaining `useMockData` dependencies

## 1. Calendar review

Current risk:

- Appointments are mostly real.
- Some supporting data still comes from `useMockData`, especially contacts/tattooers in parts of the UI.

Review checklist:

- Replace mock contacts dependency with `useContacts` or direct lookup.
- Replace mock tattooers dependency with `useTattooers` everywhere.
- Verify create/update/delete appointment flows.
- Verify business hours validation.
- Verify double-booking/conflict logic.
- Verify Google Calendar is not marketed as two-way sync unless inbound sync is implemented.
- Verify organization-scoped queries and RLS.

## 2. Automations review

Current risk:

- Automation CRUD is mostly real.
- Trigger options still come from `useMockData`.
- Workflows were removed from the base UI and must stay out of core unless premium/agency scope is approved.

Review checklist:

- Remove `automationTriggers` dependency from `useMockData`.
- Create a real trigger registry in code or database.
- Confirm canonical trigger names.
- Confirm execution logs.
- Confirm idempotency.
- Confirm retry behavior.
- Confirm delay execution.
- Confirm anti-spam/rate-limit behavior for WhatsApp automations.

## 3. Dashboard review

Current risk:

- Dashboard is mostly real through `useDashboard`.
- Legacy fallback logic still supports deals without `stage_id`.

Review checklist:

- Migrate all deals to `stage_id`.
- Stop using text `stage` as logic source.
- Keep `stage` only as display label if still needed.
- Confirm financial KPIs use `financial_transactions`.
- Confirm closed/won/lost logic uses canonical financial status.
- Confirm lead sources use the intended source of truth.

## 4. useMockData retirement review

Current risk:

- `useMockData` still centralizes unrelated state.
- It is hard to know which parts of the app are real.

Review checklist:

- List all remaining imports of `useMockData`.
- Classify each import as remove, replace, or temporarily keep.
- Remove Forms and Workflows dependencies first.
- Replace Payments with real client payments/app billing models.
- Replace Team members with real membership data.
- Replace Calendar supporting data with real hooks.
- Replace Automation trigger list with real registry.

## Rule

No new feature should be built on top of `useMockData`.
