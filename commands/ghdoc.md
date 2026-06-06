---
description: Generate a complete MkDocs documentation site for this repository — structured pages, Material theme, GitHub Actions CI, and GitHub Pages setup instructions
argument-hint: "[color-scheme=indigo|teal|blue-grey|deep-purple|orange] [versioned=yes|no] [api=yes|no] [override: <free text instructions>]"
---

# ghdoc — Documentation Site Generator

Generate a complete, structured MkDocs documentation site for this repository, following the organization documentation standard. Work through the phases below in order, stopping for user confirmation at the required checkpoints.

User arguments: $ARGUMENTS

Parse `$ARGUMENTS` for these optional overrides:
- `color-scheme=<name>` — override the default color scheme (indigo | teal | blue-grey | deep-purple | orange)
- `versioned=yes` — enable mike versioning
- `api=yes` — force API reference generation even if not auto-detected
- `override: <text>` — free-form instructions that override any defaults

---

## Pre-check: Existing docs detection

Before doing anything else, check whether a `docs/` directory already exists:

```bash
find docs -name "*.md" 2>/dev/null | head -50
```

**If `docs/` contains markdown files:**

Stop and notify the user prominently:

> **Found existing documentation** — `docs/` contains N markdown files.
> Reading all of them now to preserve their knowledge before generating the new site structure.
>
> Files found:
> - `docs/some-file.md` (N lines)
> - …
>
> This content will be carefully integrated into the new docs. Nothing will be discarded.

Then launch one Explore agent to read every existing markdown file in `docs/` and return a structured summary:

"Read every markdown file under `docs/`. For each file return: (1) the file path, (2) a one-paragraph summary of what it covers, (3) any content that is non-obvious or not derivable from source code — things like design rationale, undocumented edge cases, known issues, operational lessons, migration notes, or any prose that represents hard-won knowledge. Return the full text of any section that contains such non-obvious content verbatim. Finally, list any `mkdocs.yml` if present and its full nav structure."

Save these findings as a section in `.ghdoc-analysis.md` (see below). Then ask the user:

> I found [N] existing doc pages. I'll read and preserve all the knowledge in them and integrate it into the new structure. Does that sound right, or are there specific files you want to exclude?

Wait for the user's reply before proceeding to Phase 1.

**If `docs/` does not exist or contains no markdown files:** proceed directly to Phase 1 with no interruption.

---

## Phase 1: Repository Analysis

**Goal**: Understand what the project is, what it does, and what doc sections it needs. Then save findings to disk so all downstream agents can read them.

Launch 4 Explore agents in parallel. Each agent must return structured findings:

**Agent A — Core Identity**
"Read all README files (root README.md, any README in subdirectories), the root pyproject.toml / setup.py / setup.cfg / package.json / Cargo.toml / go.mod / pom.xml (whichever exists), LICENSE, and any CHANGELOG or HISTORY file. Extract: (1) project tagline and one-paragraph purpose, (2) key features as a bullet list, (3) tech stack and language, (4) current version if any, (5) target audience, (6) links to any existing docs. Return as structured text."

**Agent B — CLI Surface**
"Search for CLI entry points: look in pyproject.toml [project.scripts] / [tool.poetry.scripts], package.json bin, Makefile targets, any `main.go`, `main.py`, `cli.py`, `cmd/` directory, `click`/`typer`/`argparse`/`cobra`/`clap` usage. For each command and subcommand found: extract the name, description/docstring, all flags/options with their types and defaults, and any examples in the source. Return a complete structured list of every command path with its parameters. If no CLI is found, return 'No CLI detected'."

**Agent C — API Surface**
"Search for HTTP/REST API definitions: FastAPI/Flask/Django/Express/Gin/Spring route decorators, OpenAPI/Swagger YAML or JSON files, any `routes.py`, `router.py`, `api.py`, `views.py`, `controllers/` directory. For each endpoint found: HTTP method, path, description, request body schema, response schema, auth requirements. Also look for gRPC proto files. Return a structured list of all endpoints grouped by resource. If no API is found, return 'No REST API detected'."

**Agent D — Architecture & Configuration**
"Search for: (1) config files/schemas — `config.py`, `settings.py`, `config.yaml`, `*.env.example`, Pydantic Settings models, env vars in README — list every key with type, default, and description; (2) architectural pattern — is this a library, CLI tool, web service, daemon, or combination?; (3) data flow — find any architecture diagrams, sequence diagrams, Mermaid code, or prose describing how data moves through the system; (4) deployment targets — Docker, k8s manifests, Helm charts, docker-compose, systemd units; (5) notable integrations — databases, message queues, external services. Return all findings structured."

After all 4 agents return, synthesize their findings. Read any key files they identify.

**Save findings to `.ghdoc-analysis.md`** in the repo root. Structure it as:

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

The concept inventory and existing docs sections are both critical — they are passed to content agents in Phase 4 and the cross-linking agent in Phase 5.

---

## Phase 2: Structure Proposal

**Goal**: Propose a documentation structure tailored to what was found, and get user approval before writing anything.

Based on Phase 1 findings, determine which page sections apply:

**Always required:**
- `index.md` — landing page with tagline, key features, quick-start snippet, links
- `about.md` — project purpose, design philosophy, why it exists, comparison to alternatives
- `getting-started.md` — fastest path from zero to first working result
- `use-cases/index.md` — overview of common use cases
- `faq.md` — frequently asked questions
- `troubleshooting.md` — known issues, error messages, diagnostic steps
- `changelog.md` — version history
- `contributing.md` — how to contribute

**Include if CLI tool detected:**
- `reference/cli.md` (or `reference/cli/<command>.md` — one page per top-level command if more than 6 commands)

**Include if REST API detected:**
- `reference/api.md` — all endpoints with examples, or per-resource pages if > 10 endpoints

**Include if configuration is non-trivial (more than 5 config keys):**
- `reference/configuration.md` — all config options, types, defaults, env vars

**Include if architecture/data flow is documented:**
- `concepts/architecture.md` — system design, component roles
- `concepts/data-flow.md` — how data moves through the system with Mermaid diagrams

**Include if deployment/ops is relevant:**
- `operations/deployment.md` — install, docker, k8s, production checklist
- `operations/monitoring.md` — metrics, logs, health checks, observability

**Use cases: propose specific slugs** (2–5 pages) based on Phase 1 findings:
- For each proposed use case: show `use-cases/<slug>.md` — one-line scenario description

**Include if Python package with public API:**
- `reference/api-reference.md` — generated from docstrings via mkdocstrings

Present the proposed structure to the user as a concrete nav tree with real slug names for the use cases based on what was found. Show which pages are required vs. detected as needed.

**Color scheme options** — present all five with a brief personality note:

| Option | Primary | Accent | Personality |
|--------|---------|--------|-------------|
| `indigo` **(default)** | indigo | blue | Professional, trustworthy |
| `teal` | teal | cyan | Modern, clean, technical |
| `blue-grey` | blue-grey | blue | Muted, sophisticated |
| `deep-purple` | deep purple | purple | Creative, bold |
| `orange` | orange | amber | Energetic, open-source |

If `color-scheme=<name>` was passed in `$ARGUMENTS`, highlight it as pre-selected.

**If existing docs were found**, include this notice before asking for confirmation:

> **Existing docs will be integrated** — the content from the following files will be preserved and merged into the new structure:
> - `docs/<file>.md` → will be merged into `<target page>`
> - …
>
> Any non-obvious knowledge (design rationale, edge cases, operational notes) found in existing pages is recorded in `.ghdoc-analysis.md` and will be given to the content agents. If an existing page maps directly to a new page, its content takes priority over generated boilerplate.

**STOP and wait for user confirmation.** Ask:
1. Does the proposed nav structure look right? Any pages to add, remove, or rename?
2. Are the proposed use-case slugs correct? Add, remove, or rename any?
3. Which color scheme would you like?
4. Do you have a logo file path to use? (otherwise a text logo will be used)
5. Any other overrides or instructions?

Do NOT proceed to Phase 3 until the user explicitly confirms.

---

## Phase 3: Infrastructure Files

**Start by reading two reference files** (do this before writing any files):
- Read `skills/ghdoc/references/mkdocs-standard.md` — for plugin versions, feature flags, and CI workflow template
- Read `skills/ghdoc/references/color-schemes.md` — for the exact Material palette YAML for the selected color scheme

Then write all MkDocs infrastructure:

**3a. Write `mkdocs.yml`** using the confirmed structure and color scheme.
- Use the exact nav from the confirmed proposal (with real page slugs — no placeholders)
- Apply the selected palette YAML verbatim from `references/color-schemes.md`
- Enable all baseline plugins from the standard: search, tags, privacy, git-revision-date-localized, git-authors, redirects, glightbox
- Add mkdocstrings if Python API was detected
- Set `strict: true`
- Detect repo remote: run `git remote get-url origin`. If it succeeds, set `repo_name` and `repo_url` from the output. If it fails or has no remote yet, set those fields to `<!-- TODO: set repo_url -->` and note this for the user.
- Set fonts: `text: Inter, code: JetBrains Mono`
- Add social links for any registries found (PyPI, Docker Hub, etc.)

**3b. Write `requirements-docs.txt`** with pinned versions from the standard (read from `references/mkdocs-standard.md`). Uncomment mkdocstrings lines if Python API detected.

**3c. Write `.github/workflows/docs.yml`** — use the workflow template from `references/mkdocs-standard.md` verbatim, adapting only the trigger paths if the docs are in a non-standard location.

**3d. Write `docs/assets/stylesheets/extra.css`** with minimal org branding:
```css
/* Organization baseline — keep shallow */
:root {
  -webkit-font-smoothing: antialiased;
}

.md-code__content {
  border-radius: 4px;
}
```

**3e. Create placeholder file `docs/assets/.gitkeep`** so the assets directory is tracked by git. If a logo path was provided by the user, note its expected location in a comment.

---

## Phase 4: Content Generation

**Goal**: Write all confirmed doc pages with real, accurate content from Phase 1 findings.

Before launching content agents, read `.ghdoc-analysis.md` to have the full Phase 1 inventory in scope — including the "Existing docs" section if present.

Launch parallel agents, one per major section. Each agent prompt must:
1. Start with: "Read `skills/ghdoc/references/page-templates.md` for the template structure to follow."
2. Include the relevant section of `.ghdoc-analysis.md` as inline context (paste the specific section text into the prompt).
3. Include the full "Existing docs" section from `.ghdoc-analysis.md` in every agent prompt.
4. Specify exactly which files to write and their paths.

**Merge rule for existing content**: if an existing page maps to a new page you are writing, do not start from the template — start from the existing content and improve/extend it. Preserve all non-obvious knowledge verbatim (move it into the appropriate section of the new page structure). Only replace boilerplate, outdated install instructions, or content that is clearly superseded by what was found in the source analysis. When in doubt, keep the existing prose and add new content around it.

**Agent assignments** (adapt to confirmed structure, one agent per group):

1. **Landing + About**: Write `docs/index.md` and `docs/about.md`
   - Include Agent A findings (identity, features, purpose)
   - Include Agent D findings (architecture type, limitations)

2. **Getting started**: Write `docs/getting-started.md`
   - Include Agent A findings (install methods from tech stack)
   - Include Agent B CLI findings (the first-run command)

3. **Use cases**: Write `docs/use-cases/index.md` and each `docs/use-cases/<slug>.md`
   - Include all findings — use cases should demonstrate real capabilities
   - One agent writes all use case pages to avoid index/page inconsistency

4. **Concepts**: Write `docs/concepts/architecture.md` and `docs/concepts/data-flow.md` (if applicable)
   - Include Agent D findings (architecture, data flow descriptions)
   - Must produce Mermaid diagrams — use flowchart TD for architecture, sequenceDiagram for data flow

5. **CLI reference**: Write `docs/reference/cli.md` (if CLI detected)
   - Include complete Agent B findings — every command, subcommand, flag, default
   - Use collapsible `???` blocks for commands with more than 5 flags

6. **API reference**: Write `docs/reference/api.md` (if REST API detected)
   - Include complete Agent C findings — every endpoint with method, path, request/response

7. **Configuration**: Write `docs/reference/configuration.md` (if config is non-trivial)
   - Include complete Agent D config findings — every key, type, default, env var

8. **Operations**: Write `docs/operations/deployment.md` and `docs/operations/monitoring.md` (if applicable)
   - Include Agent D deployment targets and integration findings

9. **FAQ + Troubleshooting**: Write `docs/faq.md` and `docs/troubleshooting.md`
   - Derive FAQ questions from the most common config/usage patterns found
   - Troubleshooting entries must use verbatim error messages from source when available

10. **Changelog + Contributing**: Write `docs/changelog.md` and `docs/contributing.md`
    - If CHANGELOG.md exists at repo root, import its content into docs/changelog.md
    - If CONTRIBUTING.md exists at repo root, link to it from docs/contributing.md rather than duplicating

Each agent writes directly to disk. Content rules for every agent:
- Use real names, flags, keys, and endpoints from the analysis — never invent
- Mark unknown content as `<!-- TODO: add <description> -->` — never fabricate facts
- At least one `!!! tip` and one `!!! warning` per page
- Add `## Related pages` section at the bottom of each page with 2–4 links

**After all agents complete**: verify every file listed in `mkdocs.yml` nav exists on disk. List any missing files and create stub pages for them rather than leaving a broken nav.

---

## Phase 5: Cross-Link Verification

Read `.ghdoc-analysis.md` to get the concept inventory. Then launch one agent:

"Read `.ghdoc-analysis.md` to get the full concept inventory for this project. Then read all markdown files in `docs/`. For each concept in the inventory (command names, config keys, API endpoints, component names, proper nouns), find every page that mentions it without a link to the page where it is defined. Add markdown links for these. Then: (1) check that every nav entry in `mkdocs.yml` has a corresponding file — report any missing; (2) check that every `[text](relative-path)` in any docs page resolves to a real file — report and fix any broken links; (3) add a `## Related pages` section to any page that currently has none. Return a summary: links added (count and pages), broken links fixed, missing nav files."

After the agent completes, show the summary to the user.

---

## Phase 6: Build Verification + GitHub Pages Setup

**Build verification** (run these commands and report results):

```bash
# Check if uv is available; use pip as fallback
if command -v uv &>/dev/null; then
  uv pip install -r requirements-docs.txt 2>&1 | tail -5
else
  pip install -r requirements-docs.txt 2>&1 | tail -5
fi
mkdocs build --strict 2>&1
```

Report any build errors with suggested fixes before showing the setup guide.

**GitHub Pages setup guide** (print this, substituting actual values detected from the repo remote):

```
## How to Enable GitHub Pages

1. Stage and commit the docs:
   git add docs/ mkdocs.yml requirements-docs.txt .github/workflows/docs.yml
   git commit -m "Add MkDocs documentation site"
   git push

2. Enable GitHub Pages:
   Go to: https://github.com/<org>/<repo>/settings/pages
   Source → GitHub Actions

3. The docs workflow runs automatically on the next push to main.
   Site will be published at: https://<org>.github.io/<repo>/

4. First deploy (optional — do locally before pushing):
   uv pip install -r requirements-docs.txt   # or: pip install -r requirements-docs.txt
   mkdocs gh-deploy --force

5. Preview locally at any time:
   mkdocs serve
   Open: http://localhost:8000

6. Add a logo (optional):
   Place logo at: docs/assets/logo.svg
   Update mkdocs.yml → theme.logo: assets/logo.svg
```

If the repo remote was not detected in Phase 3, remind the user to fill in `repo_url` and `repo_name` in `mkdocs.yml` before pushing.

Finally, delete `.ghdoc-analysis.md` (it was a working file, not part of the docs).

---

## Content Principles

Apply these to every generated page:

- Write from actual source analysis — no generic boilerplate.
- Every code example must be copy-pasteable using real command/function names.
- CLI flags must match source exactly (read from `.ghdoc-analysis.md`, not from memory).
- Config keys must match actual schemas exactly.
- Use Mermaid diagrams for any architecture with more than 2 components.
- Prefer tabs for the same operation across install methods, OS, or language variants.
- Prefer `???` collapsible blocks for exhaustive parameter lists — show the 3 most common inline.
- Mark unknown content `<!-- TODO: add <description> -->` — never invent facts.
- `!!! tip` for non-obvious best practices; `!!! warning` for known gotchas.
- Keep contributing.md short — link to repo-root CONTRIBUTING.md if it exists.
- Import changelog from repo-root CHANGELOG.md via snippet reference if it exists.
