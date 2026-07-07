# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Fast Lane Queue v4 — continue small tasks after CalendarSettings check
Date: 2026-07-07

## Context

You completed Fast Lane Queue v3 through Task 5.

Latest confirmed report:

```text
agent-room/reports/2026-07-07-antenor-task5-calendar-settings-tattooers-check-report.md
```

Key conclusion from Task 5:

```text
Tattooers can likely be migrated using hooks/useTattooers.ts.
Business Hours is blocked because no real table/hook/RLS exists yet.
```

Business Hours is now Claude-level work. Do not implement it.

## Read first

Read:

```text
rtzforyou/keyros-engine
```

Mandatory files:

```text
AGENT_EXECUTION_PROTOCOL.md
REPORTING_STANDARD.md
PARALLEL_AGENT_WORKFLOW.md
agent-room/decisions/2026-07-07-antenor-fast-lane-rule.md
agent-room/TASK_STATUS_AND_REPORT_BRANCH_RULE.md
agent-room/decisions/2026-07-07-orchestrator-autonomy-rule.md
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
agent-room/reports/2026-07-07-antenor-task5-calendar-settings-tattooers-check-report.md
```

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Global execution rule

Execute only the current task.

After finishing each task:

1. create the required report;
2. provide product branch + commit SHA if code changed;
3. provide engine branch + report commit SHA;
4. stop;
5. request authorization before starting the next queued task.

Do not automatically continue to the next task.

## Global forbidden areas

Do not touch:

```text
supabase/
hooks/useAutomations.ts
hooks/useMockData.ts
hooks/useInvitations.ts
hooks/useDashboard.ts
hooks/useAppointments.ts
hooks/useContacts.ts
payments architecture
team permissions/auth/RLS
calendar sync engine
dashboard financial logic
automation execution logic
```

Do not create migrations.

Do not modify remote services.

Do not touch Claude branches.

---

# Task 6 — CalendarSettings tattooers-only real hook implementation

## Status

Current authorized task.

## Objective

Replace only the `tattooers` part of `components/calendar/CalendarSettings.tsx` with the existing real hook:

```text
hooks/useTattooers.ts
```

Do not touch `businessHours` or `updateBusinessHours` yet.

Business Hours remains blocked for Claude/schema decision.

## Product branch

```text
antenor/calendar-settings-tattooers-real-hook
```

## Allowed files

Only touch:

```text
components/calendar/CalendarSettings.tsx
```

Do not modify `hooks/useTattooers.ts` unless the task becomes blocked and you request permission first.

## Allowed change

Replace mock-sourced tattooers operations:

```text
tattooers
addTattooer
removeTattooer
```

with real hook operations from:

```text
hooks/useTattooers.ts
```

Adapt names/signatures only inside `CalendarSettings.tsx`.

## Forbidden

Do not touch:

```text
businessHours
updateBusinessHours
supabase/
hooks/useTattooers.ts
hooks/useMockData.ts
appointments/calendar sync
```

Do not create migrations.

Do not change persisted schema.

## Verification

Run:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task6-calendar-settings-tattooers-real-hook-report.md
```

## Stop after Task 6

After Task 6, stop and ask authorization for Task 7.

---

# Task 7 — Verify product main still contains useMockData imports after Antenor branches

## Status

Queued, not authorized until Task 6 is reported and approved.

## Objective

Produce a fresh, narrow verification of remaining `useMockData` imports on product `main` after Antenor's latest branches/merges.

This is report-only.

## Product branch

None.

## Required check

Search current product `main` for:

```text
useMockData
```

List only active files that import it.

Do not include reports, docs or node_modules.

## Forbidden

Do not change product code.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task7-current-usemockdata-imports-check-report.md
```

## Stop after Task 7

After Task 7, stop and ask authorization for Task 8.

---

# Task 8 — Check dead form/filler leftovers after Forms removal

## Status

Queued, not authorized until Task 7 is reported and approved.

## Objective

Inspect whether `components/forms/` still contains active code used anywhere after Forms was removed from core.

This is verification-first.

## Product branch

None if report-only.

If deletion is clearly safe and only deletes orphaned Forms UI files, use:

```text
antenor/remove-dead-form-components
```

## Files/folders to inspect

```text
components/forms/
components/settings/
components/contacts/
```

## Forbidden

Do not delete anything that is still imported.

Do not touch form-related types in `types.ts`.

Do not touch Supabase.

Do not touch landing page integrations.

Do not touch `LandingIntegrationsSettings.tsx`.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task8-dead-form-components-check-report.md
```

## Stop after Task 8

After Task 8, stop and ask authorization for Task 9.

---

# Task 9 — UI label scan for Forms/Workflows dead navigation

## Status

Queued, not authorized until Task 8 is reported and approved.

## Objective

Search active UI/navigation files for leftover visible labels related to removed core modules:

```text
Forms
Workflows
```

This is verification-first.

## Product branch

None if no code change.

If only dead labels are removed from active navigation, use:

```text
antenor/remove-dead-forms-workflows-labels
```

## Allowed areas

Only inspect UI/navigation/config files.

## Forbidden

Do not touch translation files unless a visible active label is proven dead and removal is explicitly safe.

Do not touch automation execution logic.

Do not touch Supabase.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task9-forms-workflows-label-scan-report.md
```

## Stop after Task 9

After Task 9, stop and ask authorization for Task 10.

---

# Task 10 — Small build health report for active fast-lane branches

## Status

Queued, not authorized until Task 9 is reported and approved.

## Objective

Produce a build health summary for Antenor fast-lane branches that are not yet merged.

This is report-only.

## Product branch

None.

## Required output

List:

```text
branch
commit SHA
purpose
build status
merge readiness
risks
```

## Forbidden

Do not change product code.

Do not merge.

Do not open PR unless ChatGPT/Victor explicitly asks.

## Report path

```text
agent-room/reports/2026-07-07-antenor-task10-fast-lane-build-health-report.md
```

## Stop after Task 10

After Task 10, stop and request the next queue from ChatGPT/Victor.

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
Why changed:
Verification performed:
Remote changes:
Risks / not verified:
Next queued task:
Authorization requested before next task: yes
```

## Final rule

Speed is good, but no silent continuation.

One task, one branch, one report, one authorization gate.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
