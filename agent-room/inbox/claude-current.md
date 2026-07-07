# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Claude Queue — confirm Team plan report, then wait for next heavy task approval
Date: 2026-07-07

## Status correction

You were assigned:

```text
Step 4 — Team members real model plan only
```

Victor says you finished, but ChatGPT cannot currently see the expected report on GitHub at:

```text
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
```

or the expected branch:

```text
claude/team-real-model-plan
```

Therefore, before any new Claude work, you must first make the Step 4 report traceable in `keyros-engine`.

## Current authorized task — Task 0

Confirm and publish the Team real model plan report.

If the report already exists locally:

1. push the branch to GitHub;
2. provide the correct branch name;
3. provide the correct full commit SHA;
4. confirm the report path.

If the report does not exist:

1. recreate it from your completed analysis;
2. commit it on your own engine branch;
3. push it;
4. provide the branch + commit SHA.

## Required report path

```text
agent-room/reports/2026-07-07-claude-team-real-model-plan.md
```

## Required engine branch

```text
claude/team-real-model-plan
```

## Required final response for Task 0

```text
Task: Team real model plan
Agent: Claude
Status: completed / needs_review / blocked
Engine branch:
Report path:
Engine commit SHA:
Files inspected:
Permission requested from Victor: yes
```

## Forbidden during Task 0

Do not modify product code.

Do not create migrations.

Do not apply Supabase changes.

Do not push to product `main`.

Do not touch Antenor branches.

Do not implement Team changes yet.

---

# Next heavy task queue for Claude

The tasks below are queued for planning only. They are not automatically authorized until ChatGPT/Victor reviews the previous report.

## Task 1 — Team real model implementation plan review / implementation slice

Status: queued, not authorized yet.

Possible next step after Task 0:

- if the Team plan is accepted, implement only the first safe slice of the Team real model;
- likely scope: migration + hook plan or migration-only branch, depending on the plan.

Do not start until this file is updated.

## Task 2 — Client payments real model plan

Status: queued, not authorized yet.

Objective:

Create a plan for replacing `components/payments/PaymentsContent.tsx` useMockData dependency with the real client-payments model described in:

```text
domains/payments.md
audits/mock-to-real-backlog.md
```

This is money-related and must be planned by Claude, not Antenor.

Do not start until this file is updated.

## Task 3 — Calendar businessHours schema decision

Status: queued, not authorized yet.

Objective:

Plan how to replace `businessHours` and `updateBusinessHours` in `components/calendar/CalendarSettings.tsx` with real persisted settings.

Do not start until this file is updated.

## Stop rule

For now, complete only Task 0: make the Team plan report visible and traceable in GitHub.

After Task 0, stop and request permission.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
