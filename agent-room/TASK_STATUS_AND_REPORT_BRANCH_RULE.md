# Task Status and Report Branch Rule

## Decision

Every agent must write their final report and task status update in their own branch.

This prevents Claude, Antenor or any future agent from crashing the simultaneous execution flow by editing the same files on `main` at the same time.

## Why

If multiple agents write reports directly to `main`, they can create:

- push conflicts;
- accidental overwrites;
- unclear task ownership;
- mixed reports;
- broken execution order;
- confusion about which task is complete.

## Branch rule

Each agent must use their own branch for the task.

Claude:

```text
claude/<task-name>
```

Antenor:

```text
antenor/<task-name>
```

ChatGPT:

```text
chatgpt/<task-name>
```

Examples:

```text
claude/usemockdata-inventory
antenor/remove-forms-ui
chatgpt/update-agent-rules
```

## Report location

Each agent must create their final report inside:

```text
agent-room/reports/
```

Report file naming:

```text
agent-room/reports/YYYY-MM-DD-agent-task-report.md
```

Examples:

```text
agent-room/reports/2026-07-07-claude-usemockdata-inventory-report.md
agent-room/reports/2026-07-07-antenor-remove-forms-ui-report.md
```

## Task status update

Each report must include a clear task status:

```text
Status: completed | blocked | needs_review | failed
```

Definitions:

- `completed`: task finished, verified, and report provided.
- `blocked`: task cannot continue without Victor's decision or missing access/context.
- `needs_review`: task is implemented but requires review before merge/use.
- `failed`: task could not be completed and rollback/status is explained.

## Current task file rule

Agents may read their current task from:

```text
agent-room/inbox/claude-current.md
agent-room/inbox/antenor-current.md
```

But they must not overwrite another agent's current task file.

If they need to update their own current task status, they must do it only in their own branch.

## Main branch rule

Do not push report/status updates directly to `main` during simultaneous execution unless Victor explicitly authorizes it.

Preferred flow:

1. Agent creates task branch.
2. Agent executes task.
3. Agent creates report in `agent-room/reports/` on the same branch.
4. Agent commits the report and task changes.
5. Agent opens PR or reports branch + commit SHA to Victor.
6. Victor/ChatGPT reviews.
7. Merge only after approval.

## Required final report fields

Every report must include:

```text
Task:
Agent:
Status:
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
Permission requested from Victor:
```

## Collision prevention rule

Two agents cannot edit the same report file, current task file, product file, migration, or domain file in the same cycle.

If overlap is discovered, the agent must stop and ask Victor before continuing.

## Completion rule

A task is not considered complete until:

1. code or inspection is finished;
2. verification is reported;
3. report exists in `agent-room/reports/` on the agent's branch;
4. branch and commit SHA are provided;
5. Victor has reviewed or approved the next step.
