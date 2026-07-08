# CURRENT ORCHESTRATION

Status: ZONE 1 UNBLOCKED

## Decision Log

PR #9
Status: MERGED

Product repo:
- Repository: `rtzforyou/easytattoo-crm`
- PR: `#9`
- Merge commit: `541ea2c7f79da85e4c3d35f83f613c173bdeb569`
- Scope: migration-only Team real model foundation

Important boundary:
- PR #9 merge is complete.
- Supabase migration apply is NOT authorized by this file.
- Migration apply remains a separate Victor/ChatGPT gate.

## After PR #9 merge

1. Claude
- Apply migration `20260711000000` using the published runbook only after explicit authorization.
- Execute full Team validation checklist.
- Report:
  - commit SHA
  - migration result
  - rollback status
  - test results
- Stop.

2. ChatGPT
- Review migration report.
- Approve PR #12 only after successful validation.
- Define canonical Team Permissions modules.
- Design Billing Entitlements architecture.

3. Antenor
- Wait for migration completion.
- Merge Team read-only wiring after approval.
- Continue MockData elimination.
- Prepare authenticated smoke tests.

## Next EPIC

Billing Entitlements + Team Seat Limits

## Rules

- Billing is independent from Team.
- Gateway never grants permissions directly.
- Webhook updates subscription state.
- Keyros decides access.
- Team invitations must validate plan limits.
- If seat limit reached:
  - block invite
  - show Upgrade
  - redirect to Billing
- Stripe implementation remains blocked until Victor approval.

## Gates

1. PR #9 merge — DONE
2. Migration apply — WAITING AUTHORIZATION
3. Team validation
4. Merge PR #12
5. Team Permissions Editing
6. Billing Entitlements
