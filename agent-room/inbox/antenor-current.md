# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Fast Lane Queue v5 — post-merge verification and safe cleanup
Date: 2026-07-07

## Context

Fast Lane Queue v4 was completed, reviewed, approved and merged.

Merged product PRs:

```text
PR #2 — chore: remove orphaned Forms and Workflow UI files
PR #3 — chore: remove Forms from static module lists
PR #4 — refactor(pipeline): extract formatters from mock data hook
PR #5 — chore(messages): remove inactive mock data import from AI assistant
PR #6 — refactor(automations): remove trigger labels from mock data hook
PR #7 — feat(calendar): use real tattooers hook in Calendar settings
PR #8 — chore(forms): remove orphaned FormFiller component
```

Claude is handling Team/Supabase/RLS work. Do not overlap with Claude.

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Global execution rule

Execute only the current task.

After each task:

1. create the required report in `keyros-engine`;
2. provide product branch + commit SHA if code changed;
3. provide engine branch + report commit SHA;
4. stop;
5. request authorization before starting the next queued task.

Do not automatically continue.

Do not paste long reports into chat. Return only:

```text
Task:
Status:
Product branch:
Product commit:
Report path:
Engine commit:
Permission requested:
```

## Global forbidden areas

Do not touch:

```text
supabase/
RLS/auth/security
hooks/useInvitations.ts
hooks/useAutomations.ts
hooks/useMockData.ts
hooks/useDashboard.ts
hooks/useAppointments.ts
hooks/useContacts.ts
team permissions / TeamContent.tsx
payments architecture / PaymentsContent.tsx
calendar sync engine
businessHours persistence
dashboard financial logic
automation execution logic
Claude branches
```

Do not create migrations.

Do not modify remote services.

---

# Task 11 — Fresh useMockData import audit after merged fast lane

## Status

Current authorized task.

## Objective

Produce a fresh audit of remaining active `useMockData` imports on product `main` after PR #2–#8 were merged.

This is report-only.

## Product branch

None.

## Required check

Search current product `main` for active imports/usages of:

```text
useMockData
```

Classify remaining results by owner:

```text
Claude / Antenor / ChatGPT decision needed
```

Expected likely heavy items include Team, Payments, Calendar businessHours and App loading/error. Verify, do not assume.

## Forbidden

Do not change product code.

Do not edit `useMockData.ts`.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task11-fresh-usemockdata-audit-report.md
```

## Stop after Task 11

Stop and request authorization for Task 12.

---

# Task 12 — Dead imports/types check after Forms/Workflow removal

## Status

Queued. Not authorized until Task 11 is reported and approved.

## Objective

Check whether the removed Forms/Workflow UI left unused imports, unused types, or broken exports in active code.

This is verification-first.

## Product branch

None if report-only.

If a tiny cleanup is clearly safe, use:

```text
antenor/cleanup-dead-forms-workflows-refs
```

## Allowed areas

Inspect only:

```text
components/
types.ts
hooks/
lib/
```

## Forbidden

Do not touch Supabase.

Do not remove form/workflow types unless proven unused and no active DB/API shape depends on them.

Do not touch automation execution logic.

## Verification

Run if code changes:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task12-dead-forms-workflows-refs-report.md
```

## Stop after Task 12

Stop and request authorization for Task 13.

---

# Task 13 — ProfileSettings avatar_url bug check

## Status

Queued. Not authorized until Task 12 is reported and approved.

## Objective

Inspect `ProfileSettings.tsx` avatar upload/update behavior.

Claude found that existing code tried to update `users.avatar_url` before the column existed. PR #9 will add the column, but migration is not applied yet.

This task is check-only unless a UI-side bug is obvious and safe.

## Product branch

None if report-only.

If code change is tiny and safe:

```text
antenor/profile-avatar-url-check
```

## Allowed files

Inspect:

```text
components/settings/ProfileSettings.tsx
```

Optional read-only inspection:

```text
hooks/useAuth.ts
hooks/useUser.ts
```

## Forbidden

Do not touch Supabase/migrations.

Do not change auth flow.

Do not change storage bucket logic unless clearly UI-only and already broken.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task13-profile-avatar-url-check-report.md
```

## Stop after Task 13

Stop and request authorization for Task 14.

---

# Task 14 — UI navigation consistency check after Forms removal

## Status

Queued. Not authorized until Task 13 is reported and approved.

## Objective

Verify that Forms is fully removed from active navigation/UI while landing page integrations remain untouched.

This is report-only unless a tiny dead label remains.

## Product branch

None if report-only.

If code change is tiny and safe:

```text
antenor/forms-navigation-consistency-cleanup
```

## Check active UI only

Search for visible labels/routes/tabs:

```text
Forms
Formulários
Workflows
```

## Forbidden

Do not touch landing page integrations.

Do not touch Supabase.

Do not touch translations unless a visible active dead label is proven.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task14-forms-navigation-consistency-report.md
```

## Stop after Task 14

Stop and request authorization for Task 15.

---

# Task 15 — Build/typecheck health on updated main

## Status

Queued. Not authorized until Task 14 is reported and approved.

## Objective

Run final build/typecheck health on updated `main` after fast-lane merges.

This is report-only.

## Product branch

None.

## Required commands

Run:

```text
npm run build
```

If available and not too slow:

```text
npx tsc --noEmit
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task15-updated-main-build-health-report.md
```

## Stop after Task 15

Stop and request next queue.

---

## Required report format for every task

```text
Task:
Agent: Antenor
Status: completed / needs_review / blocked / failed
Repo:
Product branch:
Product commit SHA:
PR:
Engine branch:
Engine commit SHA:
Files inspected:
Files changed/deleted:
What changed:
Verification performed:
Remote changes:
Risks / not verified:
Next queued task:
Authorization requested before next task: yes
```

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
