# Changelog

## Unreleased

Add a deploy command, an infrastructure-as-code skill, and a web-app scaffold type.

### Added

- `/forge:deploy`: deploys a project using its own infrastructure. Detects an
  `infra/` OpenTofu directory, a Dockerfile, and the app's release steps, then
  applies them with a plan-before-apply discipline.
- `infrastructure-as-code` skill: the shared source of truth for OpenTofu. Covers
  the `infra/` layout, modules by role, provider abstraction, remote state,
  plan-before-apply, and secrets handling.
- `web-app` scaffold type: a server-rendered web application that follows the web
  framework's own conventions instead of the four-layer split.
- Optional `cache` and `search` infrastructure modules, opt-in and detection-driven,
  with Redis and Meilisearch as their default implementations.

### Changed

- `/forge:scaffold` now references the `infrastructure-as-code` skill for all
  OpenTofu rules instead of restating them inline.

## v0.3.0

Add test harness command for MCP-based automated QA.

### Added

- `/forge:test-harness`: generates an MCP server that exposes the
  project's domain and application layers as tools Claude can call
  for headless testing and QA.

## v0.2.1

Add clean-code skill and update documentation to reflect current plugin state.

### Added

- `clean-code` skill: code-level quality rules covering magic values,
  cognitive complexity, guard clauses, naming, single responsibility,
  locality, tempered DRY, parameter count, error handling, and comments.

### Changed

- All commands now reference the `clean-code` skill where relevant.
- README and CLAUDE.md updated to reflect the full set of commands and skills.

## v0.2.0

Add token budget management for Max 5x plan.

### Added

- `token-budget` skill: session management rules for working within Claude
  Code rate limits. Covers batch sizing, commit frequency, graceful stopping,
  resume paths, and model selection.

### Changed

- All commands now reference the `token-budget` skill for session management.
- Model selection defaults to Opus for all coding and writing tasks. Sonnet
  is reserved for lightweight, mechanical operations only.

## v0.1.2

Fix the version command missing plugin.json on every release.

### Fixed

- `.claude-plugin/plugin.json`: was stuck at `0.1.0` after the v0.1.1
  release. `/forge:version` now updates it explicitly every release,
  instead of relying on the generic manifest list.

### Added

- README: a new "Upgrading" section documenting `claude plugin update`
  and `claude plugin list`.

## v0.1.1

Add the version command and release tracking files.

### Added

- `/forge:version`: bump the version, update VERSION.md and CHANGELOG.md,
  create an annotated tag, and push it.
- `VERSION.md`: single source of truth for the plugin's current version.
- `CHANGELOG.md`: release history for the plugin.

## v0.1.0

Initial release of forge.

### Commands

- `/forge:do-work`: execute a development task from understanding through
  implementation and testing. Stops before pushing.
- `/forge:review`: audit the current branch diff and return a SHIP IT, NEEDS
  WORK, or RETHINK verdict.
- `/forge:ship`: run pre-flight checks, push the branch, and open a draft PR
  through the gh CLI.
- `/forge:scaffold`: generate a complete Clean Architecture project in any
  language with optional OpenTofu infrastructure.
- `/forge:write`: write and revise long-form prose in any genre or subject.
- `/forge:publish`: compile a manuscript into EPUB for Amazon KDP and Barnes
  and Noble Nook.

### Skills

- `clean-architecture`: layering rules, dependency direction, naming, and
  per-layer testing strategy.
- `conventional-commits`: commit subject format, allowed types, body and
  footer rules.
- `writing-style`: sentence length, punctuation, vocabulary consistency,
  and paragraph form rules.
