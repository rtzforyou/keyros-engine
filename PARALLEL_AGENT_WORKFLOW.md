# Parallel Agent Workflow

This protocol allows Claude and Antenor to work at the same time without interfering with each other's code.

## Main rule

Two agents cannot edit the same file or the same domain at the same time.

Parallel work is allowed only when ownership is clearly separated.

## Branch naming

Claude branches:

```text
claude/<domain>-<task>
```

Examples:

```text
claude/team-real-membership
claude/payments-client-model
claude/automation-trigger-registry
```

Antenor branches:

```text
antenor/<domain>-<task>
```

Examples:

```text
antenor/remove-forms-tab
antenor/cleanup-workflows-ui
antenor/fix-button-labels
```

## Task ownership model

Each task contract must define:

```text
Owner agent:
Repo:
Branch:
Domain:
Allowed files:
Forbidden files:
Database changes: yes/no
RLS changes: yes/no
Graph update needed: yes/no
Rollback path:
```

## File lock rule

Before an agent starts, ChatGPT must define file ownership.

Example:

```text
Claude owns:
- hooks/useTeamMembers.ts
- hooks/usePermissions.ts
- supabase/migrations/*team*
- components/team/TeamContent.tsx only if required

Antenor owns:
- components/settings/SettingsContent.tsx
- components/settings/FormsSettings.tsx removal references
```

If a file is owned by Claude, Antenor cannot edit it in the same cycle.

If a file is owned by Antenor, Claude cannot edit it in the same cycle.

## Safe parallel combinations

Allowed:

```text
Claude: Team members real model
Antenor: Remove Forms tab from Settings
```

Allowed:

```text
Claude: Payment database architecture
Antenor: Remove Workflows UI references
```

Allowed:

```text
Claude: Automation execution backend
Antenor: Copy/UI cleanup in Dashboard
```

Not allowed:

```text
Claude: Automations refactor
Antenor: Workflows removal inside AutomationsContent.tsx
```

Reason: same domain and likely same files.

Not allowed:

```text
Claude: Team permissions
Antenor: Team UI edit
```

Reason: overlapping `components/team` ownership.

## Merge order

1. Small UI cleanup can merge first if isolated.
2. Heavy architecture work merges after build/test review.
3. If both touch product navigation, merge one first and rebase the other.
4. Never merge two branches blindly.

## Review checklist before merge

- Did the agent edit only allowed files?
- Did build pass?
- Did any import break?
- Did it create new `useMockData` dependency?
- Did it change graph/domain behavior?
- Does `keyros-engine` need update?
- Is rollback obvious?

## Claude usage rule

Use Claude for expensive/heavy tasks only.

Claude tasks should be high-impact:

- team membership real model;
- payments architecture;
- RLS/security;
- Supabase migrations;
- automation execution engine;
- critical integration bugs.

## Antenor usage rule

Use Antenor for small bounded tasks.

Antenor tasks should be simple:

- remove a tab;
- remove an import;
- clean UI;
- change labels;
- small component extraction;
- simple mapping changes.

## ChatGPT orchestration rule

ChatGPT must create the task contract before either executor starts.

ChatGPT must not assign tasks with vague scope.

Bad task:

```text
Fix automations.
```

Good task:

```text
Remove Workflows tab from AutomationsContent.tsx and ensure no WorkflowTabContent import remains in the base automation UI. Do not touch automation backend or database.
```

## Production rhythm

Recommended daily flow:

1. Victor decides priority.
2. ChatGPT creates two possible tasks: one Claude task and one Antenor task.
3. Victor approves.
4. Agents work in separate branches.
5. Results are reviewed one at a time.
6. Merge small isolated task first.
7. Merge heavy task after tests.
8. Update `keyros-engine` if needed.
