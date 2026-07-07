# Antenor Fast Lane Rule

Date: 2026-07-07

## Decision

Antenor may receive a small-task queue instead of waiting idle after every tiny verification.

This optimizes execution speed while keeping Claude responsible for heavy/risky domains.

## Why

Antenor completes small isolated tasks very quickly. Making him wait for Claude after every small task slows the project unnecessarily.

## Fast lane scope

Antenor may work on tasks that are:

- UI-only;
- dead-code cleanup;
- import cleanup;
- label/tab cleanup;
- documentation/reporting;
- verification-only;
- no database impact;
- no RLS/security impact;
- no Supabase/Edge Function impact;
- no business logic rewrite.

## Forbidden for Antenor fast lane

Antenor must not touch:

- Supabase migrations;
- RLS/security/auth;
- payments architecture;
- team permissions model;
- automation execution engine;
- calendar sync engine;
- dashboard financial logic;
- broad `useMockData` removal;
- files assigned to Claude in the current cycle.

## Execution rule

Antenor can receive a numbered queue.

He must execute one task at a time, in order.

For each task:

1. create a separate branch;
2. make the narrow change or verification;
3. run build/typecheck if relevant;
4. create/report in `agent-room/reports/`;
5. provide commit SHA and status;
6. only continue to the next queued task if the previous task had no conflict, no unexpected files, and no high-risk finding.

If any conflict, unexpected file, build failure or scope expansion appears, Antenor must stop.

## Branch naming

Use one branch per task:

```text
antenor/<small-task-name>
```

## Status values

```text
completed
needs_review
blocked
failed
```

## Current goal

Keep Antenor productive with low-risk cleanup while Claude completes the full `useMockData` inventory.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
