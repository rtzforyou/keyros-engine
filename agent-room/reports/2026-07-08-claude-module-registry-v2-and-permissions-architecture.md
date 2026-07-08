# Technical Report: Canonical Module Registry v2 & Permissions Architecture

This document presents the finalized architecture designs for the Canonical Module Registry v2, editable permissions, billing limits validation triggers, and security parameters.

---

## 1. Canonical Module Registry v2

To enforce consistency across client, server, and reports, Keyros CRM adopts a unified **11-module domain registry** v2.

### 1.1 Typescript Definitions (`types.ts`)
```typescript
export type KeyrosModule =
  | 'dashboard'
  | 'crm'
  | 'calendar'
  | 'messages'
  | 'automations'
  | 'team'
  | 'billing'
  | 'finance'
  | 'reports'
  | 'settings'
  | 'agents';

export interface ModuleDefinition {
  id: KeyrosModule;
  labelEn: string;
  labelPt: string;
  descriptionPt: string;
  isTeamAssignable: boolean; // Settings/Team/Billing are admin-only
}
```

### 1.2 Normalization of Legacy Aliases
Legacy references across existing components and databases will be mapped to the canonical module registry using a helper function:

```typescript
export function normalizeModuleAlias(alias: string): KeyrosModule {
  const mapping: Record<string, KeyrosModule> = {
    contacts: 'crm',
    deals: 'crm',
    pipeline: 'crm',
    payments: 'finance',
    expenses: 'finance',
    ai: 'agents',
    aiAgent: 'agents',
  };
  return mapping[alias] || (alias as KeyrosModule);
}
```

---

## 2. Database Design for Editable Permissions

Admins will update a team member's permissions by executing an `UPDATE` on `public.users` targeting `role` and `permissions`.

### 2.1 Supabase DDL Validation Constraint
To ensure integrity at the database layer, we enforce a check constraint on `public.users` to prevent the injection of invalid module strings:

```sql
-- DDL migration file: supabase/migrations/20260712000000_validate_module_permissions_v2.sql

ALTER TABLE public.users DROP CONSTRAINT IF EXISTS check_permissions_valid_modules_v2;
ALTER TABLE public.users ADD CONSTRAINT check_permissions_valid_modules_v2
  CHECK (
    jsonb_typeof(permissions) = 'array' AND
    (
      SELECT COALESCE(bool_and(value->>0 IN (
        'dashboard', 'crm', 'calendar', 'messages', 'automations',
        'team', 'billing', 'finance', 'reports', 'settings', 'agents'
      )), true)
      FROM jsonb_array_elements(permissions) AS value
    )
  );
```

### 2.2 Security Trigger & Guard Compliance
- **Admin Isolation:** The database trigger `protect_sensitive_user_columns_trg` intercepts updates. If the acting user (`auth.uid()`) is not an admin, any change to `role`, `permissions`, or `organization_id` raises a Postgres exception.
- **RLS isolation:** Admins are locked into their own organization via:
  ```sql
  CREATE POLICY "Admins can update organization members" ON public.users
    FOR UPDATE USING (
      organization_id = public.get_user_org_id() AND
      (SELECT role FROM public.users WHERE id = auth.uid()) = 'admin'
    );
  ```

---

## 3. Plan Keys & Seat Limits Architecture

Subscribers are allocated access limits according to their active tier.

### 3.1 Plan Tiers Map
| Plan Key | Monthly Cost | Seat Limit | Included Modules |
| :--- | :--- | :--- | :--- |
| `'solo'` / null | €0 | **1 seat** | `['dashboard', 'crm', 'calendar', 'messages', 'settings']` |
| `'team'` | €29 | **3 seats** | `['dashboard', 'crm', 'calendar', 'messages', 'automations', 'finance', 'settings', 'team', 'billing']` |
| `'business'` | €49 | **10 seats** | `['dashboard', 'crm', 'calendar', 'messages', 'automations', 'finance', 'settings', 'team', 'billing', 'reports']` |
| `'enterprise'` | €120 | **Unlimited (999)**| All modules (`dashboard`, `crm`, `calendar`, `messages`, `automations`, `team', `billing`, `finance`, `reports`, `settings`, `agents`) |

### 3.2 Database Trigger for Seat Limit Checks
To prevent limits bypass (e.g. sending multiple invitations to exceed tier limits), we attach a trigger checking both active users and pending invitations:

```sql
CREATE OR REPLACE FUNCTION public.enforce_invitation_seat_limit()
RETURNS TRIGGER AS $$
DECLARE
  current_seats INTEGER;
  allowed_seats INTEGER;
  org_plan VARCHAR(50);
BEGIN
  -- Count active members
  SELECT COUNT(*) INTO current_seats 
  FROM public.users 
  WHERE organization_id = NEW.organization_id;

  -- Add pending invitations to prevent limits oversaturation
  SELECT current_seats + COUNT(*) INTO current_seats 
  FROM public.invitations 
  WHERE organization_id = NEW.organization_id 
    AND status = 'pending';

  -- Resolve organization plan
  SELECT COALESCE(plan_type, 'solo') INTO org_plan 
  FROM public.organizations 
  WHERE id = NEW.organization_id;

  IF org_plan = 'team' THEN
    allowed_seats := 3;
  ELSIF org_plan = 'business' THEN
    allowed_seats := 10;
  ELSIF org_plan = 'enterprise' THEN
    allowed_seats := 999;
  ELSE
    allowed_seats := 1; -- Solo / Default
  END IF;

  IF current_seats >= allowed_seats THEN
    RAISE EXCEPTION 'Seat limit of % users reached for organization on % plan.', allowed_seats, org_plan;
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS enforce_invitation_seat_limit_trg ON public.invitations;
CREATE TRIGGER enforce_invitation_seat_limit_trg
  BEFORE INSERT ON public.invitations
  FOR EACH ROW EXECUTE FUNCTION public.enforce_invitation_seat_limit();
```

---

## 4. Billing Entitlements & Stripe Separation

The Stripe payment checkout logic remains completely separated from Keyros entitlement enforcement.

- **Isolation Strategy:** Payment webhooks only update the `plan_type` and `subscription_status` on `public.organizations`. 
- **Entitlements Check:** Keyros queries the organization plan details via RLS policies and exposes the active limits through standard hooks, avoiding any direct references to Stripe APIs.

---

## 5. Dashboard Real-Data & Security Review Plans

### 5.1 Real-Data Dashboard Calculations
- **Gross Revenue:** Calculated from the sum of deal values in `public.deals` matching `financial_status = 'won'`.
- **Expenses:** Calculated from the sum of transaction amounts in `public.financial_transactions` where type is `'expense'`.
- **Conversion Rate:** Calculated as: `won_deals / (won_deals + lost_deals) * 100`.

### 5.2 Security Parameters
- Validate that SQL constraints enforce permissions format compatibility.
- Ensure triggers raise database exceptions if a compromised non-admin attempts to forge roles.

---

## 6. Organization Entitlement Resolution Function

To resolve plan entitlements database-side:

```sql
CREATE OR REPLACE FUNCTION public.resolve_organization_entitlement(org_id UUID, module_name TEXT)
RETURNS BOOLEAN AS $$
DECLARE
  org_plan VARCHAR(50);
  allowed_modules TEXT[];
BEGIN
  -- Fetch organization plan
  SELECT COALESCE(plan_type, 'solo') INTO org_plan FROM public.organizations WHERE id = org_id;

  -- Map plan to allowed modules list
  IF org_plan = 'enterprise' THEN
    RETURN true;
  ELSIF org_plan = 'business' THEN
    allowed_modules := ARRAY['dashboard', 'crm', 'calendar', 'messages', 'automations', 'finance', 'settings', 'team', 'billing', 'reports'];
  ELSIF org_plan = 'team' THEN
    allowed_modules := ARRAY['dashboard', 'crm', 'calendar', 'messages', 'automations', 'finance', 'settings', 'team', 'billing'];
  ELSE
    allowed_modules := ARRAY['dashboard', 'crm', 'calendar', 'messages', 'settings']; -- Solo
  END IF;

  RETURN module_name = ANY(allowed_modules);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## 7. Member Permission Resolution Function

Resolves if a user has access to a specific module:

```sql
CREATE OR REPLACE FUNCTION public.check_user_module_access(user_id UUID, module_name TEXT)
RETURNS BOOLEAN AS $$
DECLARE
  user_record RECORD;
  has_entitlement BOOLEAN;
BEGIN
  -- Fetch user profile
  SELECT organization_id, role, permissions INTO user_record FROM public.users WHERE id = user_id;

  -- 1. Check organization plan entitlement
  SELECT public.resolve_organization_entitlement(user_record.organization_id, module_name) INTO has_entitlement;
  IF NOT has_entitlement THEN
    RETURN false;
  END IF;

  -- 2. Check role and permissions list
  IF user_record.role = 'admin' THEN
    RETURN true;
  ELSE
    RETURN (user_record.permissions @> jsonb_build_array(module_name));
  END IF;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## 8. Audit Event Model for Team and Plan Changes

We automatically log permissions modifications:

```sql
CREATE OR REPLACE FUNCTION public.log_user_security_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.role IS DISTINCT FROM OLD.role OR NEW.permissions IS DISTINCT FROM OLD.permissions THEN
    INSERT INTO public.audit_logs (user_id, organization_id, action, metadata)
    VALUES (
      auth.uid(),
      NEW.organization_id,
      'user_permissions_updated',
      jsonb_build_object(
        'target_user_id', NEW.id,
        'old_role', OLD.role,
        'new_role', NEW.role,
        'old_permissions', OLD.permissions,
        'new_permissions', NEW.permissions
      )
    );
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## 9. Rollback Plan

To rollback the database schema:
```sql
ALTER TABLE public.users DROP CONSTRAINT IF EXISTS check_permissions_valid_modules_v2;
DROP TRIGGER IF EXISTS enforce_invitation_seat_limit_trg ON public.invitations;
DROP FUNCTION IF EXISTS public.enforce_invitation_seat_limit();
DROP FUNCTION IF EXISTS public.check_user_module_access(uuid, text);
DROP FUNCTION IF EXISTS public.resolve_organization_entitlement(uuid, text);
```

---

## 10. Migration & Security Matrix

### 10.1 Migration Checklist
- Ensure no users have invalid legacy strings (e.g. `contacts`, `ai`) in permissions.
- Validate triggers are registered successfully.

### 10.2 Test Matrix
- **Admin:** full access to all organization-enabled modules.
- **Member:** access restricted to checked options.
- **Invited User:** inherits permissions set in invitation upon signup.

---

## 11. Backend Roadmap

1. **Slice 1 (Enforcement):** Validate permissions check constraint and resolver functions.
2. **Slice 2 (Limits):** Implement invite seat limits database-side trigger check.
3. **Slice 3 (Editing):** Implement UI updating hook in TeamContent.
