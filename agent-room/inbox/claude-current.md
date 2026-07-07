# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Claude Ultra Queue — sequential heavy tasks with approval gates
Date: 2026-07-07

## Context

Victor is activating you for a heavier execution run.

You may see a long queue, but you must execute only the currently authorized task.

After each task:

1. commit/report;
2. return only summary metadata to Victor/ChatGPT;
3. request authorization;
4. stop.

Do not silently continue to the next task.

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Engine repository

```text
rtzforyou/keyros-engine
```

## Mandatory reading

Read before acting:

```text
GRAPH.md
AGENT_EXECUTION_PROTOCOL.md
REPORTING_STANDARD.md
TEAM_OPERATING_MODEL.md
PARALLEL_AGENT_WORKFLOW.md
agent-room/TASK_STATUS_AND_REPORT_BRANCH_RULE.md
agent-room/decisions/2026-07-07-orchestrator-autonomy-rule.md
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
agent-room/reports/2026-07-07-claude-team-migration-slice-report.md
```

## Current PR status

Team migration PR is open:

```text
PR #9 — feat(team): prepare real users permissions model migration
Branch: claude/team-real-model-migration-slice
Head: 9a2859f4bb6291664a74ad9810b969537f85271a
```

ChatGPT opened the PR after Victor approval.

Important:

```text
PR opened/approved for review readiness only.
Supabase production migration is NOT approved yet.
Do not apply migration until a later explicit task says so.
```

---

# Global rules for this ultra queue

Execute only one task at a time.

Do not apply Supabase migrations unless the current task explicitly authorizes it.

Do not merge PRs unless the current task explicitly authorizes it.

Do not touch Antenor branches.

Do not touch unrelated product areas.

For every task, create/update a report in:

```text
keyros-engine/agent-room/reports/
```

Final response to Victor/ChatGPT must be short:

```text
Task:
Status:
Product branch:
Product commit:
PR:
Report path:
Engine commit:
Permission requested:
```

Do not paste long reports into chat.

---

# Task 0 — PR #9 merge-readiness check

## Status

Current authorized task.

## Objective

Check PR #9 for merge readiness after the latest main updates.

This is review/check only.

## Required checks

Verify:

```text
PR #9 still has exactly one migration file
no frontend/hooks/components changed
no Supabase production apply happened
migration includes guard against non-admin self-escalation
migration includes rollback notes in report
```

## Forbidden

Do not merge PR #9.

Do not apply migration.

Do not modify product code unless PR #9 is stale/broken and requires a small rebase/revision.

If a revision is needed, stop and report `needs_revision`.

## Report path

```text
agent-room/reports/2026-07-07-claude-task0-pr9-merge-readiness-report.md
```

## Stop after Task 0

Stop and request authorization for Task 1.

---

# Task 1 — Apply/test plan for Team migration

## Status

Queued. Not authorized until Task 0 is reviewed.

## Objective

Create an exact apply/test/rollback runbook for PR #9 migration.

No production apply yet unless explicitly authorized later.

## Required output

Include SQL/test plan for:

```text
normal signup without invite
signup with pending invite
member updates own full_name/avatar_url -> allowed
member updates own permissions -> blocked
member updates own role -> blocked
member updates own organization_id -> blocked
admin updates member permissions -> allowed
admin updates member role -> allowed
rollback plan
```

## Report path

```text
agent-room/reports/2026-07-07-claude-task1-team-migration-apply-test-runbook.md
```

## Stop after Task 1

Stop and request authorization for Task 2.

---

# Task 2 — Apply Team migration and verify in Supabase

## Status

Queued. Not authorized until Victor explicitly approves production apply.

## Objective

Apply PR #9 migration to Supabase and run the approved verification plan.

## Forbidden until explicit approval

Do not run this task just because it is listed here.

Only execute if Victor/ChatGPT explicitly says:

```text
Approve Task 2 — apply Team migration to Supabase
```

## Report path

```text
agent-room/reports/2026-07-07-claude-task2-team-migration-apply-verification-report.md
```

## Stop after Task 2

Stop and request authorization for Task 3.

---

# Task 3 — Team members hook plan/read-only implementation

## Status

Queued. Not authorized until migration is applied and verified.

## Objective

Create `hooks/useTeamMembers.ts` and, if authorized in that task, wire read-only Team members listing.

Prefer split if risk is high:

```text
Task 3A: hook only
Task 3B: TeamContent read-only wiring
```

## Forbidden

Do not change Login permission enforcement yet.

Do not implement removeMember yet.

Do not modify RLS unless explicitly scoped.

## Report path

```text
agent-room/reports/2026-07-07-claude-task3-use-team-members-report.md
```

## Stop after Task 3

Stop and request authorization for Task 4.

---

# Task 4 — Team role/permissions editing

## Status

Queued. Not authorized until Team read-only listing works.

## Objective

Allow admin to update team member role and permissions using the real `public.users` model.

## Forbidden

Do not change Login enforcement yet unless explicitly included.

Do not implement remove member yet.

## Report path

```text
agent-room/reports/2026-07-07-claude-task4-team-edit-role-permissions-report.md
```

## Stop after Task 4

Stop and request authorization for Task 5.

---

# Task 5 — Login permission enforcement plan and implementation

## Status

Queued. Not authorized until Team editing works.

## Objective

Stop granting all modules blindly in Login.

Implement real permission resolution:

```text
admin -> all active modules
non-admin -> permissions from public.users.permissions
```

## Required caution

Avoid locking users out.

Keep an emergency fallback plan.

## Report path

```text
agent-room/reports/2026-07-07-claude-task5-login-permission-enforcement-report.md
```

## Stop after Task 5

Stop and request authorization for Task 6.

---

# Task 6 — removeMember decision and soft-remove design

## Status

Queued. Not authorized until permission enforcement is stable.

## Objective

Design and implement the first safe remove member path.

Default recommendation:

```text
soft remove / disabled / revoked access
```

Hard delete via service_role Edge Function is not authorized unless Victor explicitly chooses it.

## Report path

```text
agent-room/reports/2026-07-07-claude-task6-team-remove-member-report.md
```

## Stop after Task 6

Stop and request authorization for Task 7.

---

# Task 7 — Client payments real model plan

## Status

Queued. Not authorized until Team critical path is stable or Victor reprioritizes.

## Objective

Plan replacement of `components/payments/PaymentsContent.tsx` mock dependency with real client payments model.

Separate clearly:

```text
client payments
app billing/subscription
expenses
financial dashboard
```

## Report path

```text
agent-room/reports/2026-07-07-claude-task7-client-payments-real-model-plan.md
```

## Stop after Task 7

Stop and request authorization for Task 8.

---

# Task 8 — Business Hours schema plan

## Status

Queued. Not authorized until Team/Payments priority is decided.

## Objective

Plan persisted business hours for `CalendarSettings.tsx`.

Compare:

```text
organization_business_hours table
organization_settings JSONB
hybrid
```

## Report path

```text
agent-room/reports/2026-07-07-claude-task8-business-hours-schema-plan.md
```

## Stop after Task 8

Stop and request next queue.

---

## Final instruction

Current authorized task is Task 0 only.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
