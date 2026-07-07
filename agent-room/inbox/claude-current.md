# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Step 4A — Team real model migration-only implementation slice
Date: 2026-07-07

## Context

Your Team real model plan is now visible and verified:

```text
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
Engine commit: c96b7f23632828ac5a5f73c50827baaa567efc15
Status: needs_review
```

ChatGPT reviewed the plan and authorizes only the first safe implementation slice.

This task is migration-only.

Do not implement frontend yet.

Do not apply the migration to Supabase production yet.

## Repository to read first

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
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
agent-room/TASK_STATUS_AND_REPORT_BRANCH_RULE.md
agent-room/decisions/2026-07-07-orchestrator-autonomy-rule.md
```

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Objective

Create the first Team real model implementation slice:

A migration-only branch that prepares `public.users` for real team members and permissions.

## Product branch

```text
claude/team-real-model-migration-slice
```

## Allowed files

Only add one new migration file under:

```text
supabase/migrations/
```

Do not touch any frontend file in this task.

Do not modify existing migrations.

Do not modify hooks/components.

## Migration scope

Prepare `public.users` for real team member data.

The migration may include:

```text
ALTER TABLE public.users ADD COLUMN IF NOT EXISTS permissions jsonb NOT NULL DEFAULT '[]'::jsonb;
ALTER TABLE public.users ADD COLUMN IF NOT EXISTS email varchar(255);
ALTER TABLE public.users ADD COLUMN IF NOT EXISTS avatar_url text;
```

It may update `public.handle_new_user()` so future signups/invite signups set:

```text
email = new.email
permissions = matched_invite.permissions for invited users
permissions = admin default/all appropriate default for normal admin signup, if justified
```

It may include safe backfill statements for existing `public.users` rows only if non-destructive and clearly explained.

## Critical restrictions

Do not apply the migration to Supabase production.

Do not use `apply_migration`.

Do not run destructive SQL.

Do not create DELETE policies yet.

Do not implement remove-member semantics yet.

Do not create `useTeamMembers.ts` yet.

Do not change `TeamContent.tsx` yet.

Do not change `Login.tsx` yet.

Do not touch Antenor branches.

## Required verification

Because this is migration-only and not applied remotely, verify by:

1. inspecting migration syntax carefully;
2. comparing with existing `handle_new_user()` implementation;
3. explaining expected behavior for:
   - normal signup without invite;
   - signup with pending invite;
   - existing users;
4. confirming rollback strategy.

If you can safely validate locally without touching production, report how.

Do not claim production validation unless it was actually done and authorized.

## Required report path

Create report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-claude-team-migration-slice-report.md
```

Use your own engine branch:

```text
claude/team-migration-slice-report
```

## Required final report

```text
Task: Team real model migration-only slice
Agent: Claude
Status: completed / needs_review / blocked / failed
Product repo:
Product branch:
Product commit SHA:
Migration file:
Engine branch:
Engine commit SHA:
Files changed:
What changed:
Why changed:
Verification performed:
Remote changes: none expected
Supabase applied: no
Risks / not verified:
Rollback plan:
Next recommended Team slice:
Permission requested from Victor: yes
```

## Stop rule

After this migration-only slice, stop.

Do not apply the migration.

Do not implement frontend.

Do not continue to Team hook/UI without a new task in this file.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
