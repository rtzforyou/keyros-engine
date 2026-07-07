# Runbook — Apply / Test / Rollback for PR #9 Team migration

```text
Task: Task 1 — Apply/test/rollback runbook for Team migration
Agent: Claude
Status: needs_review (runbook only — NOTHING applied to production)
Product PR: #9
Migration: supabase/migrations/20260711000000_team_users_columns_and_signup.sql
Head SHA: 9a2859f4bb6291664a74ad9810b969537f85271a
Engine branch: claude/task1-team-migration-runbook
Report path: agent-room/reports/2026-07-07-claude-task1-team-migration-apply-test-runbook.md
Permission requested from Victor: yes
```

This runbook is documentation only. Do NOT run any of it until Task 2 is explicitly authorized ("Approve Task 2 — apply Team migration to Supabase"). All test blocks are wrapped in `BEGIN … ROLLBACK` so they leave no trace; the actual apply step is the only non-rollback action and is gated on Task 2.

## Fixed test UUIDs (fake — safe to reuse)
- org A (inviting studio): `aaaaaaaa-0000-0000-0000-000000000001`
- org B (a different studio): `bbbbbbbb-0000-0000-0000-000000000002`
- admin of org A: `aaaaaaaa-1111-1111-1111-111111111111`
- member (non-admin) of org A: `aaaaaaaa-2222-2222-2222-222222222222`
- invitee auth id (signup-with-invite): `cccccccc-3333-3333-3333-333333333333`
- normal signup auth id: `dddddddd-4444-4444-4444-444444444444`
- invite token: `eeeeeeee-5555-5555-5555-555555555555`

---

## STEP A — Apply (Task 2 only; do NOT run under Task 1)

Preferred: apply via the standard migration path so it is recorded.
```
mcp apply_migration  name=team_users_columns_and_signup  (contents of 20260711000000_...sql)
```
Pre-apply safety probe (optional, non-persistent) — run the whole migration inside a transaction and roll back, to confirm it executes without error before the real apply:
```sql
BEGIN;
\i 20260711000000_team_users_columns_and_signup.sql   -- or paste the file body
ROLLBACK;
```

Post-apply sanity:
```sql
select column_name from information_schema.columns
where table_schema='public' and table_name='users'
  and column_name in ('permissions','email','avatar_url');   -- expect 3 rows
select tgname from pg_trigger where tgrelid='public.users'::regclass
  and tgname='protect_sensitive_user_columns_trg';           -- expect 1 row
```

---

## STEP B — Functional tests (each self-contained, BEGIN…ROLLBACK)

### Shared setup (prepended to each test block below)
```sql
BEGIN;
insert into public.organizations (id,name) values
  ('aaaaaaaa-0000-0000-0000-000000000001','Studio A'),
  ('bbbbbbbb-0000-0000-0000-000000000002','Studio B');
insert into public.users (id,organization_id,role,full_name,email,permissions) values
  ('aaaaaaaa-1111-1111-1111-111111111111','aaaaaaaa-0000-0000-0000-000000000001','admin','Admin A','admin@a.com','[]'::jsonb),
  ('aaaaaaaa-2222-2222-2222-222222222222','aaaaaaaa-0000-0000-0000-000000000001','tattooer','Member A','member@a.com','["calendar"]'::jsonb);
-- ... test-specific statements ...
ROLLBACK;
```

### B1 — normal signup without invite → creates own org, admin, email/permissions set
```sql
BEGIN;
insert into auth.users (id,email,raw_user_meta_data)
  values ('dddddddd-4444-4444-4444-444444444444','owner@new.com','{"full_name":"Owner","studio_name":"New Studio"}'::jsonb);
-- EXPECT: exactly one public.users row for this id, role='admin', email='owner@new.com',
--         permissions='[]', organization_id = a brand-new org named 'New Studio'.
select role, email, permissions::text,
       (select name from public.organizations o where o.id=u.organization_id) as org
from public.users u where u.id='dddddddd-4444-4444-4444-444444444444';
ROLLBACK;
```

### B2 — signup with pending invite → joins invited org, role+permissions from invite
```sql
BEGIN;
insert into public.organizations (id,name) values ('aaaaaaaa-0000-0000-0000-000000000001','Studio A');
insert into public.invitations (id,organization_id,email,role,permissions,status,expires_at,token)
  values ('11111111-aaaa-aaaa-aaaa-111111111111','aaaaaaaa-0000-0000-0000-000000000001',
          'invitee@x.com','staff','["dashboard","calendar"]'::jsonb,'pending', now()+interval '7 days',
          'eeeeeeee-5555-5555-5555-555555555555');
insert into auth.users (id,email,raw_user_meta_data)
  values ('cccccccc-3333-3333-3333-333333333333','invitee@x.com','{"full_name":"Invitee"}'::jsonb);
-- EXPECT: user linked to Studio A, role='staff', permissions='["dashboard","calendar"]';
--         invite row now status='accepted'.
select u.role, u.permissions::text, (u.organization_id='aaaaaaaa-0000-0000-0000-000000000001') as joined_invited_org,
       (select status from public.invitations where id='11111111-aaaa-aaaa-aaaa-111111111111') as invite_status
from public.users u where u.id='cccccccc-3333-3333-3333-333333333333';
ROLLBACK;
```

### B3 — member updates own full_name / avatar_url → ALLOWED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-2222-2222-2222-222222222222"}';
update public.users set full_name='Member Renamed', avatar_url='https://x/y.png'
  where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: succeeds (0 errors), full_name/avatar_url updated.
select full_name, avatar_url from public.users where id='aaaaaaaa-2222-2222-2222-222222222222';
ROLLBACK;
```

### B4 — member updates own permissions → BLOCKED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-2222-2222-2222-222222222222"}';
update public.users set permissions='["dashboard","contacts","pipeline","payments"]'::jsonb
  where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: ERROR 'Not allowed to change role, permissions, or organization_id'.
ROLLBACK;
```

### B5 — member updates own role → BLOCKED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-2222-2222-2222-222222222222"}';
update public.users set role='admin' where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: ERROR 'Not allowed to change role, permissions, or organization_id'.
ROLLBACK;
```

### B6 — member updates own organization_id → BLOCKED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-2222-2222-2222-222222222222"}';
update public.users set organization_id='bbbbbbbb-0000-0000-0000-000000000002'
  where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: ERROR 'Not allowed to change role, permissions, or organization_id'.
ROLLBACK;
```

### B7 — admin updates member permissions → ALLOWED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-1111-1111-1111-111111111111"}';  -- admin A
update public.users set permissions='["dashboard","calendar","contacts"]'::jsonb
  where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: succeeds. (RLS 'Admins can update organization members' + guard allows admin.)
select permissions::text from public.users where id='aaaaaaaa-2222-2222-2222-222222222222';
ROLLBACK;
```

### B8 — admin updates member role → ALLOWED
```sql
BEGIN;
-- shared setup here --
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-1111-1111-1111-111111111111"}';  -- admin A
update public.users set role='staff' where id='aaaaaaaa-2222-2222-2222-222222222222';
-- EXPECT: succeeds.
select role from public.users where id='aaaaaaaa-2222-2222-2222-222222222222';
ROLLBACK;
```

### B9 (regression) — accept_invitation() still works for a non-admin invitee → ALLOWED
```sql
BEGIN;
insert into public.organizations (id,name) values ('aaaaaaaa-0000-0000-0000-000000000001','Studio A');
insert into public.invitations (id,organization_id,email,role,permissions,status,expires_at,token)
  values ('22222222-bbbb-bbbb-bbbb-222222222222','aaaaaaaa-0000-0000-0000-000000000001',
          'existing@x.com','staff','["dashboard"]'::jsonb,'pending', now()+interval '7 days',
          'ffffffff-6666-6666-6666-666666666666');
-- an already-existing non-admin user in another org accepts the invite:
insert into public.organizations (id,name) values ('bbbbbbbb-0000-0000-0000-000000000002','Studio B');
insert into public.users (id,organization_id,role,full_name,email,permissions)
  values ('aaaaaaaa-2222-2222-2222-222222222222','bbbbbbbb-0000-0000-0000-000000000002','tattooer','X','x@x.com','[]'::jsonb);
set local role authenticated;
set local request.jwt.claims = '{"sub":"aaaaaaaa-2222-2222-2222-222222222222"}';
select * from public.accept_invitation('ffffffff-6666-6666-6666-666666666666');
-- EXPECT: succeeds (bypass GUC lets this controlled self-transition through the guard);
--         user now org=Studio A, role='staff', permissions='["dashboard"]'; invite='accepted'.
select organization_id::text, role, permissions::text from public.users where id='aaaaaaaa-2222-2222-2222-222222222222';
ROLLBACK;
```

## Pass criteria
B1,B2,B3,B7,B8,B9 succeed with the expected rows; B4,B5,B6 each raise `Not allowed to change role, permissions, or organization_id`. Any deviation → do not merge/keep applied; roll back (STEP C) and report.

---

## STEP C — Rollback (if the applied migration must be reverted)

Because everything sensitive is additive, rollback is a follow-up migration (no data loss for core fields):
```sql
-- 1. remove the guard
drop trigger if exists protect_sensitive_user_columns_trg on public.users;
drop function if exists public.protect_sensitive_user_columns();

-- 2. restore accept_invitation() to the 20260710000000 body (without the bypass + without permissions copy)
--    [paste the original accept_invitation from 20260710000000]

-- 3. restore handle_new_user() to the 20260710000000 body (without email/permissions columns)
--    [paste the original handle_new_user from 20260710000000]

-- 4. drop the new columns
alter table public.users drop column if exists permissions;
alter table public.users drop column if exists email;
alter table public.users drop column if exists avatar_url;
```
Note: dropping `permissions`/`email`/`avatar_url` discards any data written into them after apply. If members were already edited via the future Team UI, export those values first. If the migration is reverted BEFORE any Team UI writes exist, there is nothing to preserve.

Pre-apply, "rollback" is trivial: do not merge PR #9 / do not apply.

## Compliance
Runbook only. Nothing applied to Supabase. No product code changed. No PR merged. No Antenor branch touched.

Signed, Claude — Heavy Implementation Agent
