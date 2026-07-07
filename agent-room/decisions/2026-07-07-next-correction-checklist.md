# Next Correction Checklist — MockData / Core Cleanup

## Status at creation

Date: 2026-07-07

Victor reported:

- Antenor finished his current task.
- Claude is still running his current task.

Important: Antenor's result still needs verification through a formal report, branch/commit/PR, and changed files.

## Current operating rule

One task at a time per agent.

No agent continues automatically.

Every completed task must include:

```text
Repo:
Branch:
Commit SHA:
Files changed:
Verification:
Risks / not verified:
Permission requested from Victor:
```

## Checklist order

### 0. Verify Antenor's completed task

Owner: ChatGPT / Victor

Goal:

Confirm exactly what Antenor changed.

Required evidence:

```text
Repo:
Branch:
Commit SHA or PR:
Files changed:
Build/test result:
Confirmation that only allowed files were touched:
```

Expected task:

- Remove Forms from core Settings UI.

Do not mark this as closed until the evidence is available.

Status: pending verification.

---

### 1. Wait for Claude's useMockData inventory

Owner: Claude

Goal:

Produce complete inventory of current `useMockData` imports/usages.

Expected output:

```text
File:
What it uses from useMockData:
Domain:
Risk:
Recommended action:
Suggested owner:
```

No code changes expected.

Status: in progress.

---

### 2. Compare Claude inventory with existing backlog

Owner: ChatGPT

Goal:

Compare Claude's fresh inventory with:

```text
audits/mock-to-real-backlog.md
audits/priority-2-code-review.md
```

Output:

- confirmed remaining mock areas;
- removed/obsolete mock areas;
- new mock areas discovered;
- recommended next isolated task.

Status: waiting for Claude inventory.

---

### 3. Decide next Antenor task

Owner: Victor + ChatGPT

Antenor should only receive small isolated tasks.

Candidate tasks after verification:

1. Clean leftover Forms imports/references if Antenor did not fully finish.
2. Remove visible Workflows leftovers only if any remain.
3. Small UI cleanup from Claude inventory.
4. Remove dead labels/tabs that do not touch logic.

Forbidden for Antenor:

- RLS/security;
- database migrations;
- payments architecture;
- team permissions model;
- automation execution engine;
- calendar sync;
- broad refactors.

Status: waiting for Antenor report and Claude inventory.

---

### 4. Decide next Claude task

Owner: Victor + ChatGPT

Claude should receive heavy/critical tasks.

Candidate tasks after inventory:

1. Team Members / Permissions real plan.
2. Client Payments manual/Supabase-first plan.
3. Calendar cleanup plan: replace mock contacts/tattooers.
4. Automations trigger registry plan.

Rule:

Claude should plan first, then pause for Victor approval before implementation.

Status: waiting for Claude inventory.

---

### 5. Update inbox/current commands

Owner: ChatGPT

Goal:

Keep one current file per agent:

```text
agent-room/inbox/claude-current.md
agent-room/inbox/antenor-current.md
```

Victor should only need one generic paste command:

```text
Leia o repositório `rtzforyou/keyros-engine`.
Depois vá para `agent-room/inbox/`.
Abra o arquivo correspondente ao seu agente:
- Claude: `claude-current.md`
- Antenor: `antenor-current.md`
Siga exatamente o comando do seu arquivo e pare para pedir permissão ao Victor antes de continuar.
```

Status: active operating model.

## Immediate next actions

1. Get Antenor's formal final report.
2. Verify Antenor's commit/PR/files.
3. Wait for Claude inventory.
4. Compare inventory against backlog.
5. Choose exactly one next task for each agent.

## Stop condition

No new implementation should start until:

- Antenor's completed task is verified;
- Claude's inventory is received;
- Victor approves the next isolated task.
