---
description: Cut a release. Determine the current version, bump it, update VERSION.md and CHANGELOG.md, and create and push an annotated Git tag.
argument-hint: "[major|minor|patch|explicit version, e.g. \"patch\" or \"2.0.0\"]"
---

# /forge:version

You are cutting a release for this project. `/forge:version` determines the current version and bumps it. It updates `VERSION.md` and `CHANGELOG.md`. It creates an annotated Git tag and pushes it. It works for code projects, writing projects, and the forge plugin itself.

It does not open a PR or push a branch. It only handles the tag.

The requested bump level or version is: $ARGUMENTS

Accepted bump levels are `major`, `minor`, and `patch`. An explicit version string looks like `1.2.3` or `v1.2.3`. If `$ARGUMENTS` is empty, ask the user what kind of bump they want.

## Orientation

Determine the current version. Check these sources in this order, and use the first one found.

1. A `VERSION.md` file at the repo root, containing a single version string.
2. Git tags matching a semver pattern, such as `v1.2.3` or `1.2.3`. Use the most recent one by semver ordering, not by date.
3. A version field in a manifest file. Examples include `package.json`, `Cargo.toml`, `pyproject.toml`, `composer.json`, or `.claude-plugin/plugin.json`.
4. If no version is found anywhere, treat the current version as `0.0.0`. Tell the user this is the first tagged release.

## Workflow

1. Report the current version and the new version the requested bump will produce. Ask the user to confirm before proceeding. Do not tag without confirmation.
2. Update `VERSION.md` with the new version string, as a single line.
3. Update version references in manifest files, if they exist. Check `package.json`, `Cargo.toml`, `pyproject.toml`, `composer.json`, and `.claude-plugin/plugin.json`. Do not search and replace the version string across the entire repo. Only update known manifest files and version-specific locations.
4. Update `CHANGELOG.md`. Add a new section at the top, below the `# Changelog` heading. Populate it with commit subjects since the last tag, grouped by commit type from the `conventional-commits` skill. Follow the same format as the existing entries.
5. Commit the version bump, the changelog update, and any manifest changes, together. Follow the `conventional-commits` skill. Use `chore(release): v{version}` as the commit message.
6. Create an annotated Git tag. Use `v{version}` as the tag name. The tag message is `v{version}`, followed by a blank line. After that, add a brief summary of what changed, pulled from the changelog.
7. Push the tag, using `git push origin v{version}`. Also push the commit from step 5.
8. Report the new version, the tag name, and the tag's remote URL. Follow the `writing-style` skill for the report's prose. If this project has CI that triggers on tags, remind the user to check that it ran successfully.

## What this command does not do

- It does not push branches or open PRs. That is `/forge:ship`.
- It does not publish to any platform. Publishing to Gumroad, KDP, npm, or any registry is handled by CI pipelines the tag triggers, or by the user manually.
- It does not decide what version to use without user confirmation. It always asks before tagging.
