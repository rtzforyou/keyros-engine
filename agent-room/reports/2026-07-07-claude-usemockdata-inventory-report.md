# Report — useMockData Inventory (Step 1, completed)

## Inventory state

Inspected product repo `rtzforyou/easytattoo-crm` at `origin/main = 4ca84a0` (latest merged state: includes Antenor's Forms-tab removal and Claude's invite-aware signup fix).

Note on in-flight work: Antenor's branch `antenor/remove-forms-module-strings` (`72deedd`, "remove Forms from static modules list in App and Login") is pushed but **not yet merged to main**. It only removes the `'forms'` string from `ALL_MODULES` in `App.tsx`/`Login.tsx` — it does **not** change any `useMockData` import/usage, so this inventory is unaffected by it. Not touched.

## Per-file inventory

### File: App.tsx
- What it uses from useMockData: `isLoading`, `error: dataError` only
- Domain: app bootstrap (loading/error gate at line 184)
- Risk: low
- Recommended action: replace (leftover)
- Suggested owner: Claude
- Suggested isolated task: replace the loading/error gate with a lightweight real source (or remove if no longer meaningful) so App.tsx stops instantiating the 957-line hook just to read a flag. Low priority — do after higher-value domains.

### File: components/settings/FormsSettings.tsx
- What it uses from useMockData: `formTemplates`, `addFormTemplate`, `updateFormTemplate`, `archiveFormTemplate`
- Domain: Forms
- Risk: low
- Recommended action: remove (dead file — no longer imported anywhere after Antenor removed the Forms tab)
- Suggested owner: Antenor
- Suggested isolated task: delete `components/settings/FormsSettings.tsx` and any now-unused translation keys it referenced; confirm `tsc` clean.

### File: components/contacts/ContactFormsTab.tsx
- What it uses from useMockData: `formSubmissions`, `formTemplates`, `addFormSubmission`, `updateFormSubmission`
- Domain: Forms (inside Contacts)
- Risk: low
- Recommended action: remove (dead file — no longer imported anywhere)
- Suggested owner: Antenor
- Suggested isolated task: delete `components/contacts/ContactFormsTab.tsx`; confirm no Contacts view still references it; `tsc` clean.

### File: components/automations/WorkflowTabContent.tsx
- What it uses from useMockData: `workflows`, `addWorkflow`, `updateWorkflow`, `deleteWorkflow`, `toggleWorkflowStatus`
- Domain: Automations / Workflows
- Risk: low
- Recommended action: remove (dead file — no longer imported anywhere; ADR-0001 already removed Workflows from core)
- Suggested owner: Antenor
- Suggested isolated task: delete `components/automations/WorkflowTabContent.tsx` and its now-unused workflow translation keys; `tsc` clean.

### File: components/team/TeamContent.tsx
- What it uses from useMockData: `teamMembers`, `updateTeamMember`, `removeTeamMember`
- Domain: Team
- Risk: medium
- Recommended action: replace with real membership model
- Suggested owner: Claude
- Suggested isolated task: plan + implement real Team membership (real `users`/membership reads, role/permission persistence, server-side validation). Plan first, pause for Victor approval before implementation (plan Steps 4–5).

### File: components/payments/PaymentsContent.tsx
- What it uses from useMockData: `payments`, `contacts`, `addPayment`, `formatCurrency`
- Domain: Payments (client payments)
- Risk: medium
- Recommended action: replace
- Suggested owner: Claude
- Suggested isolated task: plan the client-payments / app-billing split (Supabase-first, manual methods, no Stripe), then implement client payments (plan Steps 6–7). Plan first, pause for approval.

### File: components/calendar/CalendarSettings.tsx
- What it uses from useMockData: `tattooers`, `addTattooer`, `removeTattooer`, `businessHours`, `updateBusinessHours`
- Domain: Calendar / Tattooers / Business hours
- Risk: medium
- Recommended action: replace (split into two sub-tasks)
- Suggested owner: ChatGPT decision needed (split ownership)
- Suggested isolated task: (a) Antenor — swap the `tattooers` reads to the already-existing `hooks/useTattooers.ts`; (b) Claude — `businessHours` has NO backing table (confirmed: no `business_hours` in any migration), needs a schema decision + migration + hook. Distinct file from `CalendarContent.tsx`, already migrated to real hooks earlier (backlog is stale on that).

### File: components/automations/AutomationsContent.tsx
- What it uses from useMockData: `automationTriggers`
- Domain: Automations
- Risk: low
- Recommended action: replace with a real trigger registry
- Suggested owner: ChatGPT decision needed
- Suggested isolated task: decide approach first — Antenor if it's just moving a static list into a code-level registry module; Claude if it becomes DB-backed. Execution engine is already real via `useAutomations` (plan Step 9).

### File: components/messages/AIAssistantDialog.tsx
- What it uses from useMockData: `quickMessages`
- Domain: Messaging (WhatsApp quick replies)
- Risk: low
- Recommended action: replace
- Suggested owner: Antenor
- Suggested isolated task: move `quickMessages` (translated quick-reply strings) out of `useMockData` into a small dedicated source/module; confirm the AI assistant dialog still lists them.

### File: components/pipeline/PipelineContent.tsx
- What it uses from useMockData: `formatDistanceToNow`, `formatCurrency` (pure utility functions, not mock domain data)
- Domain: none — formatting utilities only
- Risk: low
- Recommended action: replace (extract to `lib/formatters.ts`)
- Suggested owner: Antenor
- Suggested isolated task: extract the two formatter helpers into `lib/formatters.ts` and import them directly, removing the `useMockData` dependency from PipelineContent.

### File: hooks/useMockData.ts
- What it uses from useMockData: n/a — this is the hook itself (957 lines)
- Domain: cross-cutting
- Risk: low (in isolation)
- Recommended action: keep temporarily
- Suggested owner: Claude
- Suggested isolated task: delete/quarantine only after every domain above is migrated and zero active imports remain (plan Step 11).

## Summary

- Total files using useMockData: 10 product files + the hook itself (`hooks/useMockData.ts`).
- High-risk items: 0.
- Medium-risk items: 3 (TeamContent.tsx, PaymentsContent.tsx, CalendarSettings.tsx).
- Low-risk items: 7 (App.tsx leftover; FormsSettings.tsx / ContactFormsTab.tsx / WorkflowTabContent.tsx dead files; AutomationsContent.tsx triggers; AIAssistantDialog.tsx quickMessages; PipelineContent.tsx formatters).
- Already resolved since previous backlog:
  - `CalendarContent.tsx` — migrated to real `useContacts`/`useTattooers` in an earlier session (backlog `mock-to-real-backlog.md` still lists it as pending — stale).
  - Forms tab removed from `SettingsContent.tsx` (Antenor PR #1), leaving FormsSettings.tsx / ContactFormsTab.tsx as orphaned dead files.
  - Workflows tab removed from base Automations UI, leaving WorkflowTabContent.tsx orphaned.
  - App.tsx invite flow migrated off useMockData (Claude) — App.tsx now only reads isLoading/error.
- New issues discovered:
  - 3 orphaned dead files still importing useMockData (FormsSettings.tsx, ContactFormsTab.tsx, WorkflowTabContent.tsx) — safe deletions.
  - `businessHours` in CalendarSettings.tsx has no backing table in any migration — needs schema before it can be made real.
- Next recommended task for Claude: plan (only) the Team Members / Permissions real membership model — highest-value medium-risk domain; plan-first then pause for Victor approval.
- Next recommended task for Antenor: delete the 3 orphaned dead files (FormsSettings.tsx, ContactFormsTab.tsx, WorkflowTabContent.tsx) + unused translation keys — pure dead-code cleanup, fits the Antenor fast-lane rule.

## Final status

```text
Task: useMockData inventory
Agent: Claude
Status: completed
Repo inspected: rtzforyou/easytattoo-crm (origin/main = 4ca84a0)
Engine branch: claude/usemockdata-inventory-report
Report path: agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
Commit SHA: (this report commit — see PR/branch head)
Files inspected: App.tsx; components/settings/FormsSettings.tsx; components/contacts/ContactFormsTab.tsx; components/automations/WorkflowTabContent.tsx; components/team/TeamContent.tsx; components/payments/PaymentsContent.tsx; components/calendar/CalendarSettings.tsx; components/automations/AutomationsContent.tsx; components/messages/AIAssistantDialog.tsx; components/pipeline/PipelineContent.tsx; hooks/useMockData.ts
Verification: grep -rln "useMockData" across hooks/ components/ App.tsx lib/; per-file line inspection; orphan checks (grep for imports of FormsSettings/ContactFormsTab/WorkflowTabContent — all confirmed orphaned); grep for business_hours in supabase/migrations (confirmed absent). No product code changed, no commit to product, no migration, no Supabase change.
Risks / not verified: automationTriggers ownership depends on a ChatGPT/Victor approach decision (static registry vs DB-backed); before deleting the 3 orphaned files a full tsc build should confirm no type-only references remain (not run — inventory-only).
Permission requested from Victor: yes
```

## Process note (branch discipline)

I did NOT touch Antenor's branch or the product repo. I noticed the shared local checkout is currently sitting on Antenor's branch `antenor/remove-forms-module-strings` — I left it exactly as-is and performed this entire report through the GitHub API on the keyros-engine branch `claude/usemockdata-inventory-report`, with no product-side git operations.

Signed,
Claude — Heavy Implementation Agent
