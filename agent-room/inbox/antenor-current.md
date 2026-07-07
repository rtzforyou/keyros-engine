# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Fast Lane Queue — small isolated cleanups while Claude inventories useMockData
Date: 2026-07-07

## Context

You completed:

```text
Step 2 — Remove Forms from core Settings UI
Step 3 — Verify Workflows hidden from base Automations UI
```

Claude is still working on the full `useMockData` inventory.

Victor approved optimizing your workflow so you do not stay idle while Claude works.

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
```

## Fast lane rule

You may execute the queue below one task at a time.

For each task:

1. create a separate branch;
2. keep the change narrow;
3. run build/typecheck if relevant;
4. create a report in `keyros-engine/agent-room/reports/`;
5. provide branch + commit SHA;
6. continue to the next queued task only if there is no conflict, no unexpected file, and no scope expansion.

If anything unexpected appears, stop and report `blocked`.

## Product repository

```text
rtzforyou/easytattoo-crm
```

---

# Task A — Remove dead Workflow UI files

## Objective

Delete dead Workflow UI components that are no longer imported by the base app after Workflows was removed from Automations UI.

## Branch

```text
antenor/remove-dead-workflow-ui
```

## Allowed files

Only these files may be deleted:

```text
components/automations/WorkflowTabContent.tsx
components/automations/WorkflowDialog.tsx
components/automations/WorkflowItemCard.tsx
```

## Required verification before deletion

Before deleting, confirm again that no active file imports these components.

At minimum search for:

```text
WorkflowTabContent
WorkflowDialog
WorkflowItemCard
```

## Forbidden

Do not modify:

```text
components/automations/AutomationsContent.tsx
hooks/useAutomations.ts
hooks/useMockData.ts
supabase/
```

Do not change automation execution logic.

Do not remove workflow types from `types.ts` in this task.

Do not touch translation files in this task.

## Report path

```text
agent-room/reports/2026-07-07-antenor-remove-dead-workflow-ui-report.md
```

---

# Task B — Verify remaining Forms permission references

Execute Task B only after Task A is clean/completed.

## Objective

Verify where `forms` still appears in active module/permission lists after Forms UI was removed.

This is verification-first.

## Branch

If no code change is needed:

```text
none in product; report only
```

If a tiny cleanup is clearly safe:

```text
antenor/verify-forms-permission-refs
```

## Files to inspect

```text
App.tsx
components/auth/Login.tsx
```

## Allowed action

Report only unless the only remaining change is removing `'forms'` from a hardcoded module list.

If you remove `'forms'`, touch only:

```text
App.tsx
components/auth/Login.tsx
```

## Forbidden

Do not change auth flow.

Do not change invitation RPCs.

Do not modify `useInvitations.ts`.

Do not touch Supabase.

Do not change roles/permissions model beyond removing dead `forms` string from static module arrays.

## Report path

```text
agent-room/reports/2026-07-07-antenor-forms-permission-refs-report.md
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
