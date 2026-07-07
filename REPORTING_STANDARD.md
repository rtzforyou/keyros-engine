# Agent Reporting Standard

Every agent must clearly report what was done, where it was done, and how it was verified.

This applies to Claude, Antenor, ChatGPT, Codex, Gemini or any future execution agent.

## Required final report

At the end of every task, the agent must provide:

```text
Task:
Agent:
Repository:
Branch:
Commit SHA:
Pull Request:
Status:
```

## What changed

List every changed file with a short explanation.

```text
Files changed:
- path/to/file.ext
  - What changed:
  - Why it changed:
```

## Where it changed

Be specific.

```text
Location:
- Repository:
- Branch:
- File:
- Function/component/table/view/migration:
```

## Verification

The agent must say how the change was verified.

```text
Verification:
- Build:
- Tests:
- Manual check:
- Database check:
- Security/RLS check:
```

If something was not verified, the agent must say so explicitly.

## Remote-only changes

If the agent applies something directly in Supabase or any remote service, it must say:

```text
Remote-only change: yes/no
Service:
Project/environment:
Migration committed to GitHub: yes/no
Reason if not committed:
```

Remote-only database changes are not considered complete until a migration is committed to GitHub or the missing migration is explicitly documented.

## Required links/references

When available, provide:

- commit SHA;
- branch name;
- PR number/link;
- migration filename;
- Supabase function name;
- test output summary.

## Forbidden final reports

Do not say only:

```text
Done.
Fixed.
Applied.
Everything works.
```

A task is not complete without location, files, commit/reference and verification status.

## Final report template

```text
Task:
Agent:
Repo:
Branch:
Commit SHA:
PR:

Files changed:
1.
2.
3.

What changed:

Why changed:

Verification performed:
- Build:
- Tests:
- Manual:
- Database/RLS:

Remote changes:
- Supabase:
- Edge Function:
- Other:

Keyros Engine updated:
- yes/no
- files:

Risks / not verified:

Next isolated task recommended:
```
