# Current Task — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Step 3 — Verify Workflows are fully hidden from base Automations UI
Date: 2026-07-07

## Context

Your previous task, `Remove Forms from core Settings UI`, was completed and merged into product `main` through PR #1.

Do not repeat that task.

Claude is back online and has been assigned the full `useMockData` inventory. Do not touch Claude's task.

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
agent-room/TASK_STATUS_AND_REPORT_BRANCH_RULE.md
agent-room/decisions/2026-07-07-orchestrator-autonomy-rule.md
```

## Product repository to inspect

```text
rtzforyou/easytattoo-crm
```

## Objective

Step 3 of the MockData Removal Plan:

Confirm that the Workflows tab is fully hidden/removed from the base Automations UI after the previous merge.

This is a verification-first task.

## Branch rule

Use a separate branch only if a code change is necessary:

```text
antenor/verify-workflows-hidden
```

If no code change is needed, do not create a product commit. Create only the report in `keyros-engine` on your own report branch.

## Scope

You may inspect only the Automations UI area and direct imports related to Workflows visibility.

Likely files to inspect:

```text
components/automations/AutomationsContent.tsx
components/automations/WorkflowTabContent.tsx
```

## Allowed action

Preferred outcome:

- verification report only, if Workflows is already hidden.

Small code cleanup allowed only if you find a visible Workflows tab still active in the base UI.

## Forbidden

Do not modify automation execution logic.

Do not modify `hooks/useAutomations.ts`.

Do not modify Supabase.

Do not create migrations.

Do not touch payments, team, calendar, dashboard, contacts or pipeline.

Do not remove `WorkflowTabContent.tsx` unless ChatGPT/Victor explicitly approves deletion after your report.

Do not touch `useMockData` broadly; Claude is inventorying it.

## Verification checklist

Confirm:

```text
AutomationsContent.tsx does not render a Workflows tab.
AutomationsContent.tsx does not import WorkflowTabContent.
Base user cannot access Workflows from Automations UI.
No unrelated files changed.
```

## Required report path

Create report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-antenor-workflows-hidden-verification-report.md
```

Use your own branch in `keyros-engine` for the report:

```text
antenor/workflows-hidden-verification-report
```

## Required final report

```text
Task: Verify Workflows hidden from base Automations UI
Agent: Antenor
Status: completed / needs_review / blocked
Repo inspected: rtzforyou/easytattoo-crm
Product branch: none, if no code change / antenor/verify-workflows-hidden, if code change
Product commit SHA: none, if no code change
Engine branch:
Report path:
Engine commit SHA:
Files inspected:
Files changed:
What changed:
Verification performed:
Remote changes:
Risks / not verified:
Next isolated task recommended:
Permission requested from Victor: yes
```

## Stop rule

After the verification/report, stop.

Do not start any implementation beyond the narrow Workflows visibility check.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
