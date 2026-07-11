---
description: Update an existing ghdoc documentation site after code changes — targeted page refresh and standard-drift fixes, preserving existing prose
argument-hint: "[structure-only=yes] [content-only=yes] [override: <free text instructions>]"
---

# ghdoc-update — Documentation Maintenance

Refresh this repository's existing documentation site: bring pages back in sync with the code and
fix drift from the organization standard — **touching only what is stale**. This is the
maintenance counterpart to `/ghdoc` (full generation): existing prose is preserved, facts are
updated in place.

User arguments: $ARGUMENTS

Parse `$ARGUMENTS`:
- `structure-only=yes` — apply only structural/standard fixes (pins, workflow, config, marker);
  skip content re-analysis entirely
- `content-only=yes` — refresh only page content; skip structural fixes
- `override: <text>` — free-form instructions (e.g. "only update the CLI reference")

## Phase 0: Preconditions

Find `mkdocs.yml` / `properdocs.yml`. **If none exists, stop** — there is nothing to update;
recommend `/ghdoc`.

Parse the state marker (format in `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/SKILL.md`, "State Marker")
to recover engine, color scheme, and versioning mode. If the marker is missing, infer the values
(theme name → engine per the Engines table; workflow contents → versioning; palette → scheme) and
tell the user you will stamp a marker as part of the update.

## Phase 1: Drift Detection

Run two things in parallel:

**1a. Structural audit** — execute the checklist in
`${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/references/audit-checklist.md` (skip the K4 build check here;
the build runs at the end anyway). Collect all errors and warnings. *(Skipped if `content-only=yes`.)*

**1b. Content re-analysis** — execute
`${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/references/repo-analysis.md` to produce a fresh
`.claude/ghdoc-analysis.md`. *(Skipped if `structure-only=yes`.)*

Then, if content is in scope, launch one Explore agent to **diff the fresh analysis against the
current docs**:

"Read `.claude/ghdoc-analysis.md` (current state of the code) and every markdown file under
`docs/`. Report, per docs page: (1) **stale facts** — flags, defaults, config keys, endpoints,
install commands documented differently than the analysis shows; (2) **missing coverage** — new
commands/keys/endpoints in the analysis absent from any page; (3) **dead coverage** — documented
things that no longer exist in the analysis; (4) pages with no findings (leave-alone list). Also
check: does `changelog.md` still match the repo CHANGELOG; do conditional pages required by the
structure table's 'When' column exist for what the code now contains. Return a structured
page-by-page report."

## Phase 2: Update Plan (checkpoint)

Present one consolidated plan:

- **Structural fixes** (from 1a): each audit finding with its concrete fix — e.g. "pins drifted:
  mkdocs-material 9.7.2 → 9.7.6", "workflow uses gh-deploy with Pages source = GitHub Actions →
  replace with artifact workflow", "stamp missing state marker".
- **Page updates** (from 1b): each stale page with a one-line summary of what changes; new pages
  to add; pages proposed for removal (removal always requires explicit confirmation).
- **Leave-alone list**: pages verified current — these will not be touched.

**STOP and wait for user confirmation.** Honor exclusions the user requests.

## Phase 3: Apply

- **Structural fixes**: apply directly from the canonical templates in
  `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/examples/` (requirements pins, workflow file, config keys).
  Never regenerate `mkdocs.yml` wholesale — patch the specific keys.
- **Page updates**: one agent per page (or small group). Each agent prompt must include the
  relevant analysis section, the diff findings for that page, and this merge rule: **start from
  the existing page, change only what the findings require, preserve all surrounding prose and
  any hand-written content**. Follow the Content Writing Rules section of
  `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/SKILL.md`.
- **New pages**: use the matching template from
  `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/references/page-templates.md` and add them to `nav`.
- Update any `redirects` mapping if a page moved or was removed.

## Phase 4: Verify

1. **Fact-check the changed pages** per
   `${CLAUDE_PLUGIN_ROOT}/skills/ghdoc/references/fact-check.md` (only pages touched in Phase 3).
2. **Cross-link check** on changed pages: links resolve, nav matches files.
3. **Strict build**:
   `uv run --no-project --with-requirements=requirements-docs.txt <engine CLI> build --strict`
   (pip fallback as in `/ghdoc` Phase 6).
4. **Refresh the state marker**: set `version=` to the current skill version and `updated=` to
   today, preserving the other values.

## Phase 5: Report

Summarize: structural fixes applied, pages updated (with the key fact changes), pages added or
removed, fact-check results, build status, marker state. Then delete `.claude/ghdoc-analysis.md`.
Do not commit anything — leave the changes for the user to review.
