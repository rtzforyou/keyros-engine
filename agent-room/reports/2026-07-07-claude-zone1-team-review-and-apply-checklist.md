# Report — Zone 1 Team: Task 1 (review) + Task 2 (final apply checklist)

```text
Task: Zone 1 — Task 1 (review PR #9 + runbook) + Task 2 (final apply checklist)
Agent: Claude/Fable
Status: completed (review + checklist); STOPPED at Task 3 gate (apply requires explicit approval)
Product PR: #9
Product branch: claude/team-real-model-migration-slice
Head SHA: 9a2859f4bb6291664a74ad9810b969537f85271a
Base: main @ 82cc656e35f20e662e886fa911faa13f5c474ede
Engine branch: claude/zone1-team-review-and-apply-checklist
Report path: agent-room/reports/2026-07-07-claude-zone1-team-review-and-apply-checklist.md
Supabase applied: no
Permission requested from Victor: yes — explicit "Approve Task 2 — apply Team migration to Supabase"
```

## Task 1 — Review PR #9 + runbook (re-review against advanced main)

- PR #9 state: **open**, `mergeable: true`, `mergeable_state: clean` against the current `main` (82cc656, which now includes merged PRs #2–#8). No rebase needed.
- Scope: exactly **1 file** — `supabase/migrations/20260711000000_team_users_columns_and_signup.sql` (+181/-0). No frontend/hooks/components in the PR.
- Guard present in PR head: `protect_sensitive_user_columns()` + `BEFORE UPDATE ON public.users` trigger, plus guard-aware `accept_invitation()`.
- Not applied to production (verified live): `schema_migrations` has no `20260711000000`; columns `permissions/email/avatar_url` absent; guard trigger absent.
- Runbook (`2026-07-07-claude-task1-team-migration-apply-test-runbook.md`) exists and covers all 8 required scenarios + the `accept_invitation` regression, each as `BEGIN…ROLLBACK`, plus apply and rollback steps. Reviewed as sound.
- Review verdict: **PR #9 is merge-ready and the runbook is complete.** No revision required.

## Task 2 — Final apply checklist (pre-apply go/no-go)

To be run ONLY under an explicit "Approve Task 2" authorization. Order:

```text
[ ] 0. Confirm explicit authorization text received: "Approve Task 2 — apply Team migration to Supabase".
[ ] 1. Confirm PR #9 head is still 9a2859f4... and mergeable_state=clean.
[ ] 2. Take a pre-apply snapshot of public.users definition:
       select column_name,data_type from information_schema.columns
       where table_schema='public' and table_name='users' order by ordinal_position;
[ ] 3. Dry-run probe (non-persistent): run the full migration body inside BEGIN … ROLLBACK; confirm 0 errors.
[ ] 4. Apply via apply_migration (name=team_users_columns_and_signup, body = the PR #9 file) — recorded in schema_migrations.
[ ] 5. Post-apply structural checks:
       - 3 new columns exist (permissions/email/avatar_url);
       - trigger protect_sensitive_user_columns_trg exists on public.users;
       - handle_new_user() + accept_invitation() are the new bodies.
[ ] 6. Run the 8 functional tests + B9 regression from the runbook, each in BEGIN…ROLLBACK:
       B1 signup no-invite -> org+admin; B2 signup+invite -> joins org, perms copied;
       B3 self full_name/avatar -> allowed; B4/B5/B6 self permissions/role/org -> BLOCKED;
       B7/B8 admin edits member perms/role -> allowed; B9 accept_invitation non-admin -> allowed.
[ ] 7. Confirm NO existing production rows were mutated by the tests (all rolled back);
       confirm existing users got permissions='[]' default + email backfilled (non-destructive).
[ ] 8. If any check fails -> execute rollback (drop trigger/function, drop columns, restore prior
       handle_new_user/accept_invitation from 20260710000000) and report needs_fix.
[ ] 9. Decide PR #9 merge to main (separate explicit authorization — do NOT auto-merge).
```

Go/no-go: proceed to apply only if steps 0–3 pass; treat any failure in 5–7 as no-go and roll back.

## Risks
- Low, and unchanged from the slice review. The one behavioral dependency to keep in mind: applying this migration changes `handle_new_user()` and `accept_invitation()` on production — both run on real auth flows — so the signup + invite regression tests (B1, B2, B9) must pass before considering it done. The guard closes the self-escalation hole flagged earlier.
- Merge of PR #9 to `main` and the Supabase apply are two separate authorizations; neither done here.

## Compliance
Review + checklist only. Nothing applied to Supabase. PR #9 not merged. No product code changed. Payments/Calendar/Dashboard untouched. No Antenor branch touched. Stopped at the Task 3 approval gate.

## Next
Awaiting explicit: "Approve Task 2 — apply Team migration to Supabase" (and, separately, approval to merge PR #9). Tasks 4–7 (useTeamMembers, TeamContent wiring, role/permissions editing, Login enforcement) remain blocked until the migration is applied and verified.

Signed, Claude/Fable — Heavy Implementation Agent
