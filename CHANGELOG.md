# Changelog

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
