# Report — useMockData Inventory (Step 1, refreshed)

Task: Step 1 — Inventory all `useMockData` dependencies (per `agent-room/inbox/claude-current.md`)
Agent: Claude — Heavy Implementation Agent
Status: needs_review
Repo: rtzforyou/easytattoo-crm
Branch: main (inspected at commit `4ca84a0`; report written on keyros-engine branch `claude/usemockdata-inventory`)
Commit SHA: none — inventory-only, no product code changed
PR: none

## Context / freshness note

This inventory was re-run against the **current** product `main` (`4ca84a0`), which now already includes:
- Antenor's Forms-tab removal (PR #1, merge `4ca84a0`);
- my invite-aware signup fix (`f266473`).

So the picture differs from the earlier inventory in two meaningful ways (see items marked ⚠ CHANGED / 🆕 NEW below). This is exactly why re-verifying against live code matters — the backlog docs drift.

## Per-file inventory

### App.tsx  ⚠ CHANGED
- What it uses from useMockData: `isLoading`, `error: dataError` only
- Domain: app bootstrap (loading/error gate)
- Risk: Low
- Recommended action: replace (leftover) — after my invite fix, this is the *only* remaining useMockData usage in App.tsx; the whole 957-line hook is instantiated just to read a loading flag. Should eventually read loading/error from real hooks or a lightweight state.
- Suggested owner: Claude (touches app-level auth/bootstrap wiring) — but low priority.

### components/settings/FormsSettings.tsx  🆕 NOW ORPHANED
- What it uses: `formTemplates`, `addFormTemplate`, `updateFormTemplate`, `archiveFormTemplate`
- Domain: Forms
- Risk: Low
- Recommended action: **remove (dead file)** — confirmed no longer imported anywhere after Antenor removed the Forms tab from SettingsContent.tsx. The file still physically exists and still imports useMockData, but nothing renders it.
- Suggested owner: Antenor (pure dead-code deletion)

### components/contacts/ContactFormsTab.tsx  🆕 NOW ORPHANED
- What it uses: `formSubmissions`, `formTemplates`, `addFormSubmission`, `updateFormSubmission`
- Domain: Forms (inside Contacts)
- Risk: Low
- Recommended action: **remove (dead file)** — confirmed no longer imported anywhere.
- Suggested owner: Antenor

### components/automations/WorkflowTabContent.tsx  🆕 NOW ORPHANED
- What it uses: `workflows`, `addWorkflow`, `updateWorkflow`, `deleteWorkflow`, `toggleWorkflowStatus`
- Domain: Automations / Workflows
- Risk: Low
- Recommended action: **remove (dead file)** — confirmed no longer imported anywhere (Workflows tab was removed from base UI earlier). ADR-0001 already decided Workflows leave the core.
- Suggested owner: Antenor

### components/team/TeamContent.tsx
- What it uses: `teamMembers`, `updateTeamMember`, `removeTeamMember`
- Domain: Team
- Risk: Medium
- Recommended action: replace with real membership model (plan Steps 4–5)
- Suggested owner: Claude

### components/payments/PaymentsContent.tsx
- What it uses: `payments`, `contacts`, `addPayment`, `formatCurrency`
- Domain: Payments (client payments)
- Risk: Medium-high (money)
- Recommended action: replace — requires the client-payments / app-billing split (plan Steps 6–7)
- Suggested owner: Claude

### components/calendar/CalendarSettings.tsx
- What it uses: `tattooers`, `addTattooer`, `removeTattooer`, `businessHours`, `updateBusinessHours`
- Domain: Calendar / Tattooers / Business hours
- Risk: Medium
- Recommended action: replace — split: the `tattooers` part is a simple swap to the already-existing `hooks/useTattooers.ts` (Antenor-sized); the `businessHours` part has **no backing table** (confirmed: no `business_hours` in any migration) and needs a schema decision (Claude).
- Suggested owner: split — Antenor (tattooers swap) + Claude (businessHours schema). Note: distinct file from `CalendarContent.tsx`, which was already migrated to real hooks in an earlier session — the keyros-engine backlog is stale on that point.

### components/automations/AutomationsContent.tsx
- What it uses: `automationTriggers`
- Domain: Automations
- Risk: Low-medium (only the trigger option list; execution is already real via `useAutomations`)
- Recommended action: replace with a real trigger registry (plan Step 9)
- Suggested owner: Antenor if it's just moving a static list to a code registry; Claude if it becomes DB-backed. Victor/ChatGPT to decide the approach first.

### components/messages/AIAssistantDialog.tsx
- What it uses: `quickMessages`
- Domain: Messaging (WhatsApp quick replies)
- Risk: Low
- Recommended action: replace (likely just translated strings)
- Suggested owner: Antenor

### components/pipeline/PipelineContent.tsx
- What it uses: `formatDistanceToNow`, `formatCurrency` (pure utility functions, not mock domain data)
- Domain: none — formatting utilities only
- Risk: Very low
- Recommended action: extract these to a `lib/formatters.ts` instead of pulling them from the hook
- Suggested owner: Antenor

### hooks/useMockData.ts
- What it is: the hook itself (957 lines)
- Domain: cross-cutting
- Risk: n/a in isolation
- Recommended action: keep temporarily — delete only after all domains above are migrated (plan Step 11)
- Suggested owner: n/a

## Summary counts

- 10 product files still import `useMockData` + the hook itself.
- 3 of those are now **dead/orphaned files** (FormsSettings.tsx, ContactFormsTab.tsx, WorkflowTabContent.tsx) — pure deletions, no behavior change, ideal small Antenor tasks.
- 1 changed to a low-risk leftover (App.tsx now only reads isLoading/error).
- 6 remain as genuine mock→real migrations (Team, Payments, Calendar settings, Automations triggers, quickMessages, Pipeline formatters).

## Verification performed
- Build: n/a (no code changed)
- Tests: n/a
- Manual: `grep -rln "useMockData"` across hooks/ components/ App.tsx lib/; per-file line inspection; orphan checks via `grep` for imports of FormsSettings / ContactFormsTab / WorkflowTabContent (all confirmed orphaned); `grep` for `business_hours` in supabase/migrations (confirmed absent). Inventory taken at product `main` = `4ca84a0`.
- Database/RLS: n/a

## Remote changes
None. No commit, no migration, no deploy, no database change in this task.

## Keyros Engine updated
This report only — created on branch `claude/usemockdata-inventory` at `agent-room/reports/`. No product change.

## Risks / not verified
- Suggestion for `automationTriggers` ownership depends on an approach decision (static registry vs DB-backed) that ChatGPT/Victor must make.
- The 3 orphaned files are safe to delete, but a full `tsc` build should confirm no lingering type-only references before deletion (not run here since this is inventory-only).

## Next isolated task recommended
Lowest-risk, highest-clarity next step: delete the 3 now-orphaned dead files (FormsSettings.tsx, ContactFormsTab.tsx, WorkflowTabContent.tsx) + their now-unused translation keys/imports — a clean Antenor task, no logic touched. Separately, the App.tsx / Login.tsx `'forms'` module cleanup (already flagged in `2026-07-07-merge-antenor-remove-forms-ui.md`) is another small isolated Antenor task. Claude-sized heavy work (Team membership real model, Payments split) should be planned only after Victor picks one.

## Permission requested from Victor
Yes — no code change until Victor approves the next isolated task and ChatGPT assigns file ownership (to avoid overlap, given Antenor is active in the Forms/Settings area).

Signed,
Claude — Heavy Implementation Agent
