# Team Members Real Model — Plan (Step 4, planning only)

Task: Step 4 — Team members real model plan
Agent: Claude — Heavy Implementation Agent
Status: needs_review (plan only — no implementation, no code, no migration)
Repo inspected: rtzforyou/easytattoo-crm (product main `4ca84a0`)

## Honesty note

This plan did not previously exist on GitHub because Step 4 had not actually been executed before — in earlier turns I only *recommended* planning the Team model as a next step; it was never assigned or produced. This document is the plan, authored now from fresh read-only analysis, per the "if the report does not exist, recreate it from your analysis" instruction in `claude-current.md`.

## 1. Current state (verified read-only)

### What is already real
- **Invitations** — `hooks/useInvitations.ts` + `public.invitations` table. Real. Send/revoke/resend/copy-link all work. Accept flow now real too (my earlier `20260710000000_real_invite_acceptance_flow.sql`: invite-aware signup trigger + `accept_invitation()` RPC).
- **Internal artists (tattooers)** — `hooks/useTattooers.ts` + `public.tattooers`. Real. Add/delete. These are login-less profiles for calendar/pipeline assignment, NOT system users.

### What is still fake (the actual Step-4 target)
- `components/team/TeamContent.tsx` line 38: `const { teamMembers, updateTeamMember, removeTeamMember } = useMockData(locale);`
- The "Acessos & Sistema" → active members table renders `teamMembers` from `useMockData` (in-memory, resets on reload).
- Reads per member: `id, name, email, avatarUrl, role, permissions`.
- Writes: `updateTeamMember(id, { role, permissions })`, `removeTeamMember(id)`.

### Schema reality — `public.users` (verified via information_schema)
Columns: `id (uuid, PK→auth.users), organization_id (uuid, NOT NULL), role (varchar default 'admin'), full_name (varchar), created_at`.
- **No `permissions` column.** Per-user permissions are not persisted anywhere. They exist only on `invitations.permissions` (JSONB) and in the mock TeamMember.
- **No `email` column.** Email lives in `auth.users` (not directly selectable by the anon client under RLS).
- **No `avatar_url` column.** ⚠ Separate finding: `components/settings/ProfileSettings.tsx` does `supabase.from('users').update({ avatar_url: publicUrl })` against a column that does not exist — that update almost certainly errors/no-ops today. Out of scope for Team, but should be logged as its own bug.

### RLS reality — `public.users` (from `20260620000000_harden_security.sql`)
- SELECT: `organization_id = get_user_org_id()` (see own org members). ✅ good for a real members list.
- UPDATE own profile: `id = auth.uid()`. ✅
- UPDATE org members if admin: `organization_id = get_user_org_id() AND role of caller = 'admin'`. ✅ enables admin role/permission edits.
- **No DELETE policy on `public.users`** → `removeTeamMember` cannot hard-delete a member row under RLS today. Needs a decision + policy.

### Permission enforcement reality
- `components/auth/Login.tsx` grants `ALL_MODULES` to every authenticated user regardless of role/permissions. So permissions are cosmetic today; the real gate does not exist.
- The invite dialog collects `permissions` and stores them on the invitation, but they are never transferred to the user on accept (my invite migration copies `role`, not `permissions` — because `public.users` has no permissions column yet).

## 2. Objective (one sentence)

Replace the fake `teamMembers` state in TeamContent with a real, organization-scoped, RLS-protected members model backed by `public.users`, persisting role and per-user permissions, with a safe "remove member" semantic — without breaking signup, invites, or the existing profile/avatar flow.

## 3. Proposed target (planning only — not implemented)

### 3.1 Schema changes (one migration)
- `ALTER TABLE public.users ADD COLUMN permissions jsonb NOT NULL DEFAULT '[]'::jsonb;`
- `ALTER TABLE public.users ADD COLUMN email varchar(255);` (denormalized copy for display; populated by the signup trigger from `new.email`, and backfilled once). Alternative: expose email via a SECURITY DEFINER RPC/view joining auth.users — heavier, but avoids storing email twice. Recommend the column for simplicity + list performance.
- `ALTER TABLE public.users ADD COLUMN avatar_url text;` (also fixes the pre-existing ProfileSettings bug as a side effect — but flag it so it is a conscious decision, not an accident).
- Update `handle_new_user()` to also set `email` (and, for invited users, copy `matched_invite.permissions` into `users.permissions`). This threads into the invite flow already fixed.
- Add a DELETE (or a "revoke") policy — see 3.4.

### 3.2 New hook `hooks/useTeamMembers.ts`
- Resolve `organizationId` from the authenticated user (same pattern as every other real hook).
- `SELECT id, full_name, email, avatar_url, role, permissions FROM users WHERE organization_id = <org>` (RLS also enforces it — defense in depth).
- `updateMember(id, { role, permissions })` → `UPDATE users ... WHERE id = <id>` (RLS admin policy already allows this).
- `removeMember(id)` → semantics decided in 3.4.
- Returns `{ members, isLoading, error, updateMember, removeMember }` typed.

### 3.3 TeamContent wiring
- Swap line 38 from `useMockData` to `useTeamMembers`. Map DB fields → the TeamMember shape the table already expects (`name`←full_name, `avatarUrl`←avatar_url, etc.).
- No UI redesign — same table, same edit dialog.

### 3.4 "Remove member" — security decision needed from Victor
Two options; the plan must not guess:
- **(A) Soft remove / revoke access**: don't delete the auth user; instead null the org link or set a `status='removed'`/`disabled` flag so they lose access but history is preserved. Reversible. No admin API needed. Recommended for a CRM.
- **(B) Hard delete**: delete the `auth.users` row (cascades to `public.users`). Requires the Supabase Admin API from an Edge Function (service_role) — cannot be done from the anon client. Irreversible.
Recommendation: **(A)** for the first slice; (B) can come later as an explicit "delete permanently" action behind an Edge Function.

### 3.5 Real permission enforcement (ties into Login.tsx)
- `Login.tsx` must stop granting `ALL_MODULES` and instead read `role`/`permissions` from `public.users` (admin ⇒ all; otherwise the persisted `permissions`). This is a behavior change that touches auth — **Claude scope**, and must be coordinated because Antenor recently edited the `ALL_MODULES` list (`72deedd`). Do it as its own slice, not mixed with the members list.

## 4. Suggested implementation slices (each its own approved task, in order)

1. **Migration-only** slice: add `permissions`/`email`/`avatar_url` columns + update `handle_new_user()` (email + invited-user permissions) + backfill email/permissions for existing rows. Test signup + invite still work. No frontend change yet.
2. **Hook + read-only wiring**: `useTeamMembers` + TeamContent shows real members (read + edit role/permissions). Keep remove disabled or wired to soft-remove per 3.4.
3. **Remove semantics**: implement the chosen (A)/(B) remove path + DELETE/disable RLS policy.
4. **Real permission gate**: Login.tsx reads persisted permissions instead of ALL_MODULES (coordinate with whoever owns App.tsx/Login.tsx that cycle).

## 5. Risks / not verified
- `organization_id` is a single FK — no multi-org membership. Out of scope; note it.
- Changing `handle_new_user()` again touches a SECURITY DEFINER trigger that runs on every signup — must re-test both signup paths (with/without invite), as done for the invite fix.
- Storing `email` denormalized means it can drift from `auth.users.email` if a user changes email — acceptable for display, but note it.
- The `ProfileSettings` avatar_url bug is pre-existing and adjacent; adding the column fixes it incidentally — call it out so it is intentional.
- Permission-gate slice changes auth behavior; risk of locking a legitimate user out of modules — needs careful defaults (e.g. admin always full).

## 6. Files that WOULD be touched (future slices, not now)
- `supabase/migrations/<new>.sql` (new)
- `hooks/useTeamMembers.ts` (new)
- `components/team/TeamContent.tsx` (swap source)
- `components/auth/Login.tsx` (permission gate slice — coordinate ownership)
- possibly a new Edge Function if hard-delete (B) is chosen.

## 7. Final status (Task 0 fields)

```text
Task: Team real model plan
Agent: Claude
Status: needs_review
Engine branch: claude/team-real-model-plan
Report path: agent-room/reports/2026-07-07-claude-team-real-model-plan.md
Engine commit SHA: (this commit)
Files inspected: components/team/TeamContent.tsx; hooks/useInvitations.ts; hooks/useTattooers.ts (usage); components/auth/Login.tsx (ALL_MODULES / permission grant); supabase/migrations/20260618000000_core_saas_schema.sql; supabase/migrations/20260620000000_harden_security.sql; supabase/migrations/20260618000003_storage_invitations.sql; supabase/migrations/20260710000000_real_invite_acceptance_flow.sql; live public.users schema via information_schema (read-only SELECT)
Permission requested from Victor: yes
```

## Forbidden-actions compliance (Task 0)
No product code modified. No migration created or applied. No Supabase change (only a read-only information_schema SELECT to verify columns). No push to product main. No Antenor branch touched. No Team implementation started.

Signed,
Claude — Heavy Implementation Agent
