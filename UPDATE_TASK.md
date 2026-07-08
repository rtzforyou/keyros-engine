# UPDATE TASK

Status: ACTIVE
Date: 2026-07-08
Owner: ChatGPT Orchestrator

## Purpose

This is the lightweight latest-command file.

When Victor writes `START`, agents must read this file first and treat it as the freshest instruction layer.

If this file conflicts with older inbox content, this file wins.

## Current truth

- Product PR #9 is merged.
- PR #9 merge commit: `541ea2c7f79da85e4c3d35f83f613c173bdeb569`
- Team migration has been applied in production by Claude.
- Remote migration record reported by Claude: `20260708144838`
- Repo migration file: `supabase/migrations/20260711000000_team_users_columns_and_signup.sql`
- Structural checks passed: new Team columns, email backfill, Team guard trigger.
- Simulated guard checks passed: normal member blocked from protected Team changes, admin same organization allowed, own full_name edit allowed.
- Product PR #12 is ready for review and is the next merge gate.
- Do not report PR #9 as open.
- Do not report the Team migration as waiting for apply.

## Newly approved architecture

Canonical Module Registry is now approved as an official Keyros architecture layer.

The registry is the single source of truth for:

- Team member permissions
- Plan capabilities
- Billing entitlements
- Feature flags
- Future automation and AI module access

Agents must not invent module ids ad hoc.

## Canonical module draft

Initial draft for review and implementation planning:

- `dashboard`
- `crm`
- `contacts`
- `deals`
- `calendar`
- `messages`
- `automations`
- `ai`
- `payments`
- `reports`
- `settings`
- `team`

Claude may refine names if needed, but must report any change clearly.

## Claude / Fable — next tasks

EPIC: ZONE 1 — Team Real Model Completion

1. Review product PR #12 after migration apply.
2. Confirm whether PR #12 can be merged safely.
3. Report branch, commit SHA, PR number, test result, and risk list.
4. Stop before PR #12 merge unless Victor or ChatGPT explicitly authorizes merge.

Next EPIC after PR #12 path:

EPIC: Canonical Module Registry

1. Create final module registry proposal.
2. Define stable module ids.
3. Define display labels separately from ids.
4. Define which modules can be assigned as Team permissions.
5. Define which modules can be controlled by plan entitlements.
6. Define dependencies between modules if any.
7. Prepare implementation plan for Team Permissions Editing.
8. Prepare implementation plan for Billing Entitlements + Team Seat Limits.

Do not implement Stripe yet.
Do not change Payments production behavior yet.

## Antenor — next tasks

EPIC: ZONE 2 — Frontend Health + MockData Elimination Continuation

1. Stop reporting PR #9 as open.
2. Stop reporting Team migration as waiting for apply.
3. Continue frontend work that does not require PR #12 to be merged.
4. Continue MockData elimination.
5. Keep authenticated smoke tests updated:
   - login
   - dashboard
   - settings
   - team page read-only state
   - navigation
   - no white screen
   - no critical console error
6. Continue Billing UX draft without implementing Stripe:
   - current plan indicator
   - seat usage indicator
   - upgrade required state
   - invite blocked by seat limit state
   - upgrade CTA
7. Prepare UI plan for Canonical Module Registry:
   - permissions checklist
   - module labels
   - disabled modules by plan
   - upgrade prompt when module is not included in plan
8. Stop before main merge or production deploy.

## Global rule

Agents should advance in parallel:

- Claude owns backend, database, Team validation, Module Registry, and Billing architecture.
- Antenor owns frontend health, UX readiness, mock cleanup, and smoke tests.

Agents must stop only at hard gates and keep reports short.
