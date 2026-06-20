# forge

forge is a repo-agnostic engineering workflow plugin for Claude Code. It scaffolds Clean Architecture project templates, executes development tasks, reviews diffs, and ships PRs. It is installed across every machine and repo Samuel works in. That includes Hudl repos and personal side projects alike. It must behave identically everywhere.

## Working on forge itself

This repo *is* the plugin. There is no separate build step. Editing a file under `commands/` or `skills/` changes the plugin's behavior directly.

### Testing changes

Claude Code reads plugin commands and skills from disk at session start, and again on plugin reload. To test a change, follow these steps.

1. If forge is installed locally, through a local path or `claude plugin add`, reload the plugin. This applies when it points at this checked-out repo. Restart the Claude Code session, or use whatever reload mechanism the current version exposes for plugin development. This makes sure the edited `.md` files are picked up.
2. Invoke the changed command, `/forge:do-work`, `/forge:review`, `/forge:ship`, or `/forge:scaffold`, in a real or scratch repo. Check that the behavior matches the instructions in the file.
3. For a skill under `skills/`, test it through the commands that reference it. `clean-architecture` backs `/forge:scaffold` and `/forge:review`. `conventional-commits` backs `/forge:do-work` and `/forge:ship`. `writing-style` backs any command that writes prose. Confirm the skill's guidance is actually applied.

There is no automated test suite for the plugin's own behavior. These are prompt files. Testing means running them against real scenarios and checking that the output makes sense.

### Command naming convention

All commands live under the `/forge:` namespace. That includes `/forge:do-work`, `/forge:review`, `/forge:ship`, and `/forge:scaffold`. The namespace comes from the plugin name, `forge`, set in `.claude-plugin/plugin.json`, plus the filename under `commands/`. Don't rename a command file without updating every reference to it. Check the README, the other command files, and this file.

### Repo-agnostic by design

Every command must work the same whether it runs in a Go monorepo, a TypeScript side project, or a Python script. Concretely, this means the following.

- **Never hardcode a language, framework, package manager, or test runner.** Detect them from the target repo. Look at manifest files, lockfiles, and existing scripts, instead of assuming.
- **Never hardcode a base branch name.** Detect `main`, `master`, or `develop` from the repo. Do not assume `main`.
- **Never assume a CI provider, hosting platform, or cloud vendor.** Only rely on what is already evident in the target repo.
- When a command needs something repo-specific, it should look for that information. Check `CLAUDE.md`, the README, manifest files, and existing code patterns. Do not ask the user to pre-configure forge for that repo. forge ships with zero repo-specific configuration.

If you are adding or editing a command, and you notice an instruction that only makes sense for one ecosystem, stop. Rewrite it to detect and adapt instead.

### Editing the skills

`skills/clean-architecture/SKILL.md`, `skills/conventional-commits/SKILL.md`, and `skills/writing-style/SKILL.md` are shared references for the commands. Each one is the single source of truth for its topic. Don't duplicate architecture rules, commit format rules, or writing rules inside the command files. Reference the matching skill instead.
