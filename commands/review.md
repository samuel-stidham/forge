---
description: Audit the current branch's diff against its base branch. Return one of three verdicts, SHIP IT, NEEDS WORK, or RETHINK.
argument-hint: "[optional base branch, defaults to main/master]"
---

# /forge:review

You are auditing the changes on the current branch against its base branch. This is a diff review, not a full-repo audit. Scope your attention to what actually changed. Also include enough surrounding context to judge whether it fits.

Follow the `token-budget` skill in this plugin for session management.

## Setup

1. Determine the base branch. Use `$ARGUMENTS` if given. Otherwise, detect the repo's default branch by checking for `main`, then `master`, then whatever `git symbolic-ref refs/remotes/origin/HEAD` reports. Do not assume `main` blindly.
2. Get the full diff between the merge base and the current branch's HEAD, plus the list of changed files. Read enough of each changed file, not just the diff hunks, to understand context where the diff alone is unclear.
3. Look for `CLAUDE.md`, `AGENTS.md`, or `README.md` to learn project-specific conventions before judging the diff against them.

## Checks

Work through each category below. Only raise an issue if it is real and specific. Cite the file and line. Do not pad the review with generic advice that does not apply to this diff.

- **Architecture**: layer violations, the wrong dependency direction, for example domain logic importing infrastructure, and business logic leaking into presentation or controllers. Also watch for missing abstraction where one is clearly needed, and unnecessary abstraction where it is not. If the project follows Clean Architecture, use the `clean-architecture` skill in this plugin as your reference. It defines what counts as a violation and what is a false positive.
- **Code quality**: dead code, unclear naming, swallowed or generic error handling, and duplicated logic that should be shared. Also flag anything that contradicts the codebase's existing conventions. Use the `clean-code` skill in this plugin as your reference for code-level quality. It covers magic values, cognitive complexity, naming, error handling, and the like. Where `clean-architecture` governs the layers, `clean-code` governs what happens inside them.
- **Tests**: check whether new or changed behavior is covered. Tests should assert behavior, such as inputs, outputs, and observable effects. They should not assert implementation details like mock call counts or internal state. Flag obvious missing edge cases, such as empty input, error paths, or boundary values.
- **Security**: hardcoded secrets or credentials, injection risks such as SQL, command, or template injection, and missing input validation at trust boundaries. Also check for unsafe deserialization and gaps in authentication or authorization.
- **Documentation**: check whether the diff changes public behavior, APIs, or setup steps without updating the README or relevant docs and comments.

## Verdict

End with exactly one of these three verdicts. Label it clearly, and do not let anything else compete for the final answer.

- **SHIP IT**: no blocking issues. List minor nits separately, and mark them clearly as optional and non-blocking.
- **NEEDS WORK**: one or more fixable issues exist. List them ordered by severity, most important first. For each one, give the file and line, what is wrong, and what to do about it.
- **RETHINK**: the approach itself has a fundamental problem. This includes the wrong layer, the wrong abstraction, solving the wrong problem, or a structural risk that small fixes will not resolve. Explain the core concern and describe what a better approach would look like. You do not need to write the full alternative implementation.

Do not soften the verdict to be agreeable. If something is broken, say RETHINK or NEEDS WORK, even if the user clearly put a lot of effort in.
