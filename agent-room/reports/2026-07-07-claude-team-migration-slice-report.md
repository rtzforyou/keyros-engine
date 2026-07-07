# Report — Team real model migration-only slice (Step 4A)

```text
Task: Team real model migration-only slice
Agent: Claude
Status: needs_review
Product repo: rtzforyou/easytattoo-crm
Product branch: claude/team-real-model-migration-slice
Product commit SHA: 1ebb752ef156cb393a60f23b0239f438f8deb917
Migration file: supabase/migrations/20260711000000_team_users_columns_and_signup.sql
Engine branch: claude/team-migration-slice-report
Engine commit SHA: (this commit)
Files changed: 1 (the new migration only; +84 / -0 vs main)
Supabase applied: no
Remote changes: none (no apply_migration, no SQL run against production)
Permission requested from Victor: yes
```

## What changed

One new migration file, isolated on branch `claude/team-real-model-migration-slice` (based on product `main` = 4ca84a0). Verified via GitHub compare: the branch adds exactly this one file and nothing else.

The migration does three non-destructive things:

1. Adds three columns to `public.users` (all `ADD COLUMN IF NOT EXISTS`):
   - `permissions jsonb NOT NULL DEFAULT '[]'::jsonb`
   - `email varchar(255)` (nullable)
   - `avatar_url text` (nullable)
2. `CREATE OR REPLACE FUNCTION public.handle_new_user()` — based on the current invite-aware version (migration `20260710000000`), now also writing `email` and `permissions` on the two INSERT paths:
   - invited signup → `email = new.email`, `permissions = COALESCE(matched_invite.permissions, '[]')`
   - normal admin signup → `email = new.email`, `permissions = '[]'` (admin is full-access by role, not by list)
3. Non-destructive backfill: `UPDATE public.users SET email = auth.users.email WHERE email IS NULL` (fills missing email only; never overwrites). `permissions` gets `'[]'` automatically from the column default; `avatar_url` left NULL.

## Why changed

Step 4A of the approved Team real model plan (`agent-room/reports/2026-07-07-claude-team-real-model-plan.md`, commit c96b7f2). `public.users` currently has no `permissions`, `email`, or `avatar_url` (verified live). Those columns are prerequisites for a real, RLS-scoped members list and for persisting per-user permissions. This slice only prepares the schema + signup trigger; no frontend, no hook, no remove semantics, no DELETE policy — all deferred to later slices per the plan.

## Verification performed

This is migration-only and was deliberately NOT applied to production (per task constraints). Validation was by inspection:

1. **Syntax review** — standard `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`, `CREATE OR REPLACE FUNCTION`, and a single `UPDATE ... FROM` backfill. No non-standard constructs.
2. **Compared with existing `handle_new_user()`** — the new body is the current invite-aware version (`20260710000000`) with only the two INSERT column-lists extended by `email` + `permissions`. Control flow (invite match → link to invited org; else → create org as admin) is unchanged.
3. **Expected behavior:**
   - *Normal signup without invite:* creates a new organization, inserts the user as `admin` with `email = new.email` and `permissions = '[]'`. Same as before, plus email/permissions populated.
   - *Signup with pending invite:* links the user to the invited org with the invite's role, `email = new.email`, and `permissions` copied from the invite; marks the invite accepted. (Previously permissions were dropped because the column didn't exist — this slice starts carrying them.)
   - *Existing users:* unaffected structurally; each gets `permissions = '[]'` via the default and `email` backfilled from `auth.users`. No role/permission change, no access change (enforcement is still by role in Login until a later slice).
4. **Not applied remotely** — no `apply_migration`, no `BEGIN/ROLLBACK` probe, no DDL run against production. I did earlier run one read-only `information_schema` SELECT (during planning) to confirm the current columns; no write.

## Risks / not verified

- **⚠ Privilege-escalation risk to fix in the NEXT slice (not this one):** the existing RLS policy "Users can update their own profile" (`20260620000000_harden_security.sql`) allows a user to UPDATE their own row with no column restriction. Once `permissions` exists on `public.users`, a non-admin could in principle self-grant permissions by updating their own row. **This is not yet exploitable** because permissions are not enforced anywhere (Login still grants ALL_MODULES), but the permission-enforcement slice MUST also add column-level protection (e.g. a BEFORE UPDATE trigger blocking non-admins from changing their own `role`/`permissions`, or splitting the self-update policy to exclude those columns). Flagging now so it is not forgotten.
- Migration not executed against production, so runtime success is asserted by inspection, not proven. Recommend applying in a Supabase branch/preview or a `BEGIN...ROLLBACK` probe under explicit authorization before merging to main.
- `email` is denormalized from `auth.users`; it can drift if a user later changes their auth email. Acceptable for display; note it.
- `avatar_url` column also fixes a pre-existing latent bug (`ProfileSettings.tsx` updates a non-existent `avatar_url`) — intentional side effect, called out so it is a conscious decision.
- `matched_invite.permissions` assumed to be the invite's JSONB permissions column (confirmed it exists on `public.invitations`).

## Rollback plan

Reversible via a follow-up migration (no data loss for core fields):
```sql
ALTER TABLE public.users DROP COLUMN IF EXISTS permissions;
ALTER TABLE public.users DROP COLUMN IF EXISTS email;
ALTER TABLE public.users DROP COLUMN IF EXISTS avatar_url;
-- and CREATE OR REPLACE handle_new_user() back to the 20260710000000 (invite-aware) body.
```
Since the migration was never applied to production, "rollback" today is simply not merging/applying the branch.

## Next recommended Team slice

Slice 2 — `useTeamMembers` hook + read-only wiring in `TeamContent.tsx` (real members list + edit role/permissions), **after** this migration is reviewed and applied under authorization. Note the privilege-escalation item above should be scheduled together with, or before, the permission-enforcement slice (Login.tsx reading real permissions).

## Compliance with Task 4A constraints

No frontend touched. Only one new migration file added. No existing migration modified. No hook/component changed. Migration NOT applied to Supabase (no `apply_migration`, no destructive SQL). No DELETE policy, no remove-member semantics, no `useTeamMembers.ts`, no `TeamContent.tsx`/`Login.tsx` change. No Antenor branch touched. The product branch + commit were created via the GitHub API off `main` to avoid disturbing the shared local checkout (currently sitting on an Antenor branch with unrelated uncommitted files) — no local product git operations were performed.

Signed,
Claude — Heavy Implementation Agent
