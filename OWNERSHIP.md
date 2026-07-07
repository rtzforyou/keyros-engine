# Keyros Agent Ownership Model

Date: 2026-07-07

## Purpose

Keyros agents now work by ownership and EPICs, not only by small isolated tasks.

## Victor

Victor owns business direction and final approval for risky actions.

## ChatGPT

ChatGPT owns orchestration, architecture review, GitHub verification, scope control, and maintaining `keyros-engine`.

A task is confirmed only when these are available and verified:

```text
Task
Product commit, if product code changed
Engine commit
Report path
```

## Claude / Fable

Claude/Fable owns heavy domains:

```text
Backend
Supabase
Database schema
RLS
Auth
Permissions
Team model
Payments
Calendar schema
Dashboard financial logic
Integrations
Security-sensitive migrations
```

Claude/Fable may run large EPICs but must stop before migration apply, production deploy, main merge, data deletion, or high-risk security decisions unless explicitly authorized.

## Antenor

Antenor owns frontend health:

```text
UI
UX
Components
Pages
Layouts
Icons
Styles
Dead code cleanup
Unused imports
Build/typecheck health
Frontend smoke checks
Local refactors
Frontend documentation
PR drafts
```

Antenor must stop before Supabase, RLS, Auth, Team permissions, Payments, Dashboard financial logic, Calendar schema/sync, automation execution logic, production deploy, or main merge.

## Operating model

Old:

```text
Task -> report -> wait
```

New:

```text
EPIC -> Sprints -> Tasks -> reports -> PR drafts -> review
```

Agents continue through all sprints inside their authorized EPIC unless they hit a stop condition.

## EPIC Definition of Done

An EPIC cannot be marked `completed` unless it includes:

```text
branch or branches
commit SHA(s)
PR draft(s), if code changed
build result
typecheck result or explanation
report in keyros-engine
evidence of verification
what was not verified
risk assessment
rollback plan
next recommended EPIC
```

If anything is missing, status must be `needs_review`, `incomplete`, or `blocked`, not `completed`.

## Current split

```text
Claude/Fable -> Zone 1 Team and heavy backend/security domains
Antenor -> Zone 2 MockData/frontend health and UI QA/refactor domains
ChatGPT -> Orchestration, review, GitHub verification, merge decisions
Victor -> Business approvals and risky authorization gates
```

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
