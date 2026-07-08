# UPDATE TASK

Status: ACTIVE
Date: 2026-07-08
Owner: ChatGPT Orchestrator

## Purpose

This is the lightweight latest-command file.

When Victor writes `START`, agents must read this file first and treat it as the freshest instruction layer.

If this file conflicts with older inbox content, this file wins.

## Current truth

- Product PR #9 is already merged.
- Merge commit: `541ea2c7f79da85e4c3d35f83f613c173bdeb569`
- The next hard gate is Supabase migration apply.
- Do not report PR #9 as open or waiting for merge.

## Claude / Fable — next task

EPIC: ZONE 1 — Team Real Model Completion

1. Read the migration apply runbook:
   - `agent-room/reports/2026-07-08-claude-team-migration-apply-runbook.md`
2. Confirm migration target and checklist.
3. Stop before applying Supabase migration unless Victor explicitly authorized migration apply in the current run.
4. After migration apply is authorized and completed:
   - run Team validation checklist
   - verify normal signup
   - verify invited signup
   - verify member profile edit
   - verify protected Team fields cannot be changed by a normal member
   - verify admin member update inside same organization
   - report rollback status
5. After validation, prepare PR #12 review path.
6. Then prepare next EPIC plan:
   - Team Permissions Editing
   - Billing Entitlements + Team Seat Limits

Do not implement Stripe yet.
Do not change Payments production behavior yet.

## Antenor — next task

EPIC: ZONE 2 — Frontend Health + MockData Elimination Continuation

1. Stop reporting PR #9 as open.
2. Continue frontend work that does not depend on unapplied Supabase migration.
3. Continue MockData elimination.
4. Prepare authenticated smoke tests:
   - login
   - dashboard
   - settings
   - team page read-only state
   - navigation
   - no white screen
   - no critical console error
5. Prepare frontend UX plan or draft for Billing Entitlements without implementing Stripe:
   - current plan indicator
   - seat usage indicator
   - upgrade required state
   - invite blocked by seat limit state
   - upgrade CTA
6. Do not touch backend access rules.
7. Do not change Payments gateway logic.
8. Stop before main merge or production deploy.

## Global rule

Agents should advance in parallel:

- Claude owns backend, database, Team validation, and Billing architecture.
- Antenor owns frontend health, UX readiness, mock cleanup, and smoke tests.

Agents must stop only at hard gates and keep reports short.
