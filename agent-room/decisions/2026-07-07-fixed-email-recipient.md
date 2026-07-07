# Decision — Fixed Email Recipient for Agent Messages

## Decision

Every formal email-style command created for Claude, Antenor or future Keyros agents must use the same recipient.

```text
rtzlatattoo@gmail.com
```

## Rule

When ChatGPT drafts an email-style command, the writing block must include:

```text
recipient="rtzlatattoo@gmail.com"
```

## Purpose

Keep all human-facing task notifications centralized and easy to find.

## Important distinction

GitHub remains the official source of truth for agent tasks, reports and decisions.

Email is only the fixed notification/copy channel.
