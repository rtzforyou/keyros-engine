# Technical Plan: Canonical Module Registry & Team Permissions Editing

This plan outlines the architecture, database constraints, API hook designs, and validation steps for implementing the **Canonical Module Registry** and **Team Permissions Editing** in Keyros CRM.

---

## 1. Canonical Module Registry Architecture

The module registry serves as the source of truth for features, seat controls, and subscription checks.

### 1.1 Typescript Definitions
We define the 12 canonical modules in `types.ts` as a union type:
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
  isTeamAssignable: boolean; // Settings/Team are admin-only
}
```

### 1.2 Registry Definition
A central registry registry object will be declared in a new file `lib/moduleRegistry.ts`:
```typescript
export const MODULE_REGISTRY: Record<KeyrosModule, ModuleDefinition> = {
  dashboard: { id: 'dashboard', labelEn: 'Dashboard', labelPt: 'Painel Principal', descriptionPt: 'Acesso às métricas gerais do estúdio', isTeamAssignable: true },
  crm: { id: 'crm', labelEn: 'CRM', labelPt: 'CRM / Clientes', descriptionPt: 'Gestão de contactos, funil e pipeline', isTeamAssignable: true },
  calendar: { id: 'calendar', labelEn: 'Calendar', labelPt: 'Agenda', descriptionPt: 'Marcação de sessões e horários', isTeamAssignable: true },
  messages: { id: 'messages', labelEn: 'Messages', labelPt: 'Mensagens', descriptionPt: 'Integração de chat e WhatsApp', isTeamAssignable: true },
  automations: { id: 'automations', labelEn: 'Automations', labelPt: 'Automações', descriptionPt: 'Regras automáticas e templates', isTeamAssignable: true },
  team: { id: 'team', labelEn: 'Team', labelPt: 'Equipa', descriptionPt: 'Gestão de membros e acessos (Admin)', isTeamAssignable: false },
  billing: { id: 'billing', labelEn: 'Billing', labelPt: 'Faturamento', descriptionPt: 'Subscrições, planos e faturas (Admin)', isTeamAssignable: false },
  finance: { id: 'finance', labelEn: 'Finance', labelPt: 'Financeiro', descriptionPt: 'Controle de despesas e receitas', isTeamAssignable: true },
  reports: { id: 'reports', labelEn: 'Reports', labelPt: 'Relatórios', descriptionPt: 'Relatórios de faturação e KPIs', isTeamAssignable: true },
  settings: { id: 'settings', labelEn: 'Settings', labelPt: 'Definições', descriptionPt: 'Configurações do estúdio (Admin)', isTeamAssignable: false },
  agents: { id: 'agents', labelEn: 'Agents', labelPt: 'Assistente IA', descriptionPt: 'Sugestões automáticas por IA', isTeamAssignable: true },
};
```

---

## 2. Database Integrity Constraints (Supabase DDL)

To enforce permissions integrity database-side, we will introduce a CHECK constraint to guarantee that the `permissions` JSONB column only contains valid module array elements.

```sql
-- DDL migration file: supabase/migrations/20260712000000_validate_module_permissions.sql

ALTER TABLE public.users DROP CONSTRAINT IF EXISTS check_permissions_valid_modules;
ALTER TABLE public.users ADD CONSTRAINT check_permissions_valid_modules
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

---

## 3. Team Permissions Editing Implementation Plan

Currently, updating a member's role or permissions in the UI is read-only and triggers a browser `alert()`. We will replace this with a secure API update flow.

### 3.1 Backend Security Rules
- **RLS Policy:** The existing `"Admins can update organization members"` policy on `public.users` permits updates where the updating user is an admin and matches the member's `organization_id`.
- **Database Trigger:** The `protect_sensitive_user_columns_trg` trigger blocks any non-admin (role is not `'admin'`) from altering `role` or `permissions` columns, even for their own user record.

### 3.2 UI Integration Code Change
We will update `TeamContent.tsx` to execute a Supabase `update` operation:
```typescript
const handleUpdateMember = async () => {
  if (!memberToEdit) return;
  setIsSaving(true);
  try {
    const { error } = await supabase
      .from('users')
      .update({
        role: editRole,
        permissions: editPermissions
      })
      .eq('id', memberToEdit.id)
      .eq('organization_id', organizationId); // Multi-tenant guard

    if (error) throw error;
    
    setIsEditMemberDialogOpen(false);
    setMemberToEdit(null);
    refetch(); // Reload the members table
  } catch (err: any) {
    setError(err.message);
  } finally {
    setIsSaving(false);
  }
};
```

---

## 4. Billing/Seat Limit Database Hook

When an admin invites a user or a user signs up, the system must validate plan entitlements and seat limits.

### 4.1 Database Check Function
We will define a postgres function to query active seat counts and assert limits:
```sql
CREATE OR REPLACE FUNCTION public.check_organization_seat_limit(org_id UUID)
RETURNS BOOLEAN AS $$
DECLARE
  current_seats INTEGER;
  allowed_seats INTEGER;
  mock_plan TEXT;
BEGIN
  -- 1. Fetch current active seats in organization
  SELECT COUNT(*) INTO current_seats FROM public.users WHERE organization_id = org_id;

  -- 2. Mock plan lookup (can be integrated with a real plan/subscription table in future)
  -- For now, default to free (2 seats), premium (5 seats), enterprise (unlimited/999)
  -- Plan lookup can check against public.organizations plan fields.
  SELECT COALESCE(plan_type, 'free') INTO mock_plan FROM public.organizations WHERE id = org_id;

  IF mock_plan = 'premium' THEN
    allowed_seats := 5;
  ELSIF mock_plan = 'enterprise' THEN
    allowed_seats := 999;
  ELSE
    allowed_seats := 2; -- Free
  END IF;

  RETURN current_seats < allowed_seats;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

This function can be attached as a trigger to the `public.invitations` table before inserts to prevent exceeding seats database-side.
