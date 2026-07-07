# Report — EPIC Team: read-only draft slices (useTeamMembers + TeamContent)

```text
EPIC: Zone 1 — Team Real Model
Agent: Claude/Fable
Status: delivered up to authorized boundary (read-only draft PRs); STOPPED — next sprints need migration applied + new authorization
Supabase applied: no
PR #9 merged: no
Permission requested from Victor: yes
```

## Scope delivered (as authorized: "até useTeamMembers + TeamContent read-only, sem aplicar Supabase")

### Sprint 4 — useTeamMembers hook
- New file `hooks/useTeamMembers.ts`. Reads real members from `public.users`, resolved by the authenticated user's `organization_id`, with explicit `.eq('organization_id', ...)` (defense-in-depth per CLAUDE.md) on top of RLS. Maps `full_name/email/role/permissions/avatar_url/created_at` → `TeamMember`. Read-only: returns `{ members, isLoading, error, refetch }`.
- Branch: `claude/team-use-team-members-hook` (base main). Commit `e5f9b9511c2dabf75bd3a81c48f0cfa905901b5e`.
- Draft PR: **#10**.

### Sprint 5 — TeamContent read-only wiring
- `components/team/TeamContent.tsx`: members list now sourced from `useTeamMembers` (real), `useMockData` removed from this file. Edit/remove row actions kept in the UI but made non-persisting placeholders (clear "chega na próxima fase" alerts + comments), since real editing/removal are later sprints. Invitations + internal-artists sections untouched (already real).
- Branch: `claude/team-content-readonly-wiring` (stacked on the hook branch). Commit `85d2297abfeeda6972e1448b419998845f4d7c65`.
- Draft PR: **#11** (base = hook branch).

## Verification
- Build: fresh clone of `claude/team-content-readonly-wiring` (hook + wiring) → `npm ci` + `npm run build` → **PASS** (`vite ✓ built in ~2s`, dist produced, 0 errors; only pre-existing chunk-size/dynamic-import warnings).
- Not run: authenticated runtime UI (needs real login + the migration applied) — stated, not faked.

## Critical dependency / why this stays DRAFT
Both PRs read `public.users.email`, `.permissions`, `.avatar_url` — columns added ONLY by PR #9's migration `20260711000000`, which is **not applied to production**. Therefore:
- These PRs must NOT be merged or deployed until PR #9's migration is applied and verified (runbook already published).
- Correct merge order once authorized: apply migration (PR #9) → verify via runbook → merge #10 → merge #11.

## EPIC status after this delivery
- Sprint 1 (review PR #9) — done (earlier).
- Sprint 2 (apply checklist) — done (earlier).
- Sprint 3 (apply migration) — BLOCKED: needs explicit "Approve Task 2 — apply Team migration to Supabase" (forbidden to me autonomously).
- Sprint 4 (useTeamMembers) — DONE as draft PR #10.
- Sprint 5 (TeamContent read-only) — DONE as draft PR #11.
- Sprint 6 (real role/permissions editing) — NOT started (beyond the authorized "read-only" boundary; also depends on applied migration + touches the RLS guard path).
- Sprint 7 (Login permission enforcement) — NOT started (touches Auth/Login — outside my autonomy without an explicit task).

## Risks / not verified
- Draft code depends on the unapplied migration; runtime behavior can only be confirmed after apply. Build-level correctness confirmed.
- Read-only slice intentionally leaves edit/remove as placeholders — reviewers should expect non-functional action buttons until Sprint 6.
- `useTeamMembers` currently has no realtime subscription (unlike some other hooks); acceptable for read-only; can add later if needed.

## Compliance
No Supabase apply. No RLS/Auth change. No merge. No deploy. No Payments/Calendar/Dashboard touched. No Antenor branch touched. All product changes are isolated draft branches created via the GitHub API (shared local checkout untouched).

## Next recommended
1. Victor: "Approve Task 2 — apply Team migration to Supabase" → I run the runbook, then request merge of PR #9, then #10, then #11.
2. After that, authorize Sprint 6 (real editing) and Sprint 7 (Login enforcement) as explicit tasks (both touch sensitive paths).

Signed, Claude/Fable — Heavy Implementation Agent
