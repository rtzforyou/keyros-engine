# Current Task — Claude

To: Claude
Copy/notification: rtzlatattoo@gmail.com
From: ChatGPT — Orchestrator / Critical Planner
Subject: Production verification gate — confirm merged work exists and deploy/test real system
Date: 2026-07-07

## Why this task exists

Victor is concerned that tasks are completing extremely fast and wants proof that work is actually present in `rtzforyou/easytattoo-crm` and verified in the running system, not only reported in chat.

ChatGPT confirmed PR #2–#8 are merged in `rtzforyou/easytattoo-crm`. Now Claude must verify the product repository and deployment status from the actual repo state.

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Engine repository

```text
rtzforyou/keyros-engine
```

## Current authorized task

Task V0 — Production verification/deploy audit for merged fast-lane work.

This task replaces the previously authorized Task 0 until completed.

## Scope

Verify the merged work from PR #2–#8 is really present on product `main`, then verify/deploy the frontend so Victor can confirm in the real system.

Merged PRs to verify:

```text
PR #2 — chore: remove orphaned Forms and Workflow UI files
PR #3 — chore: remove Forms from static module lists
PR #4 — refactor(pipeline): extract formatters from mock data hook
PR #5 — chore(messages): remove inactive mock data import from AI assistant
PR #6 — refactor(automations): remove trigger labels from mock data hook
PR #7 — feat(calendar): use real tattooers hook in Calendar settings
PR #8 — chore(forms): remove orphaned FormFiller component
```

PR #9 Team migration is open and must not be merged/applied in this task.

## Required verification against product main

Checkout/update product `main` and verify:

```text
1. removed files are absent:
   - components/settings/FormsSettings.tsx
   - components/contacts/ContactFormsTab.tsx
   - components/automations/WorkflowTabContent.tsx
   - components/automations/WorkflowDialog.tsx
   - components/automations/WorkflowItemCard.tsx
   - components/forms/FormFiller.tsx

2. forms is absent from static module lists:
   - App.tsx
   - components/auth/Login.tsx

3. PipelineContent no longer imports useMockData only for formatters and uses lib/formatters.ts

4. AIAssistantDialog no longer imports inactive useMockData

5. AutomationsContent no longer imports useMockData for trigger labels and does not modify execution logic

6. CalendarSettings uses hooks/useTattooers.ts for tattooers but still leaves businessHours untouched on mock until Claude/schema work
```

## Required build/deploy verification

Run:

```text
npm install or npm ci, according to the repo lockfile
npm run build
```

Then identify the real deployment mechanism from the repo and execute the correct deploy command if available and safe.

Do not guess. Inspect package scripts/config first.

Examples of acceptable deploy evidence:

```text
Cloudflare Pages deploy command output
Vercel deploy output
GitHub Actions deployment run URL
Netlify deploy output
```

If deploy cannot be executed from Claude's environment, report exactly why and provide the exact command Victor must run.

## Required runtime smoke verification

After deploy, or after build if deploy cannot be run, verify as much as possible:

```text
app loads without white screen
Settings no longer shows Forms tab
Automations still renders without Workflows tab
Calendar Settings renders tattooers section
Messages/AI Assistant opens without crash
Pipeline renders without formatter errors
```

If browser access is unavailable, say so explicitly and do not claim UI verification.

## Forbidden

Do not merge PR #9.

Do not apply Supabase migrations.

Do not touch Supabase production.

Do not change code unless the verification finds a build-breaking issue. If code change is required, stop and report `blocked/needs_fix` before modifying.

Do not continue to Team tasks.

Do not touch Antenor branches.

## Required report

Create report in keyros-engine:

```text
agent-room/reports/2026-07-07-claude-v0-production-verification-deploy-audit.md
```

Use engine branch:

```text
claude/production-verification-deploy-audit
```

Report must include:

```text
Task: V0 — Production verification/deploy audit
Agent: Claude
Status: completed / blocked / needs_fix
Product repo:
Product branch verified:
Product commit verified:
PRs verified:
Files confirmed absent:
Files confirmed changed:
Build command:
Build result:
Deploy mechanism detected:
Deploy command executed:
Deploy result / deploy URL:
Runtime smoke verification:
What could not be verified:
Remote changes:
Supabase touched: no
PR #9 touched: no
Risks:
Next recommended task:
Permission requested from Victor: yes
```

## Final response format

Do not paste the full report into chat.

Return only:

```text
Task:
Status:
Product branch:
Product commit:
Deploy result:
Deploy URL:
Report path:
Engine commit:
Permission requested:
```

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
