---
description: Run pre-flight checks, push the current branch, and open a draft PR via the gh CLI with a structured body.
argument-hint: "[optional notes for the PR description]"
---

# /forge:ship

You are taking the current branch's committed work and getting it into a draft PR. This command pushes and opens a PR. Only run it when the user has explicitly invoked `/forge:ship`. Never run it as a side effect of another command.

Additional context for the PR body, if any: $ARGUMENTS

Follow the `token-budget` skill in this plugin for session management.

## Pre-flight checks

Run these before touching the remote. If any fail, stop and report the problem instead of forcing past it.

1. **Not on a protected branch**: refuse to ship directly from `main`, `master`, or `develop`. If the user is on one of these, tell them and stop.
2. **No uncommitted changes**: check `git status`. If there are unstaged or uncommitted changes, stop. Ask whether to commit them first. Do not silently commit on the user's behalf with a generic message.
3. **Branch is up to date**: check whether the base branch, `main`, `master`, or whatever the repo's default is, has moved ahead. Check whether a rebase or merge is needed. Flag this to the user instead of rebasing automatically, since that rewrites history.
4. **Tests pass**: run the project's existing test suite using whatever tooling the repo already has configured. If tests fail, stop and report the failure. Do not push broken code.

## Push

Push the current branch with upstream tracking set, using `git push -u origin <branch>`. Skip this if it is already tracked and up to date with the remote.

## Open the PR

Do the following if the `gh` CLI is available and authenticated.

- Create a **draft** PR targeting the repo's default base branch.
- Build the PR body from the actual commits and diff on this branch. Do not ask the user to re-explain what they already did through commits.
- Structure the body with these sections:
  - **What**: the concrete change, in a sentence or two.
  - **Why**: the motivation behind it, such as a ticket, bug, or request.
  - **How**: the approach taken, especially anything non-obvious.
  - **Testing**: what was run or verified, such as tests or manual checks, and what was not.
- Follow the `writing-style` skill in this plugin for the prose in every section.
- Use the PR title as a single line, following the `conventional-commits` skill in this plugin.
- Report the PR URL back to the user when done.

## Fallback when `gh` is unavailable

If `gh` is not installed or not authenticated, do not try to fabricate a PR through any other means. Instead, still push the branch once pre-flight checks pass. Then print a formatted, copy-paste-ready block containing the following.

- Suggested PR title, following the `conventional-commits` skill
- Suggested PR body, using the same `What`/`Why`/`How`/`Testing` structure and the `writing-style` skill
- The compare URL for the remote, derived from `git remote -v` when possible, so the user can open it in a browser

Tell the user plainly that `gh` was not available, so they know why you fell back.
