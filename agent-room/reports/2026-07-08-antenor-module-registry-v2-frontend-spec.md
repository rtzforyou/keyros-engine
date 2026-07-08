# Technical Report: Frontend Module UI & Billing Entitlements Specifications

This document outlines the UI plans, UX states, hardcoded values audit, and implementation roadmaps for integrating the Canonical Module Registry v2 and seat controls in the frontend.

---

## 1. Locked-State UI & Entitlement Badges

When a module is unavailable in the organization's current plan, the interface restricts access gracefully:

- **Sidebar Items:** Locked links render with:
  - Opacity reduced to `60%`.
  - A gold key/lock icon `[PRO]` or `[VIP]` next to the label.
  - Intercepted click handlers that trigger the **Upgrade Prompt Modal** instead of executing routes.
- **Direct Link Router Protection:** Middleware checks the active organization plan on route change. Unentitled direct URL requests redirect to a custom `/settings?tab=billing` upgrade portal.

---

## 2. Team Member Permissions Checkbox Grid

Admins manage member permissions through an interactive grid inside the "Edit Member" modal:

- **Layout:** Assignable modules are organized in columns with checkboxes.
- **Plan Enforcement:** If a module is locked by the organization plan (e.g. `agents` on a Solo plan), its checkbox is disabled (`disabled={true}`), styled with a lock icon, and highlights an upgrade prompt tooltip on hover.

---

## 3. Invite Seat Limit Warning UI

Banners are rendered dynamically on the **Team Management** screen:

- **Usage Status Banner:** Shows `Uso de Limite: [X] / [Y] membros` inside the Team settings.
- **Action Blocking:** When `seatUsage >= seatLimit`, the "Invite User" button is disabled. A warning banner appears: `Limite de utilizadores atingido para o plano atual. Faça upgrade para adicionar novos membros.` next to an **Upgrade Plan** CTA.

---

## 4. Navigation Visibility Rules

- **Admins:** Full view of settings, billing, and team tabs.
- **Staff / Tattooers:** Sidebar navigation filter reads the user's `permissions` array (fetched via auth profile) and renders only authorized module options, preventing discovery of admin-level configurations.

---

## 5. Audit Report: Hardcoded Values

We audited the frontend codebase and identified the following hardcoded elements that should be cleaned:
1. **Locale formatters:** Hardcoded currency BRL in `DashboardContent.tsx` (resolved in Loop 1).
2. **Mock list defaults:** Mock users in `useMockData.ts` (currently being eliminated domain by domain).
3. **Hardcoded status checks:** In `useLeads.ts` line 26: `"New Lead"` is hardcoded for Prospect fallback. This should be mapped to the translations registry.

---

## 6. Frontend Slices Roadmap

1. **Slice 1 (Plan Mappings):** Update `useBillingLimits.ts` hook to support the 4 canonical plans (`solo`, `team`, `business`, `enterprise`) and active seat configurations.
2. **Slice 2 (Interface Lock):** Add sidebar locks, opacity styles, and modal upgrade prompts for disabled modules.
3. **Slice 3 (Permissions Editor):** Build the checkbox configuration grid in `TeamContent` edit member dialogue.
