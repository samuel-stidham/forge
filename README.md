# forge

forge is a lean, repo-agnostic engineering workflow plugin for [Claude Code](https://claude.com/claude-code). It scaffolds Clean Architecture projects, executes development tasks, reviews changes, and ships pull requests. It works the same way in a large company monorepo or a personal side project.

forge makes no assumptions about language, framework, or tooling. Every command detects what it needs from the repo itself. It looks at manifest files, lockfiles, and existing patterns, plus `CLAUDE.md` and `README.md`. It never requires per-repo configuration.

## Installation

```
/plugin marketplace add samuel-stidham/forge
/plugin install forge@forge-marketplace
```

That's it. There are no config files and no setup step. forge reads what it needs from whatever repo you're in.

## Upgrading

Run this command to upgrade forge to the latest published version.

```
/plugin install forge@forge-marketplace
```

forge pins an explicit version in `.claude-plugin/plugin.json`. Updates only land after a `/forge:version` release bumps that field. Check your installed version at any time with `claude plugin list`.

## Commands

All commands live under the `/forge:` namespace.

### `/forge:do-work`

`/forge:do-work` takes a Jira ticket, a feature request, a bug description, or a plain English task. It understands the task, then analyzes the codebase for existing conventions. Before writing any code, it presents a short plan of three to five bullets. It creates a branch and implements the work using Conventional Commits. It runs the project's own tests and linters. It reports back with a summary, the files touched, any assumptions made, and a suggested PR title and body.

**It stops before pushing.** It never pushes or opens a PR. That's `/forge:ship`'s job.

```
/forge:do-work fix the session refresh bug where tokens go null after 401 retry
/forge:do-work PROJ-1284
/forge:do-work add a rate limiter to the public webhook endpoint
```

### `/forge:review`

`/forge:review` checks the current branch's diff against its base branch. It checks architecture, looking for layer violations and the wrong dependency direction. It checks code quality, looking for dead code, unclear naming, and weak error handling. It checks tests, looking at coverage, edge cases, and whether they test behavior instead of implementation. It checks security, looking for secrets, injection risks, and missing input validation. It checks documentation too.

It ends with exactly one verdict. The verdict is **SHIP IT**, **NEEDS WORK** with issues ordered by severity, or **RETHINK** when the approach has a fundamental problem.

```
/forge:review
/forge:review develop
```

### `/forge:ship`

`/forge:ship` runs pre-flight checks first. It checks that tests pass, that there are no uncommitted changes, and that you are not on `main`, `master`, or `develop`. It pushes the branch with upstream tracking. It opens a **draft PR** through the `gh` CLI. The body uses `What`, `Why`, `How`, and `Testing` sections built from your actual commits. If `gh` is not available, it still pushes the branch. It then prints a formatted PR title and body you can copy and paste.

```
/forge:ship
/forge:ship this also fixes the flaky retry test from last sprint
```

### `/forge:scaffold`

`/forge:scaffold` generates a complete, real, working Clean Architecture project. It is not a stub. Every generated project includes proper domain, application, infrastructure, and presentation layers. It includes a multi-stage, non-root Dockerfile, a `docker-compose.yml`, and a `Makefile`. It includes GitHub Actions CI, a `.env.example`, and a README with an architecture overview. It includes real example tests for each layer, an `.editorconfig`, a `.gitignore`, and an MIT `LICENSE`.

```
/forge:scaffold go rest-api
/forge:scaffold typescript webhook-gateway
/forge:scaffold python background-worker
/forge:scaffold rust cli-tool
/forge:scaffold java task-runner
/forge:scaffold go rest-api --infra opentofu --provider aws
```

**Supported project types:** `rest-api`, `cli-tool`, `background-worker`, `webhook-gateway`, and `task-runner`. Each one is available in any language you ask for, using that language's real idioms and tools.

Add `--infra opentofu` to also generate cloud infrastructure as code. This adds an `infra/` directory with OpenTofu configs for networking, compute, a database when needed, and secrets management. `--provider` is required with `--infra`, and accepts `aws`, `gcp`, or `digitalocean`. The generated HCL files work with both the `opentofu` and `terraform` binaries.

### `/forge:write`

`/forge:write` writes long-form prose for a writing project in this repo. It works for fiction, nonfiction, essays, worldbuilding documents, and similar projects. It adapts to whatever voice, tone, and structure the repo already has. It orients itself by reading existing content and any project guidance files before writing anything.

**It stops before pushing.** It never pushes or opens a PR. Run `/forge:ship` for that.

```
/forge:write chapter 3: the city falls and the survivors regroup at the harbor
/forge:write section on the historical context of Markan priority
/forge:write revise the opening scene to establish the weather before the dialogue
/forge:write add the appendix covering common counterarguments
```

### `/forge:publish`

`/forge:publish` compiles a writing project into publishable ebook formats. It is the prose equivalent of `/forge:ship`. It targets Amazon KDP and Barnes and Noble Nook, and builds EPUB files with the right metadata, formatting, and structure for each. It does not upload anything. Uploading, pricing, and platform forms stay a manual step for the user.

This command needs [Pandoc](https://pandoc.org/installing.html) installed on your machine to generate EPUB files.

```
/forge:publish
/forge:publish kindle
/forge:publish nook
/forge:publish epub
```

### `/forge:version`

`/forge:version` cuts a release. It determines the current version, bumps it, updates `VERSION.md` and `CHANGELOG.md`, and creates an annotated Git tag. It works for code projects, writing projects, and the forge plugin itself. It always asks you to confirm the new version before it tags anything.

It pushes the tag it creates, but it does not push branches or open PRs. CI pipelines can trigger on the tag to automate publishing or deployment.

```
/forge:version patch
/forge:version minor
/forge:version major
/forge:version 2.0.0
```

## Skills

### `clean-architecture`

This is a reference skill. It defines Clean Architecture's layer responsibilities, the inward dependency rule, naming conventions, and a testing strategy for each layer. `/forge:scaffold` uses it to generate projects with correct layers. `/forge:review` uses it to spot real layer violations instead of guessing.

### `conventional-commits`

This is a reference skill. It defines the commit subject format, the allowed commit types, and the body and footer rules. `/forge:do-work` and `/forge:ship` both follow it for every commit and every PR title they create.

### `writing-style`

This is a reference skill. It defines the writing rules for any prose forge generates, such as README content, PR descriptions, and commit bodies. Every command that writes prose follows it, so output stays consistent across the plugin.

### `token-budget`

This is a reference skill. It defines how every forge command manages token usage within Claude Code's rolling rate limits. It covers batch sizing, commit frequency, graceful stopping when limits approach, resume path formatting, and model selection. All commands follow it for session management. Opus is the default for coding and writing tasks. Sonnet is reserved for lightweight, mechanical operations only.

## Commit Conventions

Every commit and PR title forge generates follows the `conventional-commits` skill. Subjects use the format `{type}({scope}): {description}`, with `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`, and `perf` as the allowed types. Subjects stay lowercase, skip the trailing period, and use the imperative mood, for example `feat(auth): add JWT middleware`.

`/forge:do-work` applies this format as it commits. `/forge:ship` applies it to the PR title it generates. You do not need to configure anything for this. It is built into both commands.

## Why this exists

forge exists to run multiple independent side projects off one consistent workflow. It saves you from rebuilding process for each new project. `/forge:do-work`, `/forge:review`, and `/forge:ship` are not tied to any single venture. They are simply how work gets planned, built, and shipped, no matter what the project is.

`/forge:scaffold` is the one command tied to a specific bet right now. It is the engineering backbone behind a small Gumroad business that sells Clean Architecture starter templates. That business spans five project types and several languages each. Still, that is one income stream, not the point of the repo. The real point is having a single, dogfooded workflow. That workflow should travel with me across every project I am trying to turn into income. One example is the templates business. Another example is an ASCII roguelike built for Steam. Each new venture should start from a working process instead of starting from zero.

## Scope

Forge is built for professional and personal development work, not for academic use. Do not use it to complete coursework, assignments, or exams. If you are unsure whether AI-assisted tools are allowed, check your institution's academic integrity policy.

## License

MIT. See [LICENSE](LICENSE) for the full text.

## Inspiration

Forge took inspiration from [Shipyard](https://github.com/mattsears18/shipyard), an autonomous engineering loop plugin for Claude Code. Shipyard is a far more ambitious project. It handles parallel worker pools in isolated git worktrees. It also handles autonomous issue triage, audit pipelines, cost tracking, and PRs that merge automatically through CI.

We built forge because we needed something leaner. It has four commands, no orchestrator, and no concurrency management. It is just a clean workflow that gets out of your way. If our needs grow to require parallel agents, automated backlog triage, or worktree isolation, we will adopt Shipyard. We will not rebuild those features ourselves. Different tools fit different stages.
