# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Fast Lane Queue v3 — five small sequential tasks with approval gates
Date: 2026-07-07

## Context

Victor approved a faster Antenor workflow.

You may receive multiple small tasks in a queue, but you must execute them **one at a time**.

Important: after each task, you must stop, report, and request authorization before starting the next one.

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
useAppointments/useContacts/useDashboard internals
payments architecture
team permissions/auth/RLS
calendar sync engine
dashboard financial logic
```

Do not create migrations.

Do not modify remote services.

Do not change automation execution logic.

Do not touch Claude branches.

---

# Task 1 — Remove orphaned Forms/Workflow UI files

## Status

Current authorized task.

## Objective

Delete orphaned UI files that are no longer imported by active app code and still pull from `useMockData`.

This is pure dead-code deletion.

## Product branch

```text
antenor/remove-orphaned-mock-ui-files
```

## Allowed files

Only these files may be deleted:

```text
components/settings/FormsSettings.tsx
components/contacts/ContactFormsTab.tsx
components/automations/WorkflowTabContent.tsx
components/automations/WorkflowDialog.tsx
components/automations/WorkflowItemCard.tsx
```

## Required verification before deletion

Search for active imports/usages of:

```text
FormsSettings
ContactFormsTab
WorkflowTabContent
WorkflowDialog
WorkflowItemCard
```

If any file is still imported by active app code, stop and report `blocked`.

## Verification

Run:

```text
npm run build
```

If available and fast enough, also run:

```text
npx tsc --noEmit
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task1-remove-orphaned-mock-ui-files-report.md
```

## Stop after Task 1

After Task 1, stop and ask authorization for Task 2.

---

# Task 2 — Extract Pipeline formatters from useMockData

## Status

Queued, not authorized until Task 1 is reported and approved.

## Objective

Remove `useMockData` dependency from `components/pipeline/PipelineContent.tsx` if it is only used for formatting helpers.

Claude inventory says PipelineContent uses only:

```text
formatDistanceToNow
formatCurrency
```

This should become a tiny utility extraction, not a business-logic change.

## Product branch

```text
antenor/extract-pipeline-formatters
```

## Allowed files

Only touch:

```text
components/pipeline/PipelineContent.tsx
lib/formatters.ts
```

If `lib/formatters.ts` does not exist, you may create it.

## Forbidden

Do not change pipeline logic.

Do not modify deals, stages, drag/drop, dashboard, Supabase, hooks, or `useMockData.ts`.

Do not change currency behavior except moving the existing formatter behavior to a utility.

## Verification

Run:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task2-extract-pipeline-formatters-report.md
```

## Stop after Task 2

After Task 2, stop and ask authorization for Task 3.

---

# Task 3 — Replace AI Assistant quickMessages mock usage with static local list

## Status

Queued, not authorized until Task 2 is reported and approved.

## Objective

Remove `useMockData` dependency from `components/messages/AIAssistantDialog.tsx` if it is only used for `quickMessages`.

This should be a UI/static-data cleanup only.

## Product branch

```text
antenor/ai-assistant-static-quick-messages
```

## Allowed files

Only touch:

```text
components/messages/AIAssistantDialog.tsx
```

Optional only if strictly needed:

```text
lib/quickMessages.ts
```

## Forbidden

Do not touch WhatsApp sending.

Do not touch message storage.

Do not touch Supabase.

Do not modify automations.

Do not modify `useMockData.ts`.

Do not change assistant behavior beyond removing the mock-data dependency for quick message suggestions.

## Verification

Run:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task3-ai-assistant-static-quick-messages-report.md
```

## Stop after Task 3

After Task 3, stop and ask authorization for Task 4.

---

# Task 4 — Automations trigger registry plan/check

## Status

Queued, not authorized until Task 3 is reported and approved.

## Objective

Inspect `components/automations/AutomationsContent.tsx` and identify how `automationTriggers` from `useMockData` is used.

This is verification/plan-first.

Do not implement unless the change is only moving a static trigger list into a local code registry with no behavior change.

## Product branch

If code change is needed and safe:

```text
antenor/automation-trigger-static-registry
```

If not changing code, no product branch.

## Allowed files if implementation is clearly static-only

```text
components/automations/AutomationsContent.tsx
lib/automationTriggers.ts
```

## Forbidden

Do not modify:

```text
hooks/useAutomations.ts
supabase/
automation execution functions
trigger execution logic
automation logs
RLS/security
```

If the trigger registry looks like it should be DB-backed or affect execution, stop and report `blocked` for Claude/ChatGPT decision.

## Verification

Run build only if code changes:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task4-automation-trigger-registry-report.md
```

## Stop after Task 4

After Task 4, stop and ask authorization for Task 5.

---

# Task 5 — CalendarSettings tattooers mock usage plan/check

## Status

Queued, not authorized until Task 4 is reported and approved.

## Objective

Inspect `components/calendar/CalendarSettings.tsx` and classify the remaining `useMockData` usage into:

1. tattooers — possibly replaceable by existing `hooks/useTattooers.ts`;
2. businessHours — likely requires schema decision and must not be implemented by Antenor.

This is verification/plan-first.

## Product branch

If only a safe tattooers swap is possible and isolated:

```text
antenor/calendar-settings-tattooers-real-hook
```

If businessHours blocks safe implementation, do not change code.

## Allowed files if implementation is clearly tattooers-only

```text
components/calendar/CalendarSettings.tsx
```

## Forbidden

Do not touch businessHours logic if no real table/hook exists.

Do not create migrations.

Do not modify Supabase.

Do not touch calendar sync.

Do not modify appointments logic.

Do not change `hooks/useTattooers.ts` unless ChatGPT/Victor explicitly authorizes it later.

## Verification

Run build only if code changes:

```text
npm run build
```

## Report path

```text
agent-room/reports/2026-07-07-antenor-task5-calendar-settings-tattooers-check-report.md
```

## Stop after Task 5

After Task 5, stop and request the next queue from ChatGPT/Victor.

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
