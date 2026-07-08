# Authenticated Smoke Tests Checklist — Keyros CRM

This checklist defines the validation steps for verifying the core UI modules, navigation safety, and states under an authenticated user session in Keyros.

## 1. Auth & Login Flow

- [ ] **Login screen render:** Verify Google Login button displays correctly, styled with default branding and layout alignment.
- [ ] **Auth redirection:** Verify that an unauthenticated user attempting to access dashboard/settings/pipeline is redirected back to the Login screen.
- [ ] **Auth state resolution:** Verify the loader is triggered during session establishment, followed by rendering the correct dashboard view upon successful authentication.
- [ ] **Error states:** Verify that auth failure alerts are displayed visually without causing app crashes (white screen).

## 2. Navigation & Shell

- [ ] **Navigation buttons:** Click through all sidebar options (Dashboard, Contacts, Pipeline, Calendar, Messages, Automations, Settings). Verify each transition is immediate.
- [ ] **Console check:** Open browser dev tools and confirm no new errors or warnings are logged during navigation switches.
- [ ] **URL parameter routing:** Access manual route params (e.g. `?debug=automations`) to verify background diagnostics load safely.
- [ ] **White-screen prevention:** Verify that clicking outside boundaries or invalid routing triggers fallback states rather than a complete UI crash.

## 3. Dashboard Verification

- [ ] **KPI card loading:** Verify that Total Revenue, Conversion Rate, Deals Won, and Active Contacts show real data (or safe empty fallbacks if there is no data) instead of raw mock numbers.
- [ ] **Graph rendering:** Confirm Recharts loads without sizing layout breakages or legend overlapping.
- [ ] **Date filter updates:** Toggle period filters (e.g., 7 days, 30 days) and verify stats refresh correctly.

## 4. Settings View

- [ ] **Tabs rendering:** Verify all tabs render without styling conflicts (Profile, Studio, Preferences, Integrations, Landing Pages, Billing).
- [ ] **Leftover Forms verification:** Verify that "Forms Settings" or forms indicators are completely absent from the Settings tabs list.
- [ ] **Google/Wix forms integrations:** Ensure external integration options in the Landing Pages tab remain functional and editable.

## 5. Team Page (Read-Only State)

- [ ] **Real data loading:** Once the hook `useTeamMembers` is active, verify the table lists members fetched from `public.users` with correct email, role, and avatar.
- [ ] **Read-only validation:** Confirm that edit/remove member options are hidden or display mock placeholders, ensuring no write calls to `role` or `permissions` columns can be triggered.
- [ ] **Empty list state:** Verify the table displays a friendly empty state if the current organization has no other members.

## 6. Verification Metrics

- [ ] **TypeScript Check:** `npx tsc --noEmit` must pass with 0 errors.
- [ ] **Production Build:** `npm run build` must run successfully, generating optimized assets inside `dist/`.
