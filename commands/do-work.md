---
description: Execute a development task end to end, understand, plan, implement, and test. Stops before pushing or opening a PR.
argument-hint: "[ticket, feature request, bug description, or task]"
---

# /forge:do-work

You are executing a development task from start to finish, but you stop short of pushing anything. Pushing and PR creation belong to `/forge:ship`. Never run `git push` or open a PR from this command, even if it seems convenient.

The task is: $ARGUMENTS

If `$ARGUMENTS` is empty, ask the user what task to work on before doing anything else.

## Workflow

### 1. Understand the task

Restate the task in your own words. If it is a Jira ticket ID or key with no other detail, ask the user for the ticket description. Do not guess or fabricate acceptance criteria. If the task is vague, for example "fix the login bug," ask one or two clarifying questions. Only do this if you genuinely cannot proceed without more detail. Otherwise, make a reasonable assumption and state it clearly in your plan.

### 2. Analyze the codebase

Before writing any plan, orient yourself in this repo. Look at the following.

- Look for `CLAUDE.md`, `AGENTS.md`, or `README.md` at the root and in relevant subdirectories. These hold project-specific conventions.
- Identify the language, framework, and package manager from manifest files, such as `package.json`, `go.mod`, `pyproject.toml`, `*.csproj`, or `Cargo.toml`. Do not assume any of these in advance.
- Find existing code that resembles what you are about to build or fix. Follow its patterns for naming, layering, error handling, and test style. Do not introduce your own conventions.
- Identify how tests are run and how linting or formatting is invoked. Check the scripts in the manifest, a `Makefile`, or the CI config, so you can run them later.

### 3. Plan

Present a plan of three to five bullets, maximum. Each bullet should be a concrete step or decision, not a restatement of the task. Show this to the user before writing any code. Keep it tight. This is a quick sanity check, not a design document.

Do not move to implementation until you have shown the plan. If the user is actively chatting and clearly wants you to keep going without a pause, you may continue. Even then, the plan must still appear in your output first.

### 4. Create a branch

Branch naming follows `{type}/{short-kebab-description}`. `type` is one of `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, or `perf`, matched to the nature of the task. The description should be two to five words, lowercase, and hyphenated, drawn from the task itself. Avoid generic names. For example, use `fix/null-session-token-on-refresh`, not `fix/bug`.

Check the current branch first. If there are uncommitted changes that are not yours to discard, stop. Ask the user how to proceed instead of branching over their work. Otherwise, create and check out the new branch from the up-to-date base branch.

### 5. Implement

Write the real implementation. Do not leave placeholder stubs or `TODO` comments standing in for logic that was asked for. Follow the codebase's existing patterns and its dependency conventions. If the project follows Clean Architecture layering, respect it. See the `clean-architecture` skill in this plugin for the underlying rules if you need a refresher.

Commit as you go. Follow the `conventional-commits` skill in this plugin for the subject, body, and footer format. Keep commits small and logically scoped, instead of one giant commit at the end.

### 6. Test and lint

Run the project's existing test suite and linter using whatever tooling the codebase already has. Do not introduce a new test framework or linter. If tests fail, fix the underlying issue, or your code, instead of weakening or deleting the test. If the test itself is provably wrong, explain why before you change it. If no test tooling exists in the repo, say so plainly instead of inventing a fake passing result.

### 7. Report

Finish with a concise report. Follow the `writing-style` skill in this plugin for the report's prose. Cover the following.

- **Summary**: what changed and why, in a few sentences.
- **Files changed**: a short list, grouped by area if it is more than a handful.
- **Assumptions**: anything you inferred rather than confirmed, such as scope decisions, edge cases, or library choices.
- **Suggested PR title**: one line, following the `conventional-commits` skill.
- **Suggested PR body**: a short `What`/`Why`/`How`/`Testing` outline the user can hand to `/forge:ship`.

Then stop. Do not push, do not open a PR, and do not run `gh pr create`. If the user wants that, point them to `/forge:ship`.
