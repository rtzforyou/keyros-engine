# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Revision required — Team migration must protect sensitive user columns before approval
Date: 2026-07-07

## Status

Your migration-only slice was received and reviewed.

Product branch:

```text
claude/team-real-model-migration-slice
```

Product commit reviewed:

```text
1ebb752ef156cb393a60f23b0239f438f8deb917
```

Migration file:

```text
supabase/migrations/20260711000000_team_users_columns_and_signup.sql
```

ChatGPT confirmed the branch contains exactly one new migration file.

## Review result

Status: needs_revision before PR/merge/apply.

The migration is mostly correct and non-destructive:

- adds `permissions`, `email`, `avatar_url` with `ADD COLUMN IF NOT EXISTS`;
- updates `handle_new_user()` to populate `email` and invited-user `permissions`;
- backfills missing email only;
- does not apply production SQL.

However, the migration introduces a sensitive `permissions` column without protecting it from the existing self-update profile policy.

## Blocking issue

Existing RLS has an own-profile update path.

After this migration, if `permissions` exists and no column-level guard exists, a non-admin user may be able to update their own `permissions` row and self-grant modules once the app starts enforcing permissions.

This is not acceptable to merge/apply as-is.

## Current authorized task

Revise the same migration-only branch to add a database-side guard that prevents non-admin/self-profile updates from changing sensitive columns.

## Product branch

Continue on the same branch:

```text
claude/team-real-model-migration-slice
```

## Allowed files

Only modify the existing migration file:

```text
supabase/migrations/20260711000000_team_users_columns_and_signup.sql
```

Do not add a second migration file for this revision.

Do not modify frontend.

Do not modify hooks/components.

Do not apply the migration to Supabase production.

## Required guard

Add a trigger/function or equivalent database-side protection so that:

1. non-admin users cannot update their own `permissions`;
2. non-admin users cannot update their own `role`;
3. non-admin users cannot update their own `organization_id`;
4. normal profile edits such as `full_name` and `avatar_url` remain possible for the row owner;
5. admin updates to other users in the same organization remain possible under existing RLS policy.

Prefer a clear `BEFORE UPDATE ON public.users` trigger that compares `OLD` and `NEW` values and checks the acting `auth.uid()` user's role/org.

If a trigger is not the correct approach, explain why and implement the safer alternative in SQL.

## Required verification

Do not apply to production.

Verify by inspection and document expected outcomes for:

```text
member updates own full_name/avatar_url -> allowed
member updates own permissions -> blocked
member updates own role -> blocked
member updates own organization_id -> blocked
admin updates member permissions -> allowed
admin updates member role -> allowed
admin updates member full_name/avatar_url -> allowed if existing admin policy permits
signup without invite -> still creates organization/admin
signup with invite -> still joins invited organization and copies invite permissions
```

## Required report update

Create or update the report in `keyros-engine`:

```text
agent-room/reports/2026-07-07-claude-team-migration-slice-report.md
```

Use engine branch:

```text
claude/team-migration-slice-report
```

Report must include:

```text
Task: Team real model migration-only slice — revision
Agent: Claude
Status: needs_review / completed / blocked
Product branch:
Product commit SHA before revision:
Product commit SHA after revision:
Migration file:
Engine branch:
Engine commit SHA:
Files changed:
What changed:
Security guard added:
Verification performed:
Supabase applied: no
Risks / not verified:
Rollback plan:
Permission requested from Victor: yes
```

## Stop rule

After revising the migration and report, stop.

Do not open PR.

Do not merge.

Do not apply to Supabase production.

Do not start frontend/hook work.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
