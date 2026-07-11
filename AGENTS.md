# AGENTS.md — working on the ghdoc plugin

Guidance for LLM coding agents working on this repository.

## What this repo is

A Claude Code **plugin** (`ghdoc`) that generates opinionated MkDocs + Material documentation
sites for GitHub repositories, enforcing one org-wide docs standard. It contains no application
code — everything is Markdown instructions, YAML templates, and manifests.

## Core pieces and how they relate

- `commands/ghdoc.md` — `/ghdoc`: phased full-site generation (prior-generation pre-check →
  existing-docs pre-check → repo analysis → structure proposal checkpoint → infrastructure →
  content generation → cross-link + fact-check → build verification). Commands **read plugin
  files via `${CLAUDE_PLUGIN_ROOT}/...`** — never as repo-relative paths, because they execute
  inside a *target* repository, not this one. Keep every plugin-file reference prefixed.
- `commands/ghdoc-update.md` — `/ghdoc-update`: maintenance. Drift detection (audit checklist +
  fresh analysis diffed against current docs) → confirmed update plan → targeted patches
  preserving prose → fact-check → build → marker refresh.
- `commands/ghdoc-audit.md` — `/ghdoc-audit`: strictly read-only compliance report driven by the
  audit checklist; never writes.
- `skills/ghdoc/SKILL.md` — the skill Claude loads for docs-standard knowledge; owns the
  operative tables (see ownership map below).
- `skills/ghdoc/references/` — deep material and shared procedures (see ownership map).
- `skills/ghdoc/examples/` — copy-verbatim templates (see ownership map).
- `.claude-plugin/plugin.json` + `marketplace.json` — manifests; keep versions in sync with
  `SKILL.md` frontmatter.

## Ownership map (single source of truth per fact)

Every operative fact is owned by exactly one file; everything else must point, never copy.
When editing, change the owner and fix pointers — never re-state a fact in a second file:

| Fact | Owner |
|---|---|
| Engines table (packages, CLI, config filename, theme) | `SKILL.md` |
| Unified page structure + inclusion conditions | `SKILL.md` |
| Content writing rules | `SKILL.md` |
| State marker format | `SKILL.md` |
| Dependency pins | `examples/requirements-docs.txt` |
| CI workflows (both Pages modes) | `examples/docs-workflow.yml`, `examples/docs-versioned-workflow.yml` |
| Config feature flags / extensions baseline | `examples/mkdocs.yml` |
| Palette YAML per scheme | `references/color-schemes.md` |
| Per-page content specs | `references/unified-structure.md` |
| Page markdown templates | `references/page-templates.md` |
| Repo-analysis agent prompts + analysis file format | `references/repo-analysis.md` |
| Fact-check procedure | `references/fact-check.md` |
| Audit checks | `references/audit-checklist.md` |
| Standard rationale, governance, succession narrative | `references/mkdocs-standard.md` |

## Key invariants (easy to get wrong)

1. **Pages mode must match the workflow.** Unversioned = artifact deploy = Pages Source "GitHub
   Actions". Versioned (mike) = gh-pages branch = Pages Source "Deploy from a branch". Mixing them
   builds green and never publishes. Setup guides in `commands/ghdoc.md` Phase 6 come in two
   variants for exactly this reason.
2. **`requirements-docs.txt` is fully pinned.** Bump versions deliberately (verify against PyPI)
   and update both `examples/requirements-docs.txt` and section 9 of `mkdocs-standard.md`.
3. **The `social` plugin stays off by default** — it needs `mkdocs-material[imaging]` plus system
   cairo libs; enabling it without them breaks CI.
4. **Generated working files go to `.claude/ghdoc-analysis.md`** in target repos (gitignored),
   never the repo root.
5. **Engine choice is data.** Anything engine-specific (package, CLI binary, config filename,
   theme name) must come from the Engines table in `SKILL.md`, not be hardcoded in prose.
6. **The state marker is the contract between commands.** `/ghdoc` writes it (first line of the
   site config), `/ghdoc-update` reads and refreshes it, `/ghdoc-audit` checks it. Changing its
   format means updating all three plus the SKILL.md definition.
7. **`/ghdoc-audit` never writes.** Any fix, however trivial, belongs to `/ghdoc-update`.
8. **Content agents must not fabricate.** Tips/warnings only where genuine; FAQ/troubleshooting
   only from real sources — the Content Writing Rules in `SKILL.md` are explicit about this.

## Succession watch (check when touching the standard)

The default engine is pinned MkDocs 1.6.1 + Material 9.7.6 — both effectively frozen upstream.
**ProperDocs + MaterialX** (drop-in continuation forks, March 2026) is the approved opt-in fallback
(`engine=properdocs`) and the expected eventual default; **Zensical** is the watch-list successor.
Before recommending a switch, re-verify: mike compatibility
([jimporter/mike#259](https://github.com/jimporter/mike/issues/259)), ProperDocs release cadence,
MaterialX security-fix tracking, Zensical plugin parity. Full triggers: "Engines and succession"
in `skills/ghdoc/references/mkdocs-standard.md`.

## Verifying changes

There is no build or test suite. Sanity checks:

- YAML templates parse: `python3 -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]))" <file>`
  (note: `examples/mkdocs.yml` needs `yaml.load` with a custom loader due to `!!python/name:` tags —
  or just validate with `mkdocs build` in a scratch repo).
- End-to-end: install the plugin locally, run `/ghdoc` in a scratch repository, and confirm the
  generated site builds with `mkdocs build --strict` and deploys per the printed guide.
