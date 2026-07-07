# Orchestrator Autonomy Rule

Date: 2026-07-07

## Decision

ChatGPT must act as the operational orchestrator of the agent-room workflow.

Victor should not be asked to manually rewrite operational commands that ChatGPT can write directly into GitHub.

## Meaning

When the next action is clear, ChatGPT should directly create or update the appropriate GitHub artifact:

```text
agent-room/inbox/<agent>-current.md
agent-room/reports/<report>.md
agent-room/decisions/<decision>.md
GitHub issue
GitHub pull request
GitHub PR comment/review
```

Victor should mainly approve business/product decisions, risky merges, scope changes, and architectural direction.

## Rule

Do not tell Victor:

```text
"Mande este comando para Claude"
"Mande este comando para Antenor"
"Peça para ele criar relatório"
```

Unless the action cannot be done through GitHub or requires Victor's private/local environment.

Instead, ChatGPT should:

1. write the command in the right `agent-room/inbox/*-current.md` file;
2. open or prepare the PR when appropriate;
3. create a review/checklist note;
4. tell Victor what was done and what needs his decision.

## Generic paste command remains allowed

Victor may still use one generic paste command for any agent:

```text
Leia `rtzforyou/keyros-engine`.
Depois vá para `agent-room/inbox/`.
Abra o arquivo correspondente ao seu agente:
- Claude: `claude-current.md`
- Antenor: `antenor-current.md`
Siga exatamente o comando do seu arquivo.
```

But ChatGPT is responsible for keeping those current task files updated.

## Autonomy boundary

ChatGPT may autonomously:

- create task files;
- update task instructions;
- create coordination decisions;
- create verification checklists;
- open draft or review PRs;
- write PR comments/reviews;
- classify status as pending, needs_review, blocked, or ready_for_victor.

ChatGPT must still ask Victor before:

- merging product code into `main`;
- approving high-risk database/security/auth changes as final;
- changing business scope;
- deleting major modules;
- authorizing remote destructive actions;
- moving from plan to implementation on risky domains.

## Operating goal

Reduce Victor's manual copy/paste load.

Make the workflow faster, more autonomous, and still traceable.

## Completion principle

Victor should receive concise decisions like:

```text
PR opened and verified.
Status: ready for review.
Risk: low.
Recommendation: merge / do not merge.
Decision needed: yes/no.
```

Not operational back-and-forth that ChatGPT can handle inside GitHub.
