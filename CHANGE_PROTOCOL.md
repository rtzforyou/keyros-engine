# Keyros Change Protocol

Use this protocol whenever a new feature, integration, database change, automation, or architectural rule is added to Keyros.

The goal is to keep the graph useful as the product evolves.

## Rule

A feature is not complete until the knowledge engine is updated.

## Required update checklist

For every meaningful change, update only the files that are affected:

1. `GRAPH.md`
   - Add or update the affected node.
   - Add routing if the feature creates a new area of the product.

2. `CONTEXT_INDEX.md`
   - Point agents to the correct file path for this feature.
   - Keep it short.

3. Domain file in `domains/`
   - Add business behavior.
   - Add dependencies.
   - Add known risks.

4. Architecture file in `architecture/`
   - Update only when structure, database, APIs or integration boundaries change.

5. ADR in `adr/`
   - Required only when a decision changes architecture, data model, security, or long-term product behavior.

6. `graph/GRAPH_REPORT.md`
   - Update after Graphify or manual graph refresh.

## Feature intake template

```text
Feature:
Business objective:
Affected nodes:
Affected tables:
Affected components:
Affected edge functions:
Security impact:
Automation impact:
Dashboard/finance impact:
Migration needed:
Graph update needed: yes/no
ADR needed: yes/no
```

## Priority rule

Do not document everything. Document only what changes how the system behaves, connects, stores data, secures data, or makes decisions.

## Anti-bloat rule

If a sentence does not help an agent make a better technical decision, remove it.
