# Report — EPIC Zone 1 Team Real Model (Sprints 1–7)

```text
EPIC: Zone 1 — Team Real Model
Agent: Claude/Fable
Status: needs_review — all sprints delivered to their authorized boundary; production gates (migration apply, merges) intentionally NOT crossed
Supabase applied: no
Merges: none
Permission requested from Victor: yes
```

## Sprint 1 — PR #9 readiness ✅
PR #9 (`claude/team-real-model-migration-slice`, head `9a2859f4...`) re-checked against current `main` (`82cc656`): `state=open`, `mergeable=true`, `mergeable_state=clean`, 1 file `+181/-0`. Migration NOT applied (columns/trigger absent in prod). Merge-ready; no rebase needed.

## Sprint 2 — apply/test/rollback runbook ✅
Runbook `2026-07-07-claude-task1-team-migration-apply-test-runbook.md` reviewed: covers all 8 scenarios + `accept_invitation` regression, each `BEGIN…ROLLBACK`, plus apply + rollback steps. Complete and runnable. Not applied.

## Sprint 3 — useTeamMembers hook ✅ (draft)
`hooks/useTeamMembers.ts` — reads real members from `public.users` by authenticated org, explicit `.eq(organization_id)` + RLS, maps to `TeamMember`, read-only `{members,isLoading,error,refetch}`. Branch `claude/team-use-team-members-hook`, commit `e5f9b95`, **draft PR #10**. Build PASS. Blocked-for-merge by PR #9 apply (reads email/permissions/avatar_url columns).

## Sprint 4 — TeamContent read-only ✅ (draft)
`components/team/TeamContent.tsx` members list sourced from `useTeamMembers`; `useMockData` removed from the file; edit/remove kept as non-persisting placeholders. Branch `claude/team-content-readonly-wiring` (stacked), commit `85d2297`, **draft PR #11**. Build PASS (fresh clone, `vite ✓`). Blocked-for-merge by PR #9 apply.

## Sprint 5 — admin role/permissions editing DESIGN (plan only)
Goal: let an admin persist a member's `role` + `permissions` to `public.users`.
- Hook extension (later slice): add `updateMember(id,{role,permissions})` → `UPDATE public.users SET role, permissions WHERE id=<id> AND organization_id=<org>`. RLS policy "Admins can update organization members" already permits this; the BEFORE UPDATE guard (from PR #9) allows it because the acting user is admin.
- UI: the existing Edit dialog already collects role + permissions; replace the placeholder `alert()` with a real `await updateMember(...)` + `refetch()`.
- Constraints: admin cannot edit members of another org (RLS). An admin editing their OWN row's role/permissions is allowed by the guard (admin), but consider a UI guard to prevent an admin accidentally self-demoting to zero-access (leave a "cannot remove last admin" check for the removeMember slice).
- NOT in scope now: enforcement (Login still grants all). This slice only persists the values.
- Risk: medium (writes to sensitive columns) — must land only after migration applied + guard verified. Own PR, own task authorization.

## Sprint 6 — Login permission enforcement PLAN (plan only)
Current: `components/auth/Login.tsx` `handleUserData()` grants `ALL_MODULES` to everyone.
Target resolution:
```
role === 'admin'  -> all active modules
non-admin         -> public.users.permissions (jsonb array from the row)
```
- Read `role, permissions` from the `users` select already done in `handleUserData` (add the columns).
- Map to the `GoogleUserData.permissions` shape currently consumed by `App.tsx`'s `hasPermission`.
- Emergency fallback (avoid lockout): if `permissions` is null/empty AND role is non-admin, fall back to a minimal safe set (e.g. `['dashboard']`) rather than an empty menu; log a warning. Admins are never gated. Keep a documented manual override (set role='admin' in DB) as the break-glass path.
- Ordering dependency: do NOT ship enforcement until (a) migration applied, (b) hook + read-only UI verified, (c) admins can actually edit permissions (Sprint 5) — otherwise non-admins get locked to defaults with no UI to fix it.
- Risk: HIGH (touches Auth; can lock users out) — explicit Victor authorization required; own PR; test with an admin and a non-admin account before merge.

## Sprint 7 — removeMember soft-remove DESIGN (plan only)
Decision: **soft remove / revoke access** (default; hard delete NOT authorized).
- Schema (later migration, own task): add `public.users.status text NOT NULL DEFAULT 'active'` (values `active` | `disabled`), OR `disabled_at timestamptz NULL`. Prefer `status` for clarity + easy filtering.
- Guard interaction: `status` becomes another sensitive column — extend the BEFORE UPDATE guard so only admins can change it (same pattern as role/permissions).
- Hook: `removeMember(id)` → `UPDATE public.users SET status='disabled' WHERE id=<id> AND organization_id=<org>`; `useTeamMembers` list filters `status='active'` (or shows disabled with a badge — Victor's UI call).
- Access effect: Login enforcement (Sprint 6) must treat `status='disabled'` as no access (and ideally sign them out). This couples Sprint 7 to Sprint 6.
- Safety: block removing/disabling the last remaining admin of an org (count check).
- Hard delete (auth.users removal) stays OUT — would need a service_role Edge Function + explicit Victor choice.
- Risk: medium-high (access revocation) — own migration + task authorization.

## Definition of Done coverage
- PR #9 readiness: ✅ clean/merge-ready, not applied.
- hook branch/PR: ✅ #10 draft (blocked-for-merge by apply).
- TeamContent read-only branch/PR: ✅ #11 draft (blocked-for-merge by apply).
- permissions editing plan: ✅ Sprint 5 above.
- Login enforcement plan: ✅ Sprint 6 above.
- removeMember soft-remove plan: ✅ Sprint 7 above.
- build/typecheck: ✅ build PASS on the wiring branch (hook+wiring); earlier `tsc` only pre-existing 2 unrelated errors.
- reports in keyros-engine: ✅ this + prior slice/runbook/checklist/V0 reports.
- risks: per sprint above.
- rollback: PR #9 rollback documented in the migration-slice report; drafts #10/#11 roll back by simply not merging.
- next recommended EPIC: below.

## What was NOT verified
- Live authenticated UI (needs real login + applied migration).
- Runtime behavior of #10/#11 (depends on unapplied columns) — build-level only.

## Blocks (hard stop gates hit)
1. Apply PR #9 migration to Supabase — needs "Approve Task 2 — apply Team migration to Supabase".
2. Merge PR #9 → #10 → #11 — needs merge authorization (correct order).
3. Sprint 5 (real editing), 6 (Login enforcement), 7 (soft-remove) IMPLEMENTATION — each needs the migration applied first + explicit authorization (touch RLS/Auth/sensitive columns). Plans delivered; code intentionally not written.

## Next recommended EPIC
Unblock Team: authorize migration apply → I run the runbook → ordered merges #9/#10/#11 → then authorize Sprint 5 implementation as its own task. In parallel, a planning-only EPIC (Client Payments model plan or Business Hours schema plan) is fully within my autonomy if you want me progressing while the Team gates are decided.

Signed, Claude/Fable — Heavy Implementation Agent
