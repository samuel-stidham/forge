---
name: conventional-commits
description: Commit message and PR title format for all commits forge commands create. Every command that commits or writes a PR title must follow this format.
---

# Conventional Commits

This skill defines the commit message format for commits forge commands generate. Every command that creates commits or PR descriptions must follow these rules.

## Subject line

Use this format: `{type}({scope}): {description}`.

The scope is optional. When used, it should be a short word naming the area of the codebase. Examples include `feat(auth): add JWT middleware` and `fix(config): handle missing env vars`.

**Allowed types:**

- `feat`: a new feature or new functionality.
- `fix`: a bug fix.
- `refactor`: a code change that does not add a feature or fix a bug.
- `chore`: a maintenance task, such as a dependency update or a CI change.
- `docs`: a documentation change only.
- `test`: an added or updated test only.
- `style`: a formatting change that does not affect code behavior.
- `perf`: a performance improvement.

**Subject line rules:**

- Use lowercase for the entire subject line.
- Do not end the subject line with a period.
- Keep the subject line to seventy-two characters or less.
- Use the imperative mood. Write "add feature," not "added feature."

## Body

- Separate the body from the subject with a blank line.
- Wrap the body at seventy-two characters.
- Use the body to explain what changed and why. Do not explain how.
- Follow the `writing-style` skill for the body text.

## Footer

- Use `Closes #N` or `Fixes #N` to reference an issue when one applies.
- Use `BREAKING CHANGE:` as a footer prefix when the commit introduces a breaking change.
