# Repository Analysis Procedure

The shared analysis phase used by `/ghdoc` (Phase 1) and `/ghdoc-update`. Launches 4 Explore
agents in parallel and persists their findings to `.claude/ghdoc-analysis.md` so every downstream
agent works from one inventory. Ensure `.claude/` is gitignored before writing.

## The four analysis agents (launch in parallel)

**Agent A — Core Identity**
"Read all README files (root README.md, any README in subdirectories), the root pyproject.toml / setup.py / setup.cfg / package.json / Cargo.toml / go.mod / pom.xml (whichever exists), LICENSE, and any CHANGELOG or HISTORY file. Extract: (1) project tagline and one-paragraph purpose, (2) key features as a bullet list, (3) tech stack and language, (4) current version if any, (5) target audience, (6) links to any existing docs. Return as structured text."

**Agent B — CLI Surface**
"Search for CLI entry points: look in pyproject.toml [project.scripts] / [tool.poetry.scripts], package.json bin, Makefile targets, any `main.go`, `main.py`, `cli.py`, `cmd/` directory, `click`/`typer`/`argparse`/`cobra`/`clap` usage. For each command and subcommand found: extract the name, description/docstring, all flags/options with their types and defaults, and any examples in the source. Return a complete structured list of every command path with its parameters. If no CLI is found, return 'No CLI detected'."

**Agent C — API Surface**
"Search for HTTP/REST API definitions: FastAPI/Flask/Django/Express/Gin/Spring route decorators, OpenAPI/Swagger YAML or JSON files, any `routes.py`, `router.py`, `api.py`, `views.py`, `controllers/` directory. For each endpoint found: HTTP method, path, description, request body schema, response schema, auth requirements. Also look for gRPC proto files. Return a structured list of all endpoints grouped by resource. If no API is found, return 'No REST API detected'."

**Agent D — Architecture & Configuration**
"Search for: (1) config files/schemas — `config.py`, `settings.py`, `config.yaml`, `*.env.example`, Pydantic Settings models, env vars in README — list every key with type, default, and description; (2) architectural pattern — is this a library, CLI tool, web service, daemon, or combination?; (3) data flow — find any architecture diagrams, sequence diagrams, Mermaid code, or prose describing how data moves through the system; (4) deployment targets — Docker, k8s manifests, Helm charts, docker-compose, systemd units; (5) notable integrations — databases, message queues, external services. Return all findings structured."

After all 4 agents return, synthesize their findings. Read any key files they identify.

## Analysis file format

Save to `.claude/ghdoc-analysis.md`:

```markdown
# ghdoc analysis — <project name>

## Identity
<Agent A findings>

## CLI commands
<Agent B findings — full command list with all flags>

## API endpoints
<Agent C findings — full endpoint list>

## Architecture & configuration
<Agent D findings — config keys, architecture type, data flow, deployment>

## Concept inventory
<Synthesized list of all significant named concepts: command names, config keys,
API endpoints, components, important proper nouns — one per line>

## Existing docs (if found)
<For each existing markdown file: path, summary, and verbatim text of any non-obvious
knowledge — design rationale, edge cases, operational lessons, migration notes, anything
not directly derivable from reading the source code. Mark clearly which file each came from.
If no existing docs were found, write: "No existing docs detected.">
```

The concept inventory and existing-docs sections are both critical — they feed the content agents
and the cross-linking agent.
