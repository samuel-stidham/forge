---
name: infrastructure-as-code
description: Reference for OpenTofu infrastructure conventions, including module layout, provider abstraction, remote state, plan-before-apply discipline, secrets handling, and the optional cache and search modules. Use when scaffolding cloud infrastructure or deploying with OpenTofu.
---

# Infrastructure as Code Reference

This skill is the shared source of truth for `/forge:scaffold` and `/forge:deploy`. It defines how forge writes and applies OpenTofu. It is provider-aware but never provider-locked. Apply the conventions, not one vendor's defaults.

forge uses OpenTofu only. OpenTofu is the open source fork of Terraform. The generated HCL works with both the `opentofu` and `terraform` binaries. If a user asks for `terraform`, explain this and continue with OpenTofu.

## The `infra/` directory

Infrastructure code lives in an `infra/` directory at the repo root. This is separate from any `infrastructure/` application layer. The two never mix.

The `infra/` root holds these files:

- `main.tf`: wires the modules together for the chosen provider.
- `variables.tf`: every input the configuration accepts, with types and descriptions.
- `outputs.tf`: the values a deploy needs afterward, such as a host or a URL.
- `providers.tf`: the provider blocks and required versions.
- `terraform.tfvars.example`: placeholder values for every variable. No real secrets.

It also holds a `modules/` directory. Each module owns one role.

## Modules by role

Name modules by the role they fill, not by a brand. This keeps the layout stable across providers.

Present whenever the project needs them:

- `networking/`: the private network, subnets, and firewall rules.
- `container/`: the compute that runs the app, a container service or a VM.
- `database/`: the primary datastore, when the project type requires one.
- `secrets/`: the secret store and the bindings the app reads at runtime.

Optional and opt-in:

- `cache/`: an in-memory cache or queue backend. The default implementation is Redis.
- `search/`: a search engine. The default implementation is Meilisearch.

Generate an optional module only when the target project actually declares it. A project that uses neither a cache nor a search engine gets neither module. Never add a module the app does not use.

## Detecting what to provision

Read the target repo before writing infra. The app's own configuration states its dependencies. Look at these signals:

- the services the app runs in its `docker-compose.yml`,
- the keys and values in its `.env.example`,
- the framework config, such as a Laravel `CACHE_STORE`, `QUEUE_CONNECTION`, or `SCOUT_DRIVER`.

Provision the database, cache, or search module only when these signals call for it. The infra should mirror the app, not a wishlist.

## Provider abstraction

`--provider` accepts `aws`, `gcp`, or `digitalocean`. Each role maps to that provider's nearest managed service. It falls back to a self-hosted container when no managed option exists.

- Cache (Redis): AWS ElastiCache, GCP Memorystore, or DigitalOcean Managed Redis. A self-hosted Redis container is the low-cost fallback.
- Search (Meilisearch): no major cloud offers managed Meilisearch. Self-host it as a container or a small VM. Give it a persistent volume for the index. State this in the generated README so nobody hunts for a managed service that does not exist.

Keep provider-specific resources inside the modules. The `main.tf` wiring should read the same shape no matter the provider.

## Remote state

Configure a remote backend for state. Local state does not survive a team or a second machine.

State is sensitive. It can hold secret values in plain text. Never commit state to git. The `.gitignore` must exclude `*.tfstate`, `*.tfstate.*`, and `.terraform/`.

## Plan before apply

Every apply follows the same discipline. Never apply blind.

1. Run `tofu fmt` to format the configuration.
2. Run `tofu validate` to catch errors early.
3. Run `tofu plan` to show what will change.
4. Apply only after a human reads the plan and confirms.

Never run `tofu destroy` as part of a deploy. Never auto-approve an apply that creates or replaces resources without showing the plan first.

## Secrets

Secrets never live in HCL or in committed files.

- `terraform.tfvars.example` holds placeholders only. The real `terraform.tfvars` is gitignored.
- Runtime secrets live in the provider's secret manager, wired through the `secrets/` module.
- The app reads secrets from the environment or the secret store, never from infra source.
