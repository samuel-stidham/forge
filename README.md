# forge

forge is a lean, repo-agnostic engineering workflow plugin for [Claude Code](https://claude.com/claude-code). It scaffolds Clean Architecture projects, executes development tasks, reviews changes, ships pull requests, and deploys with OpenTofu. It also writes long-form prose, compiles ebooks, and cuts releases. It works the same way in a large company monorepo or a personal side project.

forge makes no assumptions about language, framework, or tooling. Every command detects what it needs from the repo itself. It looks at manifest files, lockfiles, and existing patterns, plus `CLAUDE.md` and `README.md`. It never requires per-repo configuration.

## Installation

```text
/plugin marketplace add samuel-stidham/forge
/plugin install forge@forge-marketplace
```

Select "Install for you (user scope)" to make forge available in every repo on that machine.

## Updating

After a new version is pushed to GitHub, refresh the marketplace and update the plugin.

```text
/plugin marketplace update forge-marketplace
```

Then open the `/plugin` menu, go to Installed, find forge, and select update.

If the update does not appear, uninstall and reinstall.

```text
/plugin uninstall forge@forge-marketplace
/plugin marketplace update forge-marketplace
/plugin install forge@forge-marketplace
```

Restart Claude Code after updating for changes to take effect.

## Commands

All commands live under the `/forge:` namespace.

### `/forge:do-work`

`/forge:do-work` takes a Jira ticket, a feature request, a bug description, or a plain English task. It understands the task, then analyzes the codebase for existing conventions. Before writing any code, it presents a short plan of three to five bullets. It creates a branch and implements the work using Conventional Commits. It runs the project's own tests and linters. It reports back with a summary, the files touched, any assumptions made, and a suggested PR title and body.

**It stops before pushing.** It never pushes or opens a PR. That's `/forge:ship`'s job.

```text
/forge:do-work fix the session refresh bug where tokens go null after 401 retry
/forge:do-work PROJ-1284
/forge:do-work add a rate limiter to the public webhook endpoint
```

### `/forge:review`

`/forge:review` checks the current branch's diff against its base branch. It checks architecture, looking for layer violations and the wrong dependency direction. It checks code quality, looking for dead code, unclear naming, and weak error handling. It checks tests, looking at coverage, edge cases, and whether they test behavior instead of implementation. It checks security, looking for secrets, injection risks, and missing input validation. It checks documentation too.

It ends with exactly one verdict. The verdict is **SHIP IT**, **NEEDS WORK** with issues ordered by severity, or **RETHINK** when the approach has a fundamental problem.

```text
/forge:review
/forge:review develop
```

### `/forge:ship`

`/forge:ship` runs pre-flight checks first. It checks that tests pass, that there are no uncommitted changes, and that you are not on `main`, `master`, or `develop`. It pushes the branch with upstream tracking. It opens a **draft PR** through the `gh` CLI. The body uses `What`, `Why`, `How`, and `Testing` sections built from your actual commits. If `gh` is not available, it still pushes the branch. It then prints a formatted PR title and body you can copy and paste.

```text
/forge:ship
/forge:ship this also fixes the flaky retry test from last sprint
```

### `/forge:scaffold`

`/forge:scaffold` generates a complete, real, working Clean Architecture project. It is not a stub. Every generated project includes proper domain, application, infrastructure, and presentation layers. It includes a multi-stage, non-root Dockerfile, a `docker-compose.yml`, and a `Makefile`. It includes GitHub Actions CI, a `.env.example`, and a README with an architecture overview. It includes real example tests for each layer, an `.editorconfig`, a `.gitignore`, and an MIT `LICENSE`.

```text
/forge:scaffold go rest-api
/forge:scaffold typescript webhook-gateway
/forge:scaffold python background-worker
/forge:scaffold rust cli-tool
/forge:scaffold java task-runner
/forge:scaffold php web-app
/forge:scaffold go rest-api --infra opentofu --provider aws
/forge:scaffold php web-app --infra opentofu --provider digitalocean
```

**Supported project types:** `rest-api`, `cli-tool`, `background-worker`, `webhook-gateway`, `task-runner`, and `web-app`. Each one is available in any language you ask for, using that language's real idioms and tools. The `web-app` type follows the web framework's own conventions, such as Laravel MVC or Rails, instead of the four-layer split.

Add `--infra opentofu` to also generate cloud infrastructure as code. This adds an `infra/` directory with OpenTofu configs for networking, compute, a database when needed, and secrets management. It also adds optional `cache` and `search` modules when the app declares them, with Redis and Meilisearch as the defaults. `--provider` is required with `--infra`, and accepts `aws`, `gcp`, or `digitalocean`. The generated HCL files work with both the `opentofu` and `terraform` binaries. Run `/forge:deploy` afterward to apply it.

### `/forge:deploy`

`/forge:deploy` deploys a project using infrastructure that already exists in the repo. It is the verb that follows `/forge:scaffold --infra`. Scaffold writes the infrastructure, and deploy applies it. It detects what is there, an `infra/` OpenTofu directory, a Dockerfile, and the app's release steps. It then deploys without assuming any cloud or pipeline.

It follows the `infrastructure-as-code` skill's plan-before-apply discipline. It applies only after you read and confirm the plan, never destroys infrastructure, and never prints secret values. It builds and pushes the container image when the project ships one. It runs release steps like migrations only when the project defines them, then verifies and reports.

```text
/forge:deploy
/forge:deploy staging
/forge:deploy production
```

### `/forge:test-harness`

`/forge:test-harness` generates an MCP server for headless testing. It reads the project's domain and application layers, finds the public API surface, and exposes it as MCP tools Claude can call. With the harness connected, Claude can launch the application, send inputs, read state, and find bugs without manual QA. It generates lifecycle, action, inspection, and targeted testing tools. It also wires up an MCP entry in `.claude/settings.json`, and writes a `TEST_PLAN.md` checklist to follow.

It produces the harness only. It does not run the QA pass. That is a separate task you start after the harness is connected.

```text
/forge:test-harness
/forge:test-harness focus on the game trait and input handling
/forge:test-harness cover the REST API endpoints
/forge:test-harness expose the job queue and worker lifecycle
```

### `/forge:write`

`/forge:write` writes long-form prose for a writing project in this repo. It works for fiction, nonfiction, essays, worldbuilding documents, and similar projects. It adapts to whatever voice, tone, and structure the repo already has. It orients itself by reading existing content and any project guidance files before writing anything.

Manuscript content follows the `writing-style` skill's creative context automatically. Pass `--type-poetry` for verse, or `--work` to force the work context instead.

**It stops before pushing.** It never pushes or opens a PR. Run `/forge:ship` for that.

```text
/forge:write chapter 3: the city falls and the survivors regroup at the harbor
/forge:write section on the historical context of Markan priority
/forge:write revise the opening scene to establish the weather before the dialogue
/forge:write add the appendix covering common counterarguments
/forge:write the elegy for the fallen city --type-poetry
```

### `/forge:publish`

`/forge:publish` compiles a writing project into publishable ebook formats. It is the prose equivalent of `/forge:ship`. It accepts the same `writing-style` context flags as `/forge:write`. It targets Amazon KDP and Barnes and Noble Nook, and builds EPUB files with the right metadata, formatting, and structure for each. It does not upload anything. Uploading, pricing, and platform forms stay a manual step for the user.

This command needs [Pandoc](https://pandoc.org/installing.html) installed on your machine to generate EPUB files.

```text
/forge:publish
/forge:publish kindle
/forge:publish nook
/forge:publish epub
```

### `/forge:version`

`/forge:version` cuts a release. It determines the current version, bumps it, updates `VERSION.md` and `CHANGELOG.md`, and creates an annotated Git tag. It works for code projects, writing projects, and the forge plugin itself. It always asks you to confirm the new version before it tags anything.

It pushes the tag it creates, but it does not push branches or open PRs. CI pipelines can trigger on the tag to automate publishing or deployment.

```text
/forge:version patch
/forge:version minor
/forge:version major
/forge:version 2.0.0
```

## Skills

### `clean-architecture`

This is a reference skill. It defines Clean Architecture's layer responsibilities, the inward dependency rule, naming conventions, and a testing strategy for each layer. `/forge:scaffold` uses it to generate projects with correct layers. `/forge:review` uses it to spot real layer violations instead of guessing.

### `clean-code`

This is a reference skill. It defines code-level quality rules that apply inside any layer, in any language. It covers magic values, cognitive complexity, guard clauses, naming, single responsibility, locality, tempered DRY, parameter count, error handling, and comments. `/forge:do-work` follows it while implementing, and `/forge:review` uses it to judge code quality. Where `clean-architecture` governs the layers, `clean-code` governs what happens inside them.

### `infrastructure-as-code`

This is a reference skill. It defines how forge writes and applies OpenTofu. It covers the `infra/` directory layout, modules named by role, provider abstraction across `aws`, `gcp`, and `digitalocean`. It also covers remote state, the plan-before-apply discipline, and secrets handling. `/forge:scaffold` uses it to generate the `infra/` directory, and `/forge:deploy` uses it to apply that infrastructure safely.

### `conventional-commits`

This is a reference skill. It defines the commit subject format, the allowed commit types, and the body and footer rules. Every command that commits or writes a PR title follows it. You do not need to configure anything for this. It is built in.

### `writing-style`

This is a reference skill. It defines the writing rules for every piece of text forge produces, selected by context. The work context is the default. It covers development output, such as README content, PR descriptions, and commit messages. The creative context covers manuscript writing, such as prose and poetry, with voice, continuity, and pacing rules. Pass `--work` or a `--type-{name}` flag to `/forge:write` or `/forge:publish` when the automatic choice is not what you want. Both contexts share the mechanical rules that keep writing human.

### `token-budget`

This is a reference skill. It defines how every forge command manages token usage within Claude Code's rolling rate limits. It covers batch sizing, commit frequency, graceful stopping when limits approach, and resume path formatting. All commands follow it for session management.

### `web-browsing`

This is a behavioral skill. It makes the [`browse` CLI](https://docs.browserbase.com) the default browser whenever Claude reads or interacts with a webpage. JavaScript-rendered sites then return their real content instead of the empty shell WebFetch sees. Without the CLI, forge falls back to WebFetch and tells you how to install `browse`. Setup, mode selection, and session rules live in the skill itself.

## Why this exists

forge exists to run multiple independent side projects off one consistent workflow. It saves you from rebuilding process for each new project. `/forge:do-work`, `/forge:review`, and `/forge:ship` are not tied to any single venture. They are simply how work gets planned, built, and shipped, no matter what the project is.

`/forge:scaffold` is the one command tied to a specific bet right now. It is the engineering backbone behind a small Gumroad business that sells Clean Architecture starter templates. That business spans five project types and several languages each. Still, that is one income stream, not the point of the repo. The real point is having a single, dogfooded workflow. That workflow should travel with me across every project I am trying to turn into income. One example is the templates business. Another example is an ASCII roguelike built for Steam. Each new venture should start from a working process instead of starting from zero.

## Scope

Forge is built for professional and personal development work, not for academic use. Do not use it to complete coursework, assignments, or exams. If you are unsure whether AI-assisted tools are allowed, check your institution's academic integrity policy.

## License

MIT. See [LICENSE](LICENSE) for the full text.

## Inspiration

Forge took inspiration from [Shipyard](https://github.com/mattsears18/shipyard), an autonomous engineering loop plugin for Claude Code. Shipyard is a far more ambitious project. It handles parallel worker pools in isolated git worktrees. It also handles autonomous issue triage, audit pipelines, cost tracking, and PRs that merge automatically through CI.

We built forge because we needed something leaner. It is just its commands, with no orchestrator and no concurrency management. It is just a clean workflow that gets out of your way. If our needs grow to require parallel agents, automated backlog triage, or worktree isolation, we will adopt Shipyard. We will not rebuild those features ourselves. Different tools fit different stages.
