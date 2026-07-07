# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Fast Lane Queue v2 — dead-code cleanup from Claude useMockData inventory
Date: 2026-07-07

## Context

Claude delivered the refreshed `useMockData` inventory:

```text
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
```

Status: `needs_review`.

The inventory confirmed several now-orphaned UI files that still import `useMockData` but are no longer imported by active app code.

Victor wants Antenor optimized in a fast lane of small isolated tasks, not waiting idle for Claude.

## Read first

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
```

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Fast lane rule

Execute one task at a time.

Each task must have its own product branch and its own report.

Continue to the next queued task only if:

- no conflict;
- no unexpected files;
- no build failure caused by your change;
- no scope expansion;
- no overlap with Claude.

If anything unexpected appears, stop and report `blocked`.

---

# Task A — Remove orphaned Forms/Workflow UI files

## Objective

Delete orphaned UI files that are no longer imported by active app code and still pull from `useMockData`.

This is a pure dead-code deletion task.

## Branch

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

Before deleting, search for active imports/usages of:

```text
FormsSettings
ContactFormsTab
WorkflowTabContent
WorkflowDialog
WorkflowItemCard
```

If any file is still imported by active app code, do not delete it. Stop and report `blocked`.

## Forbidden

Do not modify:

```text
components/automations/AutomationsContent.tsx
hooks/useAutomations.ts
hooks/useMockData.ts
types.ts
supabase/
```

Do not change automation execution logic.

Do not remove workflow types from `types.ts` in this task.

Do not touch translation files in this task.

Do not touch Forms permission strings in `App.tsx` or `Login.tsx` in Task A.

## Verification required

Run:

```text
npm run build
```

If available and fast enough, also run:

```text
npx tsc --noEmit
```

If typecheck still has pre-existing errors, report them clearly and confirm no new errors were introduced by deleted files.

## Report path

Create report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-antenor-remove-orphaned-mock-ui-files-report.md
```

Use your own engine branch for the report:

```text
antenor/remove-orphaned-mock-ui-files-report
```

---

# Task B — Clean remaining Forms module permission strings

Execute Task B only after Task A is clean/completed.

## Objective

Remove dead `'forms'` strings from static module/permission lists after Forms UI was removed from the core app.

## Branch

```text
antenor/remove-forms-module-strings
```

## Files to inspect

```text
App.tsx
components/auth/Login.tsx
```

## Allowed change

Remove only `'forms'` from hardcoded module arrays/lists.

Allowed files to change:

```text
App.tsx
components/auth/Login.tsx
```

## Forbidden

Do not change auth flow.

Do not change invitation RPCs.

Do not modify `useInvitations.ts`.

Do not touch Supabase.

Do not change roles/permissions model beyond removing the dead `forms` string from static module arrays.

Do not touch any other file.

## Report path

Create report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-antenor-remove-forms-module-strings-report.md
```

Use your own engine branch:

```text
antenor/remove-forms-module-strings-report
```

---

## Required report format for each task

```text
Task:
Agent: Antenor
Status: completed / needs_review / blocked / failed
Repo:
Branch:
Commit SHA:
PR:
Files inspected:
Files changed:
What changed:
Why changed:
Verification performed:
Remote changes:
Risks / not verified:
Next queued task status:
Permission requested from Victor: yes/no
```

## Stop conditions

Stop immediately if:

- a file outside the allowed list must be touched;
- the build fails for a new reason;
- there is a merge conflict;
- Claude's task overlaps with yours;
- a task starts touching data, Supabase, security, automation execution, payments, team, dashboard or calendar.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
