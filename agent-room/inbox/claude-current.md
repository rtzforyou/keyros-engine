# Current EPIC — Claude / Fable

To: Claude / Fable
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: EPIC ZONE 1 — Team Real Model
Date: 2026-07-07

## Read first

```text
rtzforyou/keyros-engine/OWNERSHIP.md
rtzforyou/keyros-engine/AGENT_EXECUTION_PROTOCOL.md
rtzforyou/keyros-engine/REPORTING_STANDARD.md
```

## EPIC

```text
ZONE 1 — Team Real Model
```

## Objective

Own the Team domain from database foundation to frontend readiness, without crossing production gates.

## Current state

```text
PR #9 is open: feat(team): prepare real users permissions model migration
PR #9 is not merged
Supabase migration is not applied
Task 1 runbook exists
V0 production verification exists
```

## Autonomous permission inside this EPIC

You may continue through sprints automatically up to:

```text
branch
commit
PR draft
report
build/typecheck
audit
runbook
local/static verification
```

## Hard stop gates

Stop and request Victor/ChatGPT authorization before:

```text
applying Supabase migration
changing production data
merging to main
production deploy
changing RLS/Auth beyond the approved PR scope
changing Payments/Calendar/Dashboard
hard delete member semantics
```

## Sprints

### Sprint 1 — PR #9 readiness

Verify PR #9 is still merge-ready after latest main.

Deliver report.

### Sprint 2 — Team apply/test runbook review

Verify the existing runbook is complete and runnable.

Do not apply.

### Sprint 3 — useTeamMembers hook draft

Prepare hook implementation in a draft branch/PR if possible, but do not assume migration is applied.

If hook depends on unapplied DB columns, keep PR draft and mark blocked by PR #9 apply.

### Sprint 4 — TeamContent read-only draft

Wire read-only Team UI only if it can be safely isolated behind the hook and migration dependency.

Otherwise prepare plan/report only.

### Sprint 5 — permissions editing plan/draft

Prepare safe admin role/permissions editing design.

Do not enforce Login yet.

### Sprint 6 — Login permissions enforcement plan

Plan how Login will resolve:

```text
admin -> all active modules
non-admin -> public.users.permissions
```

Do not implement until Team migration + hook + read-only UI are verified.

### Sprint 7 — removeMember soft-remove design

Default is soft remove/revoke access.

Hard delete is not authorized.

## Definition of Done

The EPIC cannot be marked completed unless it includes:

```text
PR #9 readiness status
hook branch/PR or blocked reason
TeamContent read-only branch/PR or blocked reason
permissions editing plan
Login enforcement plan
removeMember soft-remove plan
build/typecheck results where code changed
reports in keyros-engine
risks
rollback plan
next recommended EPIC
```

## Final response format

Do not paste long reports.

Return only:

```text
EPIC:
Status:
Sprints executed:
Branches:
Commits:
PRs:
Reports:
Blocks:
Next recommended EPIC:
```

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
