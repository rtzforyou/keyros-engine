# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Resume Step 1 — Complete useMockData inventory only
Date: 2026-07-07

## Context

Claude is back online after the credit interruption.

Before the interruption, Claude fixed the high-risk invite-aware signup bug and delivered the required report in:

```text
agent-room/reports/2026-07-07-claude-invite-aware-signup-report.md
```

That task is separate and must not be mixed with this one.

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

Resume and complete Step 1 of the MockData Removal Plan.

Create a complete inventory of every file that still imports or uses `useMockData` after the latest merges.

## Branch rule

This is an inventory/report task only.

Use a branch in `keyros-engine` for the report:

```text
claude/usemockdata-inventory-report
```

Do not create a product-code branch for this task.

## Scope

You may:

- inspect files;
- search for `useMockData` imports/usages;
- classify remaining risks;
- write a report in `keyros-engine`.

## Forbidden

Do not change product code.

Do not commit to `rtzforyou/easytattoo-crm`.

Do not push to product `main`.

Do not create migrations.

Do not modify Supabase.

Do not remove Forms.

Do not clean imports.

Do not implement any fix discovered during the inventory.

Do not touch Antenor's branches.

## Required report path

Create the report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
```

## Required inventory format

For every result, report:

```text
File:
What it uses from useMockData:
Domain:
Risk: low / medium / high
Recommended action: remove / replace / keep temporarily
Suggested owner: Claude / Antenor / ChatGPT decision needed
Suggested isolated task:
```

Then provide a summary:

```text
Total files using useMockData:
High-risk items:
Medium-risk items:
Low-risk items:
Already resolved since previous backlog:
New issues discovered:
Next recommended task for Claude:
Next recommended task for Antenor:
```

## Required final status

```text
Task: useMockData inventory
Agent: Claude
Status: completed / blocked / needs_review
Repo inspected: rtzforyou/easytattoo-crm
Engine branch:
Report path:
Commit SHA:
Files inspected:
Verification:
Risks / not verified:
Permission requested from Victor: yes
```

## Stop rule

After delivering the report, stop.

Do not start any implementation until Victor/ChatGPT assigns the next task through `agent-room/inbox/claude-current.md`.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
