---
name: token-budget
description: Session-management rules for long-running forge commands. Work in small committed batches, stop gracefully when rate limits approach, and always leave a resume path.
---

# Token Budget

This skill defines how forge commands manage token usage to avoid hitting Claude Code's rate limits. Every long-running command must follow these rules.

## Context

Claude Code enforces rolling rate limits. The exact budget varies by plan and shifts over time, and the plugin cannot query it. Commands must be conservative and break work into small, completable units. A larger budget does not mean careless usage.

## Rules for all commands

### Work in small batches

Every command should prefer completing one small task fully over starting a large task it might not finish. For `/forge:do-work`, this means picking the smallest meaningful unit of work and finishing it before starting the next piece. For `/forge:write`, this means completing one section or scene fully rather than drafting an entire chapter at once.

### Track your own usage

Keep a rough mental count of how many tool calls, file reads, and file writes you have made in the current session. If you have made more than 30 tool calls in a single command invocation, pause and check in with the user before continuing. Large commands like `/forge:scaffold` may need more, but most tasks should complete well under that number.

### Commit early and often

Never accumulate a large amount of uncommitted work. Commit after each logical step. If the session is interrupted or hits a limit, the user should never lose work. Small, frequent commits mean the worst case is losing one small step, not an entire feature.

### Stop gracefully when approaching limits

If Claude Code warns about approaching rate limits or token usage, do not try to squeeze in one more operation. Stop immediately. Commit any uncommitted work. Write a brief status note explaining what was done and what remains. Format the status note so the user can pass it as the `$ARGUMENTS` to the same command in the next session to resume.

### Provide a resume path

When stopping mid-task, always end with a clear resume instruction. For example:

```
Work paused due to token budget. To resume, run:
/forge:do-work continue implementing the retry logic for webhook delivery.
The branch feat/webhook-retry has the work so far. The HTTP client adapter
is done. Remaining: add the exponential backoff config and the integration
test.
```

This lets the user copy that line into a new session and pick up where the work stopped.
