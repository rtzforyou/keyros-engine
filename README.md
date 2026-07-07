# Keyros Engine

Graph-first knowledge engine for the Keyros ecosystem.

This repository is not the product app. It is the fast-reading structural brain used by agents and humans to understand the product without scanning every file first.

## Reading order

Agents must not read the whole repository by default.

Start here:

1. `GRAPH.md`
2. `graph/GRAPH_REPORT.md`
3. `CONTEXT_INDEX.md`
4. Relevant domain files
5. Relevant ADRs

## Source product repository

- Product repo: `rtzforyou/easytattoo-crm`
- Working product name: Keyros
- Legacy/current repo name: EasyTattoo CRM

## Purpose

- Reduce repeated context discovery.
- Prevent contradictory decisions between agents.
- Give Claude, ChatGPT, Codex, Gemini and future tools the same source of truth.
- Keep the project readable as a business system, not only as code.

## Rule

Graph first. Index second. Details only when needed.
