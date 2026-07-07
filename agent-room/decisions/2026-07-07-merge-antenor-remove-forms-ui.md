# Merge Decision — Antenor Remove Forms UI

Date: 2026-07-07

## Decision

Victor approved the merge of Antenor's Forms UI removal task.

## Product repository

```text
rtzforyou/easytattoo-crm
```

## Pull request

```text
PR #1 — feat(settings): remove Forms tab from core Settings UI
```

## Source branch

```text
antenor/remove-forms-ui
```

## Base branch

```text
main
```

## Final branch commit before merge

```text
21b89a54e38eb166568d82180132249ef608481c
```

## Merge commit

```text
4ca84a0aa55621a857b75a4ac5ff685bd3709281
```

## Merge method

```text
squash
```

## Scope merged

Removed Forms from the core Settings UI.

Confirmed on product `main` after merge:

```text
FormsSettings: removed from SettingsContent.tsx
FileCsv: removed from SettingsContent.tsx
TabsTrigger value="forms": removed
TabsContent value="forms": removed
```

## Final Settings tabs on main

```text
profile
studio
preferences
integrations
landing-pages
```

## Risk

Low.

Reason:

- one UI file changed;
- no Supabase migration;
- no database/RLS changes;
- no Edge Function changes;
- no auth/team/payment/calendar/automation logic changed.

## Status

```text
completed
```

## Next recommended isolated task

Clean remaining Forms references outside the Settings UI, especially module/permission lists such as:

```text
App.tsx
components/auth/Login.tsx
```

This should be a separate task and should not be mixed with other UI or permission changes.

Signed,
ChatGPT — Orchestrator / Critical Planner for Keyros
