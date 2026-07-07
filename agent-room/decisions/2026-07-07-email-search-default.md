# Decision — Default Email Search Target

## Decision

When Victor asks ChatGPT to check emails related to agent communication, Claude, Antenor, Keyros tasks, reports, or formal instructions, the default target mailbox/filter is:

```text
rtzlatattoo@gmail.com
```

## Rule

If Victor says any of the following:

```text
ve o email
ve os emails que foram enviados para mim
procura o email do Claude
procura o relatório no email
ve se chegou o email
```

ChatGPT should interpret it as:

```text
Search emails sent to or involving rtzlatattoo@gmail.com, unless Victor specifies another address.
```

## Purpose

Keep agent communication centralized and avoid asking Victor to repeat the same email address.

## Important distinction

GitHub remains the official source of truth for tasks, decisions and reports.

Email is used as notification/copy channel and may be searched when Victor asks for email-based context.

## Search behavior

Prefer precise searches such as:

```text
to:rtzlatattoo@gmail.com Claude
```

```text
to:rtzlatattoo@gmail.com Keyros
```

```text
to:rtzlatattoo@gmail.com "Agent Room"
```

```text
to:rtzlatattoo@gmail.com "Commit SHA"
```

If the exact search is not enough, broaden only after checking the most likely query first.
