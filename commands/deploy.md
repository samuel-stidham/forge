---
description: Deploy the current project using its own infrastructure. Detects an OpenTofu infra/ directory, a Dockerfile, and the app's release steps, then applies them safely.
argument-hint: "[optional target, e.g. staging|production]"
---

# /forge:deploy

You are deploying the current project using infrastructure that already exists in the repo. This command is the verb that follows `/forge:scaffold --infra`. Scaffold writes the infrastructure. Deploy applies it.

The optional target is: $ARGUMENTS

When the target is empty, detect the default environment from the repo. Ask the user only if more than one environment exists and the choice is unclear.

Follow the `infrastructure-as-code` skill in this plugin for every OpenTofu action. Follow the `token-budget` skill for session management.

## Detect, never assume

forge is repo-agnostic. Read the repo to learn how it deploys. Do not assume a cloud, a registry, or a pipeline. Look for these signals first:

- an `infra/` directory with OpenTofu configs,
- a `Dockerfile` and any image registry named in CI or env,
- a deploy stage in `.github/workflows/`,
- existing deploy scripts, a `Makefile` deploy target, or notes in `CLAUDE.md` and `README.md`.

If you find no infrastructure and no Dockerfile, stop. Report that there is nothing to deploy. Suggest `/forge:scaffold --infra opentofu` to create it.

## Pre-flight checks

Run these before touching anything remote. If one fails, stop and report it instead of forcing past it.

- The working tree is clean. Deploy from committed state, not from local edits.
- The required CLI tools are installed, such as `tofu` and the container or cloud CLI the repo uses.
- The provider credentials are present in the environment. Never read them from source.

## Workflow

### 1. Build and publish the image

When the repo has a `Dockerfile` and a target registry, build the image and push it. Tag it with the commit SHA, not only `latest`, so the deploy is traceable. Skip this step when the project type ships no container.

### 2. Apply infrastructure

When an `infra/` directory exists, apply it with the plan-before-apply discipline the `infrastructure-as-code` skill defines. Run `tofu init` first if the working directory is not initialized. Never print secret values from variables, outputs, or state.

### 3. Run release steps

Run the app's own release steps only when the project defines them. Examples include database migrations, a cache warm, or an asset build. Detect these from the framework and the `Makefile`. Do not invent a release step the project does not have.

### 4. Verify and report

Confirm the deploy succeeded. Read the relevant OpenTofu outputs, such as a host or a URL, and check the app responds. Report what changed, the image tag deployed, the OpenTofu outputs, and any manual step that remains.

## Boundaries

This command performs an operational deploy. It does not open a pull request and it does not cut a release. Use `/forge:ship` for PRs and `/forge:version` for release tags. Run `/forge:deploy` only when the user explicitly invokes it, never as a side effect of another command.
