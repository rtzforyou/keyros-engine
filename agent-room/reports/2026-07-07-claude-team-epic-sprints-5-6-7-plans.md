# EPIC Zone 1 — Team Real Model — Sprints 5–7 plans + EPIC status

```text
EPIC: ZONE 1 — Team Real Model
Agent: Claude/Fable
Status: needs_review — Sprints 1–4 delivered (code/draft), Sprints 5–7 delivered as plans; implementation past here is gated (migration apply / Auth / schema)
Supabase applied: no · main merged: no · deployed: no
```

## Sprint status recap (1–4 already delivered earlier)
- Sprint 1 — PR #9 readiness: DONE. PR #9 open, `mergeable: clean` vs main 82cc656, 1 migration file (+181), guard present, not applied. Report: `2026-07-07-claude-zone1-team-review-and-apply-checklist.md`.
- Sprint 2 — apply/test runbook review: DONE. Runbook complete (8 scenarios + accept_invitation regression + apply + rollback). Report: `2026-07-07-claude-task1-team-migration-apply-test-runbook.md`.
- Sprint 3 — useTeamMembers hook draft: DONE. Draft PR **#10** (`claude/team-use-team-members-hook`, commit e5f9b95). Build PASS. Blocked-on: PR #9 migration apply (reads new columns).
- Sprint 4 — TeamContent read-only draft: DONE. Draft PR **#11** (`claude/team-content-readonly-wiring`, commit 85d2297, stacked on #10). Build PASS. Edit/remove are non-persisting placeholders.

---

## Sprint 5 — Admin role/permissions editing (DESIGN)

Goal: let an admin persist a member's `role` + `permissions` from the existing TeamContent edit dialog, replacing the current read-only placeholder.

Design:
- `useTeamMembers` gains `updateMember(id, { role, permissions })` → `UPDATE public.users SET role, permissions WHERE id = <id>`; then `refetch()`.
- `TeamContent.handleUpdateMember` calls `await updateMember(memberToEdit.id, { role: editRole, permissions: editPermissions })` instead of the placeholder alert.
- Authorization is already enforced at the DB by the migration 20260711000000 guard: role/permissions changes require the acting user to be `admin`; RLS "Admins can update organization members" restricts to same org. So no new RLS is introduced by this sprint — it relies on the already-reviewed guard in PR #9.
- Delivery shape: a draft PR stacked on #11 (frontend + hook only), openable on request.

Depends on: PR #9 migration applied + PRs #10/#11 merged (columns + guard live). Until then it stays a plan/draft.
Risk: low; the DB guard is the real gate. UI only shows the edit action for members; non-admins are blocked at the DB even if the UI were reachable. Rollback: revert the frontend PR; no schema change in this sprint.

---

## Sprint 6 — Login permissions enforcement (PLAN)

Current: `components/auth/Login.tsx` `handleUserData()` grants `ALL_MODULES` to every authenticated user.

Target resolution:
- `role === 'admin'` → all active modules.
- non-admin → `public.users.permissions` (the persisted array).

Plan:
- `handleUserData` already `SELECT`s the user row; extend the select to include `role, permissions`.
- Compute `permissions = dbUser.role === 'admin' ? ALL_MODULES : (Array.isArray(dbUser.permissions) ? dbUser.permissions : [])`.
- Pass that to `onLogin({... permissions})` instead of the hardcoded `ALL_MODULES`.

Safety (anti-lockout — mandatory):
- Admin is ALWAYS full access (never gated by the array).
- If a non-admin has empty/missing permissions, default to a minimal safe set (e.g. `['dashboard']`) so they never hit a blank app.
- Never white-screen: if the users row/permissions can't be read, fall back to the previous permissive behavior for that session and log it, rather than locking the user out.
- Emergency override: keep a documented way (env flag or admin-role short-circuit) to restore full access if a rollout misconfigures permissions.

Depends on: migration applied + Team read-only verified. Gated (touches Auth/Login) — PLAN ONLY here, do not implement without an explicit task.
Risk: medium (lockout). Fully mitigated by the admin-always-full + minimal-default + never-crash rules above. Rollback: revert Login change → back to ALL_MODULES for everyone (safe, permissive).

---

## Sprint 7 — removeMember soft-remove (DESIGN)

Default (recommended): soft remove / revoke access. Hard delete of `auth.users` (needs service_role Edge Function) is NOT authorized.

Design:
- Schema (future small migration): add `public.users.status varchar not null default 'active'` (values `active` | `disabled`) — or `disabled_at timestamptz null`. Non-destructive, additive.
- `useTeamMembers.removeMember(id)` → `UPDATE public.users SET status='disabled' WHERE id=<id>` (admin-only, enforced by the same admin RLS/guard family). Row is kept for history/audit.
- Access resolution: Login/permission resolution treats `status='disabled'` as no-access (and the members list can filter or badge disabled members). Their auth account still exists but the app grants nothing.
- Reversible: `status='active'` restores access.
- Hard delete, if ever wanted, is a separate explicitly-authorized Edge Function task (service_role + admin API) — out of scope.

Depends on: a future additive migration (status column) + hook + Login-awareness. Gated (schema + auth). DESIGN ONLY here.
Risk: low (additive, reversible). Rollback: drop the status column / ignore it; no data loss.

---

## EPIC Definition-of-Done checklist
- PR #9 readiness status: ✓ (clean, not applied)
- hook branch/PR: ✓ PR #10 (blocked-on PR #9 apply)
- TeamContent read-only branch/PR: ✓ PR #11
- permissions editing plan: ✓ (Sprint 5 above)
- Login enforcement plan: ✓ (Sprint 6 above)
- removeMember soft-remove plan: ✓ (Sprint 7 above)
- build/typecheck where code changed: ✓ (PR #10/#11 build PASS)
- reports in keyros-engine: ✓ (this + prior Team reports)
- risks: ✓ (per sprint)
- rollback plan: ✓ (per sprint)
- next recommended EPIC: below

## Why the EPIC is needs_review, not completed
Everything I can do autonomously in Zone 1 is done (drafts + plans). The remaining value — applying the migration, merging #9/#10/#11, enforcing Login, implementing edit/remove — all sit behind hard gates (Supabase apply, main merge, Auth change, new schema) that require Victor authorization. No production enablement has happened.

## Next recommended EPIC / unblock path
1. Victor: "Approve Task 2 — apply Team migration to Supabase" → I run the runbook, then request merges #9 → #10 → #11.
2. Then authorize Sprint 5 (draft PR for real editing), Sprint 6 (Login enforcement implementation — Auth gate), Sprint 7 (status-column migration + soft-remove).
3. Parallel option (no gates): start EPIC Zone-1-adjacent planning I own — Client Payments real-model plan or Calendar businessHours schema plan — both pure planning within my domain.

Signed, Claude/Fable — Heavy Implementation Agent
