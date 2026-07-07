# Current EPIC — Antenor

To: Antenor
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: EPIC ZONE 2 — Frontend Health + MockData Elimination
Date: 2026-07-07

## Read first

```text
OWNERSHIP.md
AGENT_EXECUTION_PROTOCOL.md
REPORTING_STANDARD.md
```

## EPIC

```text
ZONE 2 — Frontend Health + MockData Elimination
```

## Objective

Own frontend health and reduce remaining `useMockData` dependencies as far as possible without crossing into backend/security domains.

## You may do automatically inside this EPIC

```text
audit
branch
commit
PR draft
report
build/typecheck
frontend smoke checklist
UI cleanup
unused import cleanup
dead code cleanup
```

## Stop before touching

```text
Supabase
RLS/Auth/security
Team permissions / TeamContent.tsx
Payments architecture / PaymentsContent.tsx
Dashboard financial logic
Calendar schema/sync/businessHours persistence
automation execution logic
hooks/useMockData.ts deletion
main merge
production deploy
```

## Sprints

1. Fresh `useMockData` audit on updated product `main`.
2. Remove frontend-only dead code and unused imports.
3. Verify Forms/Workflows are gone from active core UI navigation.
4. Check `ProfileSettings` avatar flow and report PR #9 dependency if blocked.
5. Run build/typecheck and separate pre-existing errors from new errors.
6. Create or run frontend smoke checklist for Settings, Calendar settings, Pipeline, Messages/AI Assistant, Automations UI, Navigation.
7. Produce final frontend MockData inventory with remaining blockers and owner.

## Definition of Done

Do not mark completed unless you deliver:

```text
fresh useMockData inventory
PR draft(s) for frontend-safe cleanup, if any
build result
typecheck result or explanation
frontend smoke result or limitation
reports in keyros-engine
risk list
remaining blockers with owner
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
