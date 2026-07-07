# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Correction — complete missing Task A before any new fast-lane work
Date: 2026-07-07

## Status correction

Your reported Task B is received:

```text
Clean remaining Forms module permission strings
Product commit: 72deedd3fab6728a52163d344ed6bc259d5f2ad3
```

ChatGPT verified that Task B changed only:

```text
App.tsx
components/auth/Login.tsx
```

and removed only the dead `'forms'` module string from `ALL_MODULES`.

However, Task A from the fast-lane queue is not confirmed as completed on product `main`.

At least this file still exists on `main` and still imports `useMockData`:

```text
components/settings/FormsSettings.tsx
```

Therefore, the fast-lane queue is not fully complete yet.

## Current authorized task

Complete the missing Task A only.

Do not start any other task.

---

# Task A — Remove orphaned Forms/Workflow UI files

## Objective

Delete orphaned UI files that are no longer imported by active app code and still pull from `useMockData`.

This is a pure dead-code deletion task.

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
App.tsx
components/auth/Login.tsx
```

Do not change automation execution logic.

Do not remove workflow types from `types.ts` in this task.

Do not touch translation files in this task.

Do not touch Forms permission strings. Task B already handled that separately.

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

## Required final report

```text
Task: Remove orphaned Forms/Workflow UI files
Agent: Antenor
Status: completed / blocked / failed
Repo:
Product branch:
Product commit SHA:
PR:
Engine branch:
Engine commit SHA:
Files inspected:
Files deleted:
Verification performed:
Remote changes:
Risks / not verified:
Next queued task status: none until ChatGPT updates inbox
Permission requested from Victor: yes
```

## Stop rule

After this task, stop.

Do not start another task until this file is updated by ChatGPT.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
