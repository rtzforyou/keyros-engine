# Decision — MockData Removal Plan

## Objective

Remove or replace every remaining `useMockData` dependency from the Keyros product app without stress, without broad refactors, and without mixing unrelated changes.

Product repo:

```text
rtzforyou/easytattoo-crm
```

Knowledge repo:

```text
rtzforyou/keyros-engine
```

## Core rule

Execute one isolated change at a time.

After each step, stop and ask Victor for permission before starting the next step.

Do not fix all mock data at once.

Do not refactor globally.

Do not touch unrelated files.

## Current known mock/local areas

Based on the current backlog, the main remaining areas are:

1. Forms
2. Workflows
3. Team members / permissions
4. Payments
5. Calendar supporting data
6. Automations trigger list
7. Dashboard legacy fallback / old calculations
8. Remaining `useMockData` imports

## Execution order

### Step 1 — Inventory only

Goal:

List every current import/use of `useMockData` in `easytattoo-crm`.

Allowed action:

- Search only.
- No code change.

Expected output:

```text
File:
What it uses from useMockData:
Domain:
Recommended action: remove / replace / keep temporarily
```

Stop after this and ask Victor for approval.

---

### Step 2 — Remove Forms from core UI

Goal:

Remove Forms UI from the base app because it is not core and can be replaced by Google Forms/external tools.

Allowed action:

- Remove Forms tab/entry from Settings or navigation where applicable.
- Remove active imports that become unused.

Forbidden:

- Do not implement form tables.
- Do not create form migrations.
- Do not build a form builder.

Stop after this and ask Victor for approval.

---

### Step 3 — Confirm Workflows removed from base Automations

Goal:

Verify that Workflows is no longer visible in the base Automations UI and no active base UI import remains.

Allowed action:

- Remove leftover workflow UI references if any remain.
- Do not delete future/premium files unless explicitly approved.

Forbidden:

- Do not implement workflows now.
- Do not create workflow database models.

Stop after this and ask Victor for approval.

---

### Step 4 — Team members real model plan

Goal:

Prepare the real Team Members / Permissions implementation plan.

Allowed action:

- Inspect current Team code.
- Inspect current database/schema/migrations if available.
- Propose exact file list and schema/RLS changes.

Forbidden:

- Do not implement yet.
- Do not create migrations yet.

Stop after this and ask Victor for approval.

---

### Step 5 — Team members implementation

Goal:

Replace fake/local team members with real membership data.

Expected outcome:

- Real membership hook.
- Real role/permission persistence.
- Invite acceptance connected to membership or clearly documented if not implemented in this step.
- No frontend-only security assumption.

Forbidden:

- Do not mix with payments.
- Do not mix with calendar.
- Do not mix with automations.

Stop after this and ask Victor for approval.

---

### Step 6 — Payments plan

Goal:

Plan the split between client payments and app billing.

Allowed action:

- Inspect current Payments code.
- Inspect finance tables/migrations.
- Propose exact manual/Supabase-first implementation.

Forbidden:

- Do not implement Stripe now.
- Do not mix app billing with client payments.

Stop after this and ask Victor for approval.

---

### Step 7 — Client payments implementation

Goal:

Replace local/fake payment state with real client payment records.

Expected outcome:

- Manual payment record flow.
- Organization-scoped records.
- Link to contact/deal/appointment where supported.
- No Stripe dependency.

Stop after this and ask Victor for approval.

---

### Step 8 — Calendar supporting data cleanup

Goal:

Remove Calendar dependency on mock contacts/tattooers.

Expected outcome:

- Calendar uses real `useContacts` or equivalent real lookup.
- Calendar uses real `useTattooers` or equivalent real lookup.

Forbidden:

- Do not implement full Google Calendar sync in this step.
- Do not touch security migrations unless specifically approved.

Stop after this and ask Victor for approval.

---

### Step 9 — Automations trigger registry

Goal:

Replace `automationTriggers` from `useMockData` with a real central trigger registry.

Expected outcome:

- Trigger options do not come from `useMockData`.
- Canonical trigger names are clear.
- Workflows stay out of base UI.

Forbidden:

- Do not rebuild the automation execution engine in this step.

Stop after this and ask Victor for approval.

---

### Step 10 — Dashboard fallback cleanup

Goal:

Remove old dashboard calculations and fallback logic from production-critical paths.

Expected outcome:

- `useDashboard` remains canonical.
- `stage_id` is the logic source for deals.
- `stage` remains display-only if still needed.
- Financial KPIs use the correct financial source of truth.

Stop after this and ask Victor for approval.

---

### Step 11 — Retire useMockData

Goal:

After all domains are removed/replaced, delete or quarantine `useMockData`.

Allowed action:

- Only after zero active imports remain.
- Remove dead types/state/actions.
- Confirm build passes.

Stop and produce final report.

## Required report after every step

Each step must report:

```text
Task:
Agent:
Repo:
Branch:
Commit SHA:
Files changed:
What changed:
Why changed:
Verification:
Remote changes:
Risks / not verified:
Next recommended isolated step:
Permission requested from Victor: yes/no
```

## Boss approval rule

After each step, the agent must stop and ask Victor before continuing.

No automatic continuation.

No batching.

No hidden extra cleanup.
