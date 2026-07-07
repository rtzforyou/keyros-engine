# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Step 4 — Team members real model plan only
Date: 2026-07-07

## Context

Your previous task, the refreshed `useMockData` inventory, was delivered in:

```text
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
```

Status: `needs_review`, but accepted as the current working inventory for task planning.

Antenor is active in the fast lane for small UI/dead-code cleanup. Do not touch his branches or files.

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
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
agent-room/TASK_STATUS_AND_REPORT_BRANCH_RULE.md
agent-room/decisions/2026-07-07-orchestrator-autonomy-rule.md
```

Also read the Team domain file if present:

```text
domains/team.md
```

## Product repository to inspect

```text
rtzforyou/easytattoo-crm
```

## Objective

Step 4 of the MockData Removal Plan:

Create a technical plan to replace the Team area currently using `useMockData` with a real Supabase-backed team/membership/permissions model.

This is a plan-only task.

Do not implement yet.

## Why this is Claude's task

Team membership touches:

- users;
- organizations;
- invitations;
- roles;
- permissions;
- RLS/security boundaries;
- organization isolation;
- invite acceptance flow.

This is heavy/security-sensitive work and must not be assigned to Antenor.

## Files to inspect

Inspect at minimum:

```text
components/team/TeamContent.tsx
hooks/useInvitations.ts
App.tsx
components/auth/Login.tsx
supabase/migrations/
supabase/functions/
types.ts
```

You may inspect additional files only if directly needed for the plan.

## Required analysis

Answer clearly:

1. What exactly in `TeamContent.tsx` still depends on `useMockData`?
2. What tables already exist for users, organizations, invitations and roles?
3. What is already real because of `useInvitations.ts` and the invite-aware signup fix?
4. What is still fake/local/browser-only?
5. What data model is needed for team members and permissions?
6. Should permissions be stored as JSONB, role-derived, or table-based? Compare options.
7. What RLS policies must exist or be adjusted?
8. What migrations would be needed?
9. What frontend hooks/components would change?
10. What edge cases must be tested?

## Forbidden

Do not modify product code.

Do not create migrations.

Do not apply Supabase changes.

Do not push to product `main`.

Do not touch Antenor branches.

Do not remove Forms/Workflow files.

Do not implement Team changes yet.

Do not change auth flow.

## Required deliverable

Create a plan report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
```

Use your own branch in `keyros-engine`:

```text
claude/team-real-model-plan
```

## Required report format

```text
Task: Team real model plan
Agent: Claude
Status: completed / needs_review / blocked
Repo inspected: rtzforyou/easytattoo-crm
Engine branch:
Report path:
Commit SHA:
Files inspected:

Problem:
Current state:
Existing real infrastructure:
Remaining mock/local pieces:
Recommended architecture:
Database changes needed:
RLS/security implications:
Frontend changes needed:
Migration plan:
Testing plan:
Risks:
Rollback plan:
Implementation steps:
Recommended next implementation task:
Permission requested from Victor: yes
```

## Stop rule

After the plan report, stop.

Do not implement until Victor/ChatGPT reviews and assigns an implementation task through this file.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
