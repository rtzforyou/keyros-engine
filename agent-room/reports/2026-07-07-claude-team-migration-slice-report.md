# Report — Team real model migration-only slice (Step 4A) — REVISED with security guard

```text
Task: Team real model migration-only slice — revision
Agent: Claude
Status: needs_review
Product branch: claude/team-real-model-migration-slice
Product commit SHA before revision: 1ebb752ef156cb393a60f23b0239f438f8deb917
Product commit SHA after revision: 9a2859f4bb6291664a74ad9810b969537f85271a
Migration file: supabase/migrations/20260711000000_team_users_columns_and_signup.sql
Engine branch: claude/team-migration-slice-report
Engine commit SHA: (this commit)
Files changed: 1 (the same migration file; now +181 / -0 vs main; no second migration added)
Supabase applied: no
Remote changes: none (no apply_migration, no SQL run against production)
Permission requested from Victor: yes
```

## What changed (revision on top of the original slice)

Same single migration file, same branch. The revision adds two things to the file (sections 4 and 5); sections 1–3 are unchanged from the reviewed version.

**Security guard added (section 4):**
- New function `public.protect_sensitive_user_columns()` + trigger `protect_sensitive_user_columns_trg` — `BEFORE UPDATE ON public.users FOR EACH ROW`.
- Logic: if `role`, `permissions`, or `organization_id` change (`IS DISTINCT FROM`), the update is allowed only when the acting `auth.uid()` user's role is `'admin'`. Non-admin self-edits of those columns raise an exception. `full_name`/`avatar_url` edits do not trip the guard.
- Trusted-context handling: `auth.uid() IS NULL` (service_role / definer without JWT) is allowed; a transaction-local GUC `app.bypass_user_guard='on'` lets validated internal flows perform a controlled transition.
- `SECURITY DEFINER SET search_path = public` so it reads `public.users` without RLS recursion.

**accept_invitation() made guard-aware (section 5):**
- `accept_invitation()` (originally defined in migration `20260710000000`) legitimately updates the caller's OWN row's `organization_id`/`role` — which the new guard would otherwise block for non-admin invitees. It is redefined here (via `CREATE OR REPLACE`, inside this same allowed migration file) to (a) set the transaction-local bypass GUC before its update, and (b) also copy the invite's `permissions` into the new column.
- `set_config(..., true)` is transaction-local, so the bypass cannot leak to other statements.

## Why the guard could not be a pure RLS `WITH CHECK`

The task asked to prefer a `BEFORE UPDATE` trigger, which is also the correct choice here: RLS `WITH CHECK` on the existing "update own profile" policy cannot compare OLD vs NEW values, so it cannot say "you may update your row but not change these specific columns." A row-level trigger comparing `OLD`/`NEW` is the right tool. The trigger complements (does not replace) the existing RLS policies.

## Verification performed (by inspection — NOT applied to production)

Deliberately not applied (no `apply_migration`, no BEGIN/ROLLBACK probe, no DDL against prod). Expected outcomes, traced against the SQL:

- member updates own `full_name`/`avatar_url` → **allowed** (sensitive columns unchanged → guard returns NEW).
- member updates own `permissions` → **blocked** (permissions changed, acting role ≠ admin → RAISE).
- member updates own `role` → **blocked** (same reason).
- member updates own `organization_id` → **blocked** (same reason).
- admin updates member `permissions` → **allowed** (acting role = admin; existing RLS restricts to same org).
- admin updates member `role` → **allowed** (same).
- admin updates member `full_name`/`avatar_url` → **allowed** where the existing admin RLS policy permits.
- signup without invite → **still creates organization + admin** (uses INSERT; BEFORE UPDATE trigger does not fire; email/permissions populated).
- signup with invite → **still joins invited organization and copies invite permissions** (INSERT path).
- **accept_invitation() → still works** for a non-admin invitee (bypass GUC set inside the function permits the controlled self-transition). This is the one case the guard would have broken without section 5 — verified by design and explicitly handled.

## Risks / not verified

- Not executed against production, so runtime success is asserted by inspection. Recommend applying in a Supabase branch/preview or a `BEGIN...ROLLBACK` probe under explicit authorization before merge.
- GUC-bypass safety depends on the caller not being able to run `set_config('app.bypass_user_guard','on',true)` and an UPDATE in the same transaction from the anon client. Under Supabase/PostgREST this is not exposed: there is no generic "run SQL" endpoint, and each PostgREST request is its own transaction; only defined RPCs run multi-statement transactions. So the bypass is reachable only by the definer functions that set it. Acceptable for this architecture; noted for awareness.
- The guard treats `auth.uid() IS NULL` as trusted (service_role/definer). Any future public/anon path that updates `public.users` directly would bypass the guard — but no such path exists today, and anon has no update grant on `public.users`.
- Admins can still change their own sensitive columns (e.g. self-demote or move org). That is an admin action, outside this guard's threat model (non-admin self-escalation).
- Permission enforcement is still not active in the app (Login grants ALL_MODULES); this guard is the DB-side prerequisite so that when enforcement lands, self-escalation is already impossible.

## Rollback plan

Reversible via a follow-up migration (no data loss):
```sql
DROP TRIGGER IF EXISTS protect_sensitive_user_columns_trg ON public.users;
DROP FUNCTION IF EXISTS public.protect_sensitive_user_columns();
-- CREATE OR REPLACE accept_invitation() back to the 20260710000000 body (without the bypass);
-- ALTER TABLE public.users DROP COLUMN IF EXISTS permissions, email, avatar_url;
-- CREATE OR REPLACE handle_new_user() back to the 20260710000000 body.
```
Since the migration was never applied to production, "rollback" today is simply not merging/applying the branch.

## Security guard added — summary
`BEFORE UPDATE ON public.users` trigger blocking non-admin changes to `role`, `permissions`, `organization_id`, with a transaction-local bypass for the validated `accept_invitation()` flow and a pass-through for trusted no-JWT backend contexts.

## Compliance with task constraints
Only the existing migration file `20260711000000_...sql` was modified (no second migration file added). No frontend, no hook, no component changed. Migration NOT applied to Supabase. No PR opened, no merge. No Antenor branch touched. The revision was made via the GitHub API on the same branch to avoid disturbing the shared local checkout.

Next recommended Team slice: apply/review this migration under authorization, then Slice 2 (`useTeamMembers` hook + read-only wiring). The permission-enforcement slice (Login.tsx reading real permissions) can now safely follow, since DB-side self-escalation is closed.

Signed,
Claude — Heavy Implementation Agent
