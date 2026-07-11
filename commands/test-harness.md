---
description: Generate an MCP server that exposes the project's domain and application layers as tools Claude can call for headless testing and QA.
argument-hint: "[optional focus area, e.g. \"the game trait and input handling\" or \"the REST API endpoints\"]"
---

# /forge:test-harness

You are generating an MCP server that lets Claude Code interact with this project directly. The harness reads the project's domain and application layers, finds the public API surface, and exposes it as a headless testing interface. With it connected, Claude can launch the application, send inputs, read state, and find bugs without a human in the loop.

This command produces the MCP server code. It does not run the QA pass. The QA pass is a separate task the user kicks off after the harness is built and connected.

The optional focus area is: $ARGUMENTS

If `$ARGUMENTS` is empty, generate a harness covering the full public API surface. If the user names specific areas, scope the harness to those.

```
/forge:test-harness
/forge:test-harness focus on the game trait and input handling
/forge:test-harness cover the REST API endpoints and database queries
/forge:test-harness expose the job queue and worker lifecycle
```

## Orientation

Before generating anything, read the project the same way `/forge:do-work` does.

- Identify the language, framework, and package manager from manifest files, such as `package.json`, `go.mod`, `pyproject.toml`, `*.csproj`, or `Cargo.toml`. Do not assume any of these in advance.
- Read `CLAUDE.md`, `README.md`, and any architecture docs for project-specific conventions.
- Find the domain and application layers. These are the layers the harness wraps. See the `clean-architecture` skill in this plugin for how the layers are defined and where their boundaries sit. The harness never wraps infrastructure or presentation directly. It talks to the same interfaces the presentation layer uses.
- Identify the public types, traits, interfaces, and functions that define the project's API surface. These become the MCP tools.
- Check if an MCP server already exists in the project. If one does, ask whether to extend it or replace it before generating anything.

## What to generate

### MCP server module

Create a new module, crate, or package in the project. The location depends on the language. Detect the language during orientation and pick the matching layout.

- Rust: a new crate in the workspace under `mcp-server/`.
- Go: a new `cmd/mcp-server/` binary.
- PHP: a new `mcp-server/` directory with its own composer dependencies.
- TypeScript/Node: a new `mcp-server/` directory with its own `package.json`.
- Python: a new `mcp_server/` package.

If the project uses a language not listed here, follow that language's idiomatic layout. Create a standalone executable that lives alongside the project and depends on it.

The server uses stdio transport. It registers tools that map to the project's public API surface. The harness depends on the project. The project never depends on the harness.

### Tool categories

Generate tools in these categories based on what the project exposes. Not every project needs every category, but cover each one the project's API surface supports.

**Lifecycle tools**: start, stop, reset, and configure the application or its components. For a game this is starting and stopping a game session. For an API this is starting the server with a test database. For a worker this is connecting to a test queue.

**Action tools**: send inputs or trigger operations. For a game this is player input. For an API this is sending HTTP requests. For a worker this is pushing jobs onto the queue. Include a batch or sequence tool that lets Claude chain multiple actions in one call, to cut down on round trips.

**Inspection tools**: read the current state. Return structured JSON that Claude can parse. Include detailed state, such as full entity positions and all fields. Also include summary state, such as a grid rendered as text or a single status line. Keep the response format consistent across all inspection tools.

**Targeted testing tools**: override state for edge case testing. Set a score to one below an achievement threshold. Place an entity at a specific position. Fill a buffer to near capacity. These are test-only tools. Mark them clearly as such. They exist to let Claude reach specific scenarios without playing through the entire happy path.

### Configuration

Add an MCP server entry to `.claude/settings.json` so the harness auto-connects when Claude Code opens the project. Use the run command and arguments appropriate for the detected language.

```json
{
  "mcpServers": {
    "{project-name}-test": {
      "command": "{language-appropriate-run-command}",
      "args": ["{language-appropriate-args}"],
      "cwd": "."
    }
  }
}
```

If `.claude/settings.json` already exists, merge this entry into the existing `mcpServers` object instead of overwriting the file.

### Test plan template

Generate a markdown file at `TEST_PLAN.md` in the project root. This is a checklist Claude can follow when running the QA pass. Follow the `writing-style` skill in this plugin for its prose. Structure it as:

- General checks that apply to every project, such as startup, shutdown, reset, and state consistency.
- Feature-specific checks derived from the domain layer's public API.
- Edge case checks derived from boundary values, empty inputs, and error paths.
- Achievement or milestone checks, if the project has them.

The test plan is a starting point. The user or Claude can extend it as bugs are found.

## Workflow

Follow the `token-budget` skill in this plugin for session management. This command generates a full module and will make many tool calls. Commit after each logical step and check in with the user before large batches.

1. State what you found during orientation. List the public API surface you plan to wrap. List the tool categories and specific tools you will generate. Keep this brief.
2. Create the MCP server module with all tools.
3. Create the MCP configuration entry.
4. Create the `TEST_PLAN.md` file.
5. Build and verify the MCP server compiles and starts without errors, using the project's own toolchain.
6. Commit using the `conventional-commits` skill in this plugin. Use `feat(test-harness):` as the scope.
7. Report what was created, how to connect it, and suggest the user run the QA pass next. Follow the `writing-style` skill for the report's prose.

## What this command does not do

- It does not run the QA pass. That is a separate task the user starts after the harness is connected.
- It does not modify the project's production code. The harness depends on the project. The project does not depend on the harness.
- It does not push or open a PR. That is `/forge:ship`.
- It does not wrap infrastructure or presentation layers. The harness uses the same ports the presentation layer uses. It is another adapter, not a wrapper around existing adapters.
