# AGENTS.md — working on the ghdoc plugin

Guidance for LLM coding agents working on this repository.

## What this repo is

A Claude Code **plugin** (`ghdoc`) that generates opinionated MkDocs + Material documentation
sites for GitHub repositories, enforcing one org-wide docs standard. It contains no application
code — everything is Markdown instructions, YAML templates, and manifests.

## Core pieces and how they relate

- `commands/ghdoc.md` — the `/ghdoc` slash command: a 6-phase workflow (existing-docs pre-check →
  repo analysis via parallel Explore agents → structure proposal → infrastructure files → content
  generation → cross-link + build verification). It **reads plugin files via
  `${CLAUDE_PLUGIN_ROOT}/...`** — never as repo-relative paths, because the command executes inside
  a *target* repository, not this one. Keep every plugin-file reference prefixed.
- `skills/ghdoc/SKILL.md` — the skill Claude loads for docs-standard knowledge; holds the quick
  reference and the **Engines table** (site generator as data: package, CLI, theme per engine).
- `skills/ghdoc/references/` — the deep material: `mkdocs-standard.md` (the full org standard,
  including "Engines and succession"), `unified-structure.md` (per-page content specs),
  `color-schemes.md` (palette YAML), `page-templates.md` (fill-in templates).
- `skills/ghdoc/examples/` — copy-verbatim templates: `mkdocs.yml`, `requirements-docs.txt`
  (fully pinned — never add an unpinned line), `docs-workflow.yml` (unversioned; GitHub Pages
  artifact deploy), `docs-versioned-workflow.yml` (mike; gh-pages branch deploy).
- `.claude-plugin/plugin.json` + `marketplace.json` — manifests; keep versions in sync with
  `SKILL.md` frontmatter.

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
5. **Engine choice is data.** Anything engine-specific (package, CLI binary, theme name) must come
   from the Engines table in `SKILL.md` / `mkdocs-standard.md`, not be hardcoded in prose.

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
