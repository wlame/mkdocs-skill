# Docs Compliance Audit Checklist

The data behind `/ghdoc-audit`. Each row is one check: run the verification, classify the result
by the listed severity on failure. The audit is **strictly read-only** — it reports drift, it
never fixes anything (that is `/ghdoc-update`'s job).

Severities: **error** = docs are broken or will fail to build/publish; **warn** = standard
violation, works but drifts; **info** = worth knowing, no action required.

## Structural checks

| ID | Severity | Check | How to verify |
|---|---|---|---|
| S1 | error | Site config exists | `mkdocs.yml` (or `properdocs.yml`) present at repo root |
| S2 | error | Config parses as YAML | Load it (tolerate `!!python/name:` and `!ENV` tags) |
| S3 | warn | State marker present and well-formed | First line matches `^# ghdoc: ` with `version= engine= color-scheme= versioned= updated=` pairs (see SKILL.md "State Marker") |
| S4 | warn | Marker skill version is current | Compare marker `version=` to this skill's version in SKILL.md frontmatter |
| S5 | error | Every nav entry resolves to a file | Each path in `nav:` exists under `docs/` |
| S6 | warn | No orphan pages | Every `docs/**/*.md` is reachable from `nav:` (snippets/ excluded) |
| S7 | warn | Required pages present | Apply the "When" column of the SKILL.md Unified Page Structure table to what the repo contains |
| S8 | warn | `.claude/` is gitignored | Entry present in `.gitignore` |

## Configuration checks

| ID | Severity | Check | How to verify |
|---|---|---|---|
| C1 | error | `strict: true` set | Key present in site config |
| C2 | warn | Baseline plugins enabled | `search`, `tags`, `privacy`, `git-revision-date-localized`, `git-authors`, `redirects`, `glightbox` all in `plugins:` |
| C3 | error | `social` plugin has its dependencies | If `social` is enabled: `mkdocs-material[imaging]` in requirements AND cairo system packages installed in the workflow |
| C4 | warn | Org fonts set | `theme.font` is Inter / JetBrains Mono |
| C5 | warn | Palette is an approved scheme | `theme.palette` matches one of the schemes in `references/color-schemes.md` |
| C6 | warn | Repo linkage set | `repo_url`, `repo_name`, `edit_uri` present and match `git remote get-url origin` |
| C7 | warn | Engine consistency | Theme name matches the engine per the SKILL.md Engines table (`material` ↔ mkdocs, `materialx` ↔ properdocs); config filename matches too |

## Dependency checks

| ID | Severity | Check | How to verify |
|---|---|---|---|
| D1 | error | `requirements-docs.txt` exists | File at repo root |
| D2 | error | Every line is pinned | All entries use `==` |
| D3 | warn | Pins match the current standard | Compare against `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/examples/requirements-docs.txt` — report each divergence with both versions |

## CI / publishing checks

| ID | Severity | Check | How to verify |
|---|---|---|---|
| W1 | error | Docs workflow exists | `.github/workflows/docs.yml` present |
| W2 | error | Publishing modes not mixed | Exactly one of: Pages-artifact deploy (`actions/deploy-pages`) or mike branch deploy — never `mkdocs gh-deploy` alongside artifact actions |
| W3 | error | Workflow mode matches marker | `versioned=yes` marker ⇒ mike workflow; `versioned=no` ⇒ artifact workflow |
| W4 | warn | Full git history fetched | `fetch-depth: 0` on checkout (required by git metadata plugins) |
| W5 | warn | Strict build gates PRs | `build --strict` runs on `pull_request` |
| W6 | warn | Least-privilege permissions | Artifact flow: `contents: read` + `pages: write`/`id-token: write` on deploy only; mike flow: `contents: write` |
| W7 | info | External link check present | lychee (or equivalent) step exists, non-blocking |

## Content spot-checks (sampled, not exhaustive)

| ID | Severity | Check | How to verify |
|---|---|---|---|
| K1 | error | No template placeholders leaked | Grep docs for `<PLACEHOLDER`, `<PROJECT NAME>`, `<org>/<repo>`, `<slug>` |
| K2 | info | TODO inventory | Count `<!-- TODO:` markers, list pages |
| K3 | warn | Related-pages sections present | Pages listed in nav have a `## Related pages` section (index.md exempt) |
| K4 | error | Build passes | `uv run --no-project --with-requirements=requirements-docs.txt <engine CLI> build --strict` |

## Report format

Output one table of failures only (ID, severity, finding, remediation), then a one-line verdict:

```
ghdoc audit: <N> errors, <M> warnings, <K> info — <compliant | drifted | broken>
```

`broken` if any error, `drifted` if warnings only, `compliant` if clean. For every failure, the
remediation column names the fix and whether `/ghdoc-update` can apply it.
