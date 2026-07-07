# ADR-0001 — Remove Forms from Core and Defer Workflows

## Status

Accepted

## Context

Keyros must stay focused on the core business workflow: CRM, pipeline, WhatsApp, automations, calendar, finance, dashboard and team management.

Two current areas increase complexity without enough core value for the first commercial version:

1. Forms / consent form builder.
2. Workflows / advanced multi-step automation builder.

Forms can be replaced by Google Forms or external form tools for now.

Workflows are too complex for the standard user and risk making the automation area harder to understand.

## Decision

Remove Forms from the core app.

Remove Workflows from the base app.

Keep simple automations as the standard automation model.

Do not implement a real workflow database model now.

## Future option

Workflows may return later as a premium or agency-only feature.

Target users for future workflows:

- marketing agencies;
- advanced operators;
- users who need multi-step campaigns;
- users who understand automation branching and execution logs.

If revived, workflows must be implemented as a separate premium module with clear plan gating and execution logs.

## Consequences

Positive:

- Less product complexity.
- Faster path to a stable commercial version.
- Less mock code to convert.
- Lower support burden for standard users.
- Stronger focus on CRM, WhatsApp, finance and dashboard.

Negative:

- No native form builder in the base app.
- No advanced multi-step automations in the base app.
- Some future agency users may ask for workflows.

## Implementation notes

- Remove Forms tab from Settings.
- Remove Workflows tab from Automations for base users.
- Remove Forms and Workflows from the core graph.
- Keep external form URL support as a possible lightweight future integration.
- Do not create form/workflow database migrations now unless the scope changes.
