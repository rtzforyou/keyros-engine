# Email-style Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Remove Forms from core Settings UI — isolated small task
Date: 2026-07-07

## Repository to read first

Read first:

```text
rtzforyou/keyros-engine
```

Mandatory files:

```text
GRAPH.md
AGENT_EXECUTION_PROTOCOL.md
REPORTING_STANDARD.md
TEAM_OPERATING_MODEL.md
PARALLEL_AGENT_WORKFLOW.md
agent-room/decisions/2026-07-07-mockdata-removal-plan.md
agent-room/decisions/2026-07-07-fixed-email-recipient.md
```

## Repository to work on

After Victor approves this task, work on:

```text
rtzforyou/easytattoo-crm
```

## Objective

Remove Forms from the core app UI, because Forms is not part of the current Keyros core scope and can be replaced by Google Forms or external tools.

## Owner

Antenor.

This is a small, isolated UI cleanup task.

## Branch

Use a separate branch:

```text
antenor/remove-forms-ui
```

## Allowed files

Only files directly responsible for showing Forms in Settings/navigation may be touched.

Likely allowed area:

```text
components/settings/
```

If another file is required, stop and ask Victor before editing it.

## Forbidden files / domains

Do not touch:

```text
components/automations/
hooks/useAutomations.ts
hooks/useAppointments.ts
hooks/useContacts.ts
hooks/useTattooers.ts
hooks/useDashboard.ts
supabase/migrations/
supabase/functions/
payments
team members
calendar sync
RLS/security files
```

## Forbidden actions

Do not implement forms.

Do not create form tables.

Do not create migrations.

Do not edit Supabase.

Do not touch Workflows.

Do not modify payments, team, calendar, dashboard or automations.

Do not clean unrelated `useMockData` areas.

## Expected change

Remove Forms from visible base Settings UI.

Remove imports that become unused only if they are directly related to Forms removal.

Keep the change small.

## Verification required

Run the available build/typecheck/lint command if available.

At minimum, verify:

```text
Settings page no longer shows Forms tab/entry.
No broken imports from the removal.
No unrelated files were changed.
```

## Required final report

```text
Task:
Agent: Antenor
Repo:
Branch:
Commit SHA:
PR:
Files changed:
What changed:
Why changed:
Verification performed:
Remote changes: none expected
Keyros Engine updated: no, unless unexpected scope change
Risks / not verified:
Next isolated task recommended:
```

## Stop rule

Do not start this task until Victor explicitly authorizes it.

After finishing, stop and ask Victor before any next task.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
