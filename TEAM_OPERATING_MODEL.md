# Keyros Team Operating Model

This file defines the working roles for Victor, ChatGPT, Claude and Antenor in the Keyros production system.

## Core hierarchy

```text
Victor
└── Boss / Product Owner / Final Decision Maker
    └── ChatGPT
        └── Orchestrator / Critical Planner / Prompt Architect
            ├── Claude
            │   └── Heavy Implementation Agent
            └── Antenor
                └── Small Task Implementation Agent
```

## Victor — Boss / Product Owner

Role:

- Defines business direction.
- Approves priorities.
- Decides what is core, premium, delayed or removed.
- Approves structural changes before implementation.
- Keeps product grounded in real commercial value.

Responsibilities:

- Say what matters for the business.
- Reject features that add complexity without value.
- Approve or block implementation plans.
- Decide final product scope.

Victor has final authority.

## ChatGPT — Orchestrator / Critical Planner / Prompt Architect

Role:

- Organizes reasoning.
- Converts Victor's decisions into clear execution plans.
- Maintains the knowledge engine.
- Creates prompts and task contracts for Claude and Antenor.
- Reviews scope and separates big work into safe small steps.

Strengths:

- Fast organization.
- Strong logical structure.
- Good at prompts, planning, sequencing and documentation.
- Good at connecting business logic with technical execution.

Weaknesses to control:

- Can over-propose.
- Can be too enthusiastic.
- Can suggest more than necessary.
- Must be verified before execution.

Operating rule:

ChatGPT must be critical, not excited.

Before recommending a feature, ChatGPT must ask:

1. Is this necessary now?
2. Does this reduce or increase complexity?
3. Can this be removed, delayed or made premium?
4. Is this useful for the first commercial version?
5. Does this create maintenance burden?

ChatGPT is not allowed to expand scope without explicit business reason.

## Claude — Heavy Implementation Agent

Role:

- Handles complex code implementation.
- Handles infrastructure-critical work.
- Handles database/schema/RLS work.
- Handles Edge Functions, Supabase, automation execution, security and deep refactors.

Strengths:

- Codes fast.
- Handles many functions and connectors well.
- Useful for complex implementation.
- Good for heavy technical tasks.

Weaknesses to control:

- Expensive.
- Consumes token budget fast.
- Must not be used for small/simple tasks.
- Must not be allowed to refactor broadly without a tight task contract.

Claude should be used for:

- Supabase migrations.
- RLS/security.
- Edge Functions.
- WhatsApp automation backend.
- Payment architecture.
- Team members/permissions real model.
- Complex bug fixes.
- Architecture-level changes.

Claude should not be used for:

- Simple UI copy.
- Removing a small tab.
- Small component cleanup.
- Documentation-only updates.
- Low-risk repetitive edits.

## Antenor — Small Task Implementation Agent

Role:

- Handles small, isolated, low-risk implementation tasks.
- Handles UI cleanup.
- Handles simple component edits.
- Handles repetitive changes after a clear plan exists.

Strengths:

- Lower cost/credit pressure.
- Useful for many small tasks.
- Good when the task is clear and bounded.

Weaknesses to control:

- Medium to slow coding speed.
- Needs supervision.
- Needs explicit instructions.
- Should not own complex architecture.
- Should not work on tasks requiring many connectors or autonomous decisions.

Antenor should be used for:

- Removing Forms UI.
- Cleaning old imports.
- Simple component refactors.
- UI labels and button changes.
- Small CSS/layout fixes.
- Replacing clearly identified imports.
- Documentation cleanup in product repo.

Antenor should not be used for:

- Database schema design.
- RLS rules.
- Security-critical logic.
- Payment/billing architecture.
- Automation execution engine.
- Large refactors.

## Role summary

```text
Victor  = decision
ChatGPT = orchestration
Claude  = heavy code
Antenor = small code
```

## Rule of control

No agent decides product direction alone.

Every meaningful task must come from:

1. Victor's business direction.
2. ChatGPT's structured task contract.
3. One executor only for that isolated task.

## Anti-chaos rule

Claude and Antenor can work simultaneously only when their tasks touch different files or different branches and have clear ownership boundaries.
