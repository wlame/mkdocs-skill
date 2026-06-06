# Unified Documentation Structure

Detailed content specifications for each page type in the organization documentation standard.
Every project uses this structure; sections not applicable to a project are omitted.

---

## index.md — Landing Page

**Purpose**: Answer "what is this, should I care, how do I start?" in 60 seconds.

**Required elements**:
1. H1: Project name as tagline — one sentence describing what it does and for whom.
2. A 2–3 sentence elevator pitch (the problem + the solution).
3. Feature list: 4–8 bullet points or a grid of feature cards with icons.
4. Quick install block: copy-pasteable, the single most common install path.
5. Quick-start snippet: the absolute minimum code to get a working result (real code, not pseudocode).
6. Navigation cards or a "What's next?" section linking to Getting Started, Use Cases, Reference.

**Anti-patterns to avoid**:
- Don't repeat the README verbatim.
- Don't put installation details here — link to getting-started.md.
- Don't list every feature — pick the 6 most distinctive.

---

## about.md — About & Design Goals

**Purpose**: Help users understand *why* this project exists and *what trade-offs were made*.

**Sections**:
- **Why this exists** — the specific problem that motivated creation; what alternatives were tried and why they fell short.
- **Design principles** — 3–5 guiding principles that drive decisions (e.g., "correctness over performance", "explicit over implicit").
- **Architecture summary** — 1 paragraph + optional Mermaid diagram; link to concepts/architecture.md for depth.
- **Comparison** — table comparing this project to 2–3 alternatives on key dimensions.
- **Known limitations** — honest list of what this tool doesn't do well or doesn't support.
- **Status and roadmap** — current stability (alpha/beta/stable), planned work if public.

---

## getting-started.md — Quick Start

**Purpose**: Zero to working result in under 10 minutes, for someone who has never used this before.

**Structure**:
1. **Prerequisites** — exact list: OS, runtime version, tools, credentials, permissions.
2. **Installation** — tabbed by install method (pip, brew, docker, binary download, build from source).
3. **First run** — the minimal working example. Must be < 20 lines. Must produce visible output.
4. **Expected output** — a code block showing exactly what the user should see.
5. **What just happened** — 3–4 sentences explaining the first run.
6. **Next steps** — links to: first real use case, configuration, full reference.

**Code example quality bar**:
- Must run as written with no modifications.
- Must use `localhost` or in-memory targets — no external services required.
- Must show a non-trivial result (not just "Hello, world").

---

## use-cases/*.md — Use Case Pages

**Purpose**: Show the project solving a real, named problem end-to-end.

**Page naming**: `use-cases/<verb>-<noun>.md` e.g., `use-cases/processing-large-files.md`

**Structure per page**:
1. **Scenario** — who is doing this, what they need, why this tool is the right choice.
2. **Prerequisites** — what must be set up before this walkthrough.
3. **Full working example** — complete code (not excerpts) that implements the use case.
4. **Walkthrough** — annotate each significant part of the example.
5. **Variations** — 2–3 "what if" variants (different config, different scale, different environment).
6. **Related pages** — links to relevant reference and concept pages.

**use-cases/index.md**:
- Brief intro paragraph.
- Card grid or list linking to each use case page with a one-line description.

---

## concepts/architecture.md — Architecture

**Purpose**: Explain the system's components and how they interact.

**Required**:
- Mermaid `flowchart TD` or `graph LR` showing all major components and their relationships.
- One paragraph per component: its responsibility, what it owns, what it depends on.
- Component table: name, description, when it's active.

**Optional but recommended**:
- Lifecycle diagram: how a request/message/job moves from input to output.
- Deployment topology diagram.

---

## concepts/data-flow.md — Data Flow

**Purpose**: Show how data moves through the system step by step.

**Required**:
- Mermaid `sequenceDiagram` for the primary data path.
- Numbered step list explaining each arrow in the diagram.
- A second diagram for error/failure paths if non-trivial.

---

## reference/cli.md — CLI Reference

**Purpose**: Complete, authoritative reference for every command and flag. Users should never need to run `--help` after reading this.

**Structure**:
- H1: CLI Reference
- Brief intro: how to invoke the tool, global flags (--help, --version, --verbose, etc.)
- One H2 per top-level command or subcommand group.

**Per command**:
```
## command-name

One-line description.

**Syntax**
```
tool command-name [flags] <required-arg>
```

**Flags**

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--flag-name` | `-f` | string | `""` | What it does |

**Examples**

??? example "Basic usage"
    ```bash
    tool command-name --flag value
    ```

??? example "Advanced: with optional flags"
    ```bash
    tool command-name --flag value --other-flag
    ```

**Exit codes**

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
```

---

## reference/api.md — REST API Reference

**Purpose**: Every endpoint documented with enough detail to call it without reading source code.

**Structure**:
- H1: API Reference
- Base URL, versioning, auth methods
- Error format (standard error response schema)
- Rate limiting / pagination conventions
- One H2 per resource group (Users, Items, Jobs, etc.)

**Per endpoint**:
```
### POST /resource

Create a new resource.

**Authentication**: Bearer token required

**Request**

=== "Body"
    ```json
    {
      "field": "value",
      "count": 42
    }
    ```

=== "Headers"
    | Header | Required | Description |
    |--------|----------|-------------|
    | Authorization | Yes | Bearer <token> |

**Response**

=== "201 Created"
    ```json
    { "id": "abc123", "field": "value" }
    ```

=== "400 Bad Request"
    ```json
    { "error": "validation_failed", "message": "field is required" }
    ```

=== "curl"
    ```bash
    curl -X POST https://api.example.com/resource \
      -H "Authorization: Bearer $TOKEN" \
      -H "Content-Type: application/json" \
      -d '{"field": "value", "count": 42}'
    ```
```

---

## reference/configuration.md — Configuration Reference

**Purpose**: Every configuration option documented with type, default, and example.

**Structure**:
- How to configure (file path, env vars, precedence order)
- Full example config file in a collapsible block
- One H2 per config section/namespace

**Per option table**:
```markdown
| Key | Type | Default | Env var | Description |
|-----|------|---------|---------|-------------|
| `server.host` | string | `"0.0.0.0"` | `APP_HOST` | Bind address |
| `server.port` | int | `8080` | `APP_PORT` | Listen port |
```

---

## operations/deployment.md — Deployment

**Purpose**: Get from zero to production safely.

**Sections**:
1. **Requirements** — CPU, memory, disk, OS, runtime.
2. **Install** — tabbed by method (pip, docker, k8s, systemd).
3. **Docker** — `docker run` command and compose file example.
4. **Kubernetes** — Helm chart or raw manifest if provided.
5. **Configuration for production** — key settings that differ from development.
6. **First-run checklist** — table of items to verify before going live.
7. **Upgrades** — how to upgrade to a new version safely.

---

## operations/monitoring.md — Observability

**Purpose**: Teach operators how to know the system is healthy and diagnose it when it's not.

**Sections**:
1. **Health endpoint** — path, expected response, what "healthy" means.
2. **Metrics** — all exported metrics with labels and descriptions.
3. **Logs** — log format, key fields, important log messages and what they mean.
4. **Alerts** — recommended alert rules (as Prometheus YAML or narrative).
5. **Dashboards** — link to Grafana dashboard JSON if available.

---

## faq.md — FAQ

**Purpose**: Answer the 10 questions users ask most, before they have to ask them.

**Format**: H2 per question, 1–3 paragraph answer, with code blocks where helpful.

**Required questions to consider**:
1. How do I install this? (link to getting-started.md)
2. What's the difference between this and [main alternative]?
3. Does this support [common feature people assume]?
4. How do I configure [most common config scenario]?
5. What happens when [most common failure mode]?
6. How do I debug when nothing works?
7. Is there a Docker image / can I run this without installing?
8. How do I contribute?
9. What's the performance overhead / resource usage?
10. Where can I get help?

---

## troubleshooting.md — Troubleshooting

**Purpose**: Let users self-diagnose and fix common problems without opening an issue.

**Format**: One H2 per problem category.

**Per problem entry**:
```markdown
### Error: <exact error message verbatim>

**When you see this**: Describe the scenario.

**Cause**: What causes this.

**Fix**:
```bash
# Command to fix it
```

**Still broken?**: Link to issue tracker or next diagnostic step.
```

---

## changelog.md — Changelog

**Format**: Keep a Changelog style (https://keepachangelog.com/)

```markdown
# Changelog

## [Unreleased]

## [1.2.0] - 2026-05-01

### Added
- New feature description

### Changed
- Changed behavior description

### Fixed
- Bug fix description

### Removed
- Removed feature
```

If a `CHANGELOG.md` or `HISTORY.md` exists in the repo root, import its content here with a `--8<--` snippet reference.

---

## contributing.md — Contributing

**Purpose**: Make it easy to submit a useful contribution on the first try.

**Sections**:
1. **Ways to contribute** — issues, docs, code, discussion.
2. **Development setup** — clone, install dev deps, run tests in < 5 commands.
3. **Code standards** — linting, formatting, type checking commands.
4. **Test requirements** — how to run tests, coverage expectations.
5. **Pull request process** — branch naming, commit message style, what reviewers look for.
6. **Release process** — how versions are cut (if relevant).

If `CONTRIBUTING.md` exists in the repo root, either import it via snippet or keep this page as a short intro that links to it.
