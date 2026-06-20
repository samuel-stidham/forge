---
description: Generate a complete, working Clean Architecture project template for a given language and project type. Supports an optional --infra opentofu flag for cloud provisioning.
argument-hint: "{language} {project-type} [--infra opentofu --provider aws|gcp|digitalocean]  e.g. \"go rest-api --infra opentofu --provider aws\""
---

# /forge:scaffold

You are generating a complete, real, working Clean Architecture project from scratch in the current directory. Use a new subdirectory instead if the current directory is non-empty and the user confirms. This is not a documentation exercise. Every file you create must compile, run, and pass as written. Do not leave placeholder comments standing in for logic, such as `// TODO: implement this`.

Before writing any code, use the `clean-architecture` skill bundled with this plugin. It defines the layering rules, dependency direction, and naming conventions to apply. Do not improvise a different architecture.

## Parsing $ARGUMENTS

`$ARGUMENTS` is `{language} {project-type}`, with two optional flags. The flags are `--infra opentofu` and `--provider {name}`. Examples include `go rest-api`, `typescript webhook-gateway`, `python cli-tool`, and `go rest-api --infra opentofu --provider aws`.

- If either the language or the project type is missing or ambiguous, ask the user to clarify. Do not guess a language or a type.
- **Supported project types:**
  - `rest-api`: an HTTP API with routing, request and response handling, and a persistence port.
  - `cli-tool`: a command-line application with subcommands and structured output.
  - `background-worker`: a queue or event consumer that processes jobs asynchronously.
  - `webhook-gateway`: receives external webhooks, validates and verifies them, and dispatches to internal handlers.
  - `task-runner`: schedules and executes recurring or one-off tasks and jobs.
- Language is not limited to a fixed list. Use whatever the user names, and apply that language's real idioms and tooling. This means its actual package manager, test framework, and conventional project layout, not a generic templated approximation.
- `--infra` and `--provider` are optional flags. See `Infrastructure with --infra` below for how to parse and apply them.

## Infrastructure with `--infra`

This flag is optional. Without it, generate no infrastructure code. The flag is opt-in.

When the user passes `--infra opentofu`, the generated project also gets an `infra/` directory. This is separate from the `infrastructure/` layer described above. The `infra/` directory holds `main.tf`, `variables.tf`, `outputs.tf`, `providers.tf`, and `terraform.tfvars.example`. It also holds a `modules/` directory with `networking/`, `database/`, `container/`, and `secrets/` subdirectories.

`--provider` is required whenever `--infra` is used. Accept `aws`, `gcp`, or `digitalocean`. If the user names a different provider, ask them to pick one of these three instead of guessing.

The only accepted value for `--infra` is `opentofu`. If the user passes `--infra terraform`, explain that forge uses OpenTofu only. OpenTofu is the open source fork of Terraform. Note that the generated HCL files still work with both the `opentofu` and `terraform` binaries.

The generated configs connect to the Dockerfile the scaffold already creates. They set up the minimum infrastructure the project type needs. This includes networking, such as a VPC and subnets. It includes compute, through a container service or a VM. It includes a database, when the project type requires one. It also includes secrets management. Add a deploy stage to the GitHub Actions CI pipeline from the file list below.

## What every generated project must include

Include all of the following, regardless of language or type.

1. **Layered structure**, following the `clean-architecture` skill:
   - `domain/`: entities and business rules, with zero external dependencies.
   - `application/`: use cases and ports, meaning interfaces, that the use cases need. No framework or infrastructure imports.
   - `infrastructure/`: concrete adapters that implement application ports, such as a database, a message queue, or an HTTP client.
   - `presentation/`, or `cmd/`/`interfaces/` if more idiomatic for the language: the entry point that wires everything together as the composition root. It exposes the project type's interface, such as HTTP handlers, CLI commands, or a consumer loop.
   - Use whatever directory names are idiomatic for the language's ecosystem, while keeping these four responsibilities. Do not force a Go-style layout onto a Rails or Rails-adjacent project if it fights the framework.
2. **Dockerfile**: a multi-stage build. The final stage runs as a non-root user and only ships the runtime artifact. Keep the build toolchain out of the final image.
3. **docker-compose.yml**: runs the service plus any dependency it actually needs, such as a database or queue for a `rest-api` or `background-worker`. Use sane defaults and no hardcoded secrets.
4. **Makefile**: at minimum, `build`, `run`, `test`, `lint`, and `docker-build` targets, using the language's real toolchain commands.
5. **GitHub Actions CI**, at `.github/workflows/ci.yml`: installs dependencies, runs lint, and runs tests, on push and on pull request. Add a deploy stage when `--infra opentofu` is used.
6. **.env.example**: every environment variable the project reads, with a placeholder value and a one-line comment on its purpose. No real secrets.
7. **README.md**: what the project is, plus an architecture overview. The overview should cover the four layers and how data flows through them for this project type. It should also explain how to run the project locally, how to run it in Docker, and how to test it.
8. **Example tests per layer**: at least one real test for domain logic, which should be pure and need no mocks. At least one test for an application use case, using a fake or mock for its port. At least one test for an infrastructure adapter or the presentation entry point, written as an integration test. Tests must actually run and pass.
9. **.editorconfig**: matches the language's standard conventions, such as indent style, indent size, charset, and line endings.
10. **.gitignore**: appropriate for the language and its tooling, such as build artifacts, dependency caches, env files, and IDE files.
11. **LICENSE**: MIT, copyright the current year. Use "the project owner" unless the user specifies a name. Ask if the name is unclear instead of guessing.

## Process

Follow the `token-budget` skill in this plugin for session management. `/forge:scaffold` generates many files and will exceed 30 tool calls for most project types; check in with the user after completing each major layer before moving to the next. Commit each layer as it is finished. Default to Opus for this command.

1. State the language, the project type, any flags used, and the file tree you are about to create. Keep this brief. It is not a full design document.
2. Create the files for real. Write working code, working config, and working CI. Do not write stubs.
3. After scaffolding, run the project's own test and lint commands, using the Makefile targets you just created. When `--infra opentofu` was used, also run `tofu validate` against the generated configs. Confirm everything actually works before you report success. If something fails, fix it. Do not report a scaffold as done with a known-broken build.
4. Report what was created, how to run it, and any decisions you made that the user might want to revisit. This includes choices like the HTTP framework, the database driver, or the queue library.

Do not push, commit, or open a PR as part of this command. Scaffolding only produces local files. If the user wants those committed, that is a separate, explicit step. Use `/forge:do-work` or `/forge:ship` for that.
