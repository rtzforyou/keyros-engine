# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Step 1 — Inventory all useMockData dependencies
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
```

## Repository to inspect

After reading the engine, inspect:

```text
rtzforyou/easytattoo-crm
```

## Objective

Start Step 1 of the MockData Removal Plan.

Create a complete inventory of every file that still imports or uses `useMockData`.

## Scope

This is an inventory-only task.

You may search, inspect and report.

## Forbidden

Do not change code.

Do not commit.

Do not push.

Do not create migrations.

Do not modify the database.

Do not clean imports.

Do not remove any UI.

Do not continue to Step 2 without Victor's permission.

## Required report

For every result, report:

```text
File:
What it uses from useMockData:
Domain:
Risk:
Recommended action: remove / replace / keep temporarily
Suggested owner: Claude / Antenor
```

Then provide:

```text
Task:
Agent: Claude
Repo:
Branch:
Commit SHA: none, inventory-only
Files inspected:
Verification:
Risks / not verified:
Next isolated task recommended:
Permission requested from Victor: yes
```

## Stop rule

After the inventory, stop and ask Victor for permission before any code change.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
