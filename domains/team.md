# Team Domain

Team is a core Keyros domain and must be organized before commercial use.

## Purpose

Control who can access an organization, what role they have, and which modules they can use.

## Core concepts

```text
Organization
└── Members
    ├── Owner/Admin
    ├── Tattooer
    └── Staff

Invitation
└── Pending access request by email/token

Permissions
└── Module-level access rules
```

## Current decision

Team members and permissions must become real product data.

Invitations already exist as a real flow, but members and permission editing must not stay in local/mock state.

## Required behavior

- Organization owner/admin can invite users.
- Invitation creates pending access for a specific email.
- Accepted invitation creates or updates real membership.
- Member role and permissions are persisted.
- Permissions are enforced beyond the frontend.
- Removed member loses access to the organization.
- Role changes are audited.

## Required roles

- `admin`: full organization access.
- `tattooer`: limited operational access, especially calendar, contacts, pipeline and assigned work.
- `staff`: configurable access based on selected modules.

## Required permissions

Base modules:

- dashboard
- contacts
- pipeline
- calendar
- messages
- payments
- expenses
- automations
- team
- settings

## Security rules

- Never trust frontend-only permission checks.
- Every sensitive query/mutation must be organization-scoped.
- RLS must protect memberships and organization data.
- Invitation token must be single-use or revocable.
- Invitation token must expire.
- User email must match invite email or require explicit admin approval.

## Implementation priority

1. Replace `teamMembers` from `useMockData` with real membership data.
2. Persist role and permissions.
3. Connect invite acceptance to membership creation/update.
4. Add server-side/RLS enforcement.
5. Add audit trail for role/permission changes.
6. Remove team-related local state from `useMockData`.
