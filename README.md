# ghdoc — MkDocs documentation site generator (Claude Code plugin)

A Claude Code plugin that generates a complete, opinionated documentation site for any GitHub
repository: structured pages written from real source analysis, Material theme, GitHub Actions CI,
and GitHub Pages deployment. Its purpose is a **unified documentation experience across all repos**
— same structure, same look, same build pipeline.

## What it generates

- `docs/` — a full page set (landing, about, getting started, use cases, concepts, reference,
  operations, FAQ, troubleshooting, changelog, contributing) tailored to what the repo actually
  contains; existing docs are read first and their knowledge preserved
- `mkdocs.yml` — pinned MkDocs 1.6.x + Material 9.7.x baseline with the org plugin set
- `requirements-docs.txt` — fully pinned dependencies
- `.github/workflows/docs.yml` — strict build on every PR, deploy on main (or on `v*` tags with
  `mike` when versioned)
- optional `just` recipes (`docs-build`, `docs-serve`)

## Install

Add the marketplace and install the plugin in Claude Code:

```
/plugin marketplace add wlame/mkdocs-skill
/plugin install ghdoc@wlame
```

## Usage

In any repository, run:

```
/ghdoc
```

Optional arguments:

| Argument | Values | Default | Purpose |
|---|---|---|---|
| `color-scheme=` | `indigo` \| `teal` \| `blue-grey` \| `deep-purple` \| `orange` | `indigo` | Material palette |
| `versioned=` | `yes` \| `no` | `no` | Multi-version docs via `mike` |
| `api=` | `yes` \| `no` | auto | Force API reference generation |
| `engine=` | `mkdocs` \| `properdocs` | `mkdocs` | Site generator engine (see below) |
| `override:` | free text | — | Instructions that override any default |

The command works in phases (analysis → structure proposal → infrastructure → content →
cross-linking → build verification) and stops for confirmation before writing any content.

## Engines: MkDocs today, ProperDocs on the watch list

The generator is a parameter, not a hardcoded choice. The org baseline is **pinned MkDocs 1.6.1 +
Material for MkDocs 9.7.6** — proven, universally documented, and reproducible. But both projects
changed hands in 2025–2026, so the succession path is tracked explicitly:

**[ProperDocs](https://properdocs.org/)** (with the [MaterialX](https://github.com/jaywhj/mkdocs-materialx)
theme) is the community continuation fork of MkDocs, launched March 2026 by the previously most
active MkDocs maintainer after upstream stalled and moved toward an incompatible, plugin-less
"MkDocs 2.0" ([background](https://fpgmaas.com/blog/collapse-of-mkdocs/)). It is a **drop-in
replacement**: same `mkdocs.yml` config, same plugin API, same 1.6.x lineage — switching is a
package/CLI/theme name swap, not a migration. Feature parity is therefore effectively 100% for
this standard's baseline profile, with two open questions we re-check periodically:

- `mike` versioning compatibility is unverified ([jimporter/mike#259](https://github.com/jimporter/mike/issues/259))
- the project is young (first release 2026-03) — watching for sustained release cadence

**We expect to switch the default engine to ProperDocs (or [Zensical](https://zensical.org/), the
official Rust-based successor from the Material team, if it reaches plugin parity first) — just
not yet.** Until then it is available per-repo as an opt-in: `/ghdoc engine=properdocs`. Concrete
migration triggers and the quarterly re-check checklist live in
[`skills/ghdoc/references/mkdocs-standard.md`](skills/ghdoc/references/mkdocs-standard.md)
("Engines and succession").

## Repository layout

```
.claude-plugin/       plugin + marketplace manifests
commands/ghdoc.md     the /ghdoc command (phased generation workflow)
skills/ghdoc/         the skill: org standard, palettes, page templates, examples
  references/         mkdocs-standard.md, unified-structure.md, color-schemes.md, page-templates.md
  examples/           mkdocs.yml, requirements-docs.txt, docs-workflow.yml, docs-versioned-workflow.yml
```

## License

MIT — see [LICENSE](LICENSE).
