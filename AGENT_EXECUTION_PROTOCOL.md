# Agent Execution Protocol

Use this protocol when an agent such as Claude, ChatGPT, Codex or Gemini works on the Keyros product repository.

## Repositories

Knowledge engine:

- `rtzforyou/keyros-engine`

Product app:

- `rtzforyou/easytattoo-crm`

## Core rule

Agents must read the knowledge engine first and change the product repository one isolated task at a time.

Do not allow broad refactors without an approved plan.

## Required sequence

1. Read `keyros-engine/GRAPH.md`.
2. Read `keyros-engine/CONTEXT_INDEX.md`.
3. Read the specific domain file affected by the task.
4. Read any relevant audit or ADR.
5. Produce a short implementation plan.
6. Wait for approval when the change is structural.
7. Apply one focused change in `easytattoo-crm`.
8. Run build/tests/lint if available.
9. Update `keyros-engine` only if the change affects graph, rules, architecture or domain behavior.
10. Open a small PR or commit with a clear message.

## One-change rule

One task should affect one clear product concern.

Good examples:

- Remove Workflows tab from Automations.
- Replace Team members mock state with real Supabase membership hook.
- Remove Forms tab from Settings.
- Replace automation trigger list from `useMockData` with a real trigger registry.

Bad examples:

- Refactor all CRM state.
- Fix all mock data.
- Rebuild finance and team and automations in one pass.
- Clean the entire app architecture.

## Before coding checklist

The agent must answer:

1. What is the exact problem?
2. Which domain is affected?
3. Which files will be touched?
4. Does this change database/schema/RLS?
5. Does this change the graph?
6. Does this require an ADR?
7. What is the rollback path?

## After coding checklist

The agent must report:

1. Files changed.
2. Behavior changed.
3. Tests/build result.
4. Risks or unknowns.
5. Whether `keyros-engine` was updated.
6. Next recommended isolated task.

## Rule against bloat

If the agent cannot explain the change in one paragraph, the task is probably too large and must be split.
