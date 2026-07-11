---
description: Audit this repository's documentation site against the organization standard — read-only drift report, no changes
argument-hint: "[skip-build=yes] [override: <free text instructions>]"
---

# ghdoc-audit — Documentation Compliance Audit

Check this repository's documentation against the organization standard and report drift.
**Strictly read-only**: never modify, create, or delete any file in this run. Fixes are applied by
`/ghdoc-update` (or `/ghdoc` for full regeneration).

User arguments: $ARGUMENTS

Parse `$ARGUMENTS`:
- `skip-build=yes` — skip the K4 strict-build check (useful when dependencies can't be installed
  in this environment)
- `override: <text>` — free-form instructions (e.g. "only check CI", "skip content checks")

## Procedure

1. **Load the standard** — read these plugin files first:
   - `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/references/audit-checklist.md` — the checks to run (the
     checklist is the single source of truth; do not invent additional checks or skip listed ones)
   - `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/SKILL.md` — Engines table, Unified Page Structure table,
     State Marker format
   - `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/examples/requirements-docs.txt` — current canonical pins
   - `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/examples/docs-workflow.yml` and
     `docs-versioned-workflow.yml` — workflow baselines

2. **Detect the site** — find `mkdocs.yml` / `properdocs.yml`, parse the state marker if present,
   and determine the engine and versioning mode (from the marker, falling back to inference from
   the theme name and workflow contents).

3. **Run every check in the checklist**, section by section. Most checks are deterministic
   file reads and greps — run them directly. For S7 (required pages) apply the SKILL.md structure
   table's "When" conditions to what the repo actually contains (a CLI tool without
   `reference/cli.md` is a finding; a library without one is not). For K4, run the strict build
   with the engine CLI from the Engines table unless `skip-build=yes`.

4. **Report** exactly in the checklist's report format: a table of failures only (ID, severity,
   finding, remediation), then the one-line verdict. If fully compliant, say so in one line —
   do not pad the report.

5. **Recommend next steps**: if any finding is fixable by `/ghdoc-update`, end with the single
   command the user should run next. Do not offer to fix anything yourself in this run.

## Notes

- A missing state marker (S3) on an otherwise standard-compliant site is a warning, not an error —
  recommend `/ghdoc-update`, which stamps one.
- If there is no docs site at all (S1 fails), stop after the structural section and recommend
  `/ghdoc` instead of continuing down the checklist.
- This command is safe to run across many repositories in bulk — the one-line verdict is designed
  to be collected into an org-wide drift overview.
