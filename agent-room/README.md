# Agent Room

This folder is the official communication room for Keyros agents.

Participants:

- Victor — Boss / Product Owner / final decision maker
- ChatGPT — Orchestrator / critical planner / prompt architect
- Claude — Heavy implementation agent
- Antenor — Small task implementation agent

## Purpose

Keep agent communication clear, traceable and auditable.

This folder is used to store formal task messages, agent reports and coordination notes.

## Rule

Every important instruction must be written as a formal message.

Every execution agent must respond with a clear report.

No vague reports such as:

```text
Done.
Fixed.
Applied.
Everything works.
```

## Folder structure

```text
agent-room/
├── README.md
├── inbox/
│   └── formal task messages to agents
├── reports/
│   └── final reports from agents
└── decisions/
    └── short coordination decisions approved by Victor
```

## Message naming convention

Use date and agent name.

Examples:

```text
agent-room/inbox/2026-07-07-claude-team-members.md
agent-room/reports/2026-07-07-claude-team-members-report.md
agent-room/decisions/2026-07-07-remove-workflows.md
```

## Required task message format

```text
To:
From:
Subject:
Date:
Repo:
Branch:
Domain:
Allowed files:
Forbidden files:
Objective:
Scope:
Out of scope:
Verification required:
Final report required:
Signature:
```

## Required final report format

```text
Task:
Agent:
Repo:
Branch:
Commit SHA:
PR:
Files changed:
What changed:
Why changed:
Verification performed:
Remote changes:
Keyros Engine updated:
Risks / not verified:
Next isolated task recommended:
```

## Operating principle

If it is not written here or in a linked GitHub commit/PR, it is not considered fully traceable.
