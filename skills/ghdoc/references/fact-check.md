# Reference Fact-Check Procedure

Verifies that documented claims match the source code. Generated reference pages are the highest
hallucination risk in the pipeline: content agents write from a summarized analysis file, so
flags, defaults, and endpoints can drift from what the code actually says. This procedure is the
adversarial pass that catches it. Used by `/ghdoc` (after content generation) and `/ghdoc-update`
(after applying page updates).

## Scope

Fact-check every page that makes verifiable claims about the code — at minimum the pages that
exist under `reference/`, plus `getting-started.md` (install command, first-run example). Prose
pages (about, use-case narratives) are checked only for their code blocks.

## Claim types to verify

| Claim type | Verify against |
|---|---|
| CLI command / subcommand name | The CLI framework registration in source (entry points, command decorators, `cmd/` tree) |
| CLI flag: name, short form, type, default | The flag definition in source — exact spelling, exact default |
| Config key: name, type, default, env var | The config schema/settings model in source |
| API endpoint: method, path, status codes | The route definition in source |
| Request/response body fields | The schema/model definition in source |
| Install command / package name | The package manifest (pyproject.toml, package.json, go.mod, …) |
| Code examples | Must reference only functions/classes/commands that exist in source |
| Verbatim error messages (troubleshooting) | The literal string in source |
| Exit codes | The actual exit paths in source |

## Procedure (one agent per page or page group)

For each page in scope, launch an agent with this task:

1. Read the page and extract **every** verifiable claim of the types above into a checklist.
2. For each claim, locate the defining source and compare exactly. Do not trust
   `.claude/ghdoc-analysis.md` for this — it is the same summarization the content agents used;
   go to the actual source files.
3. Classify each claim: `confirmed` | `mismatch` (doc says X, source says Y) | `unverifiable`
   (source not found).
4. **Fix every mismatch in place**, taking the source as truth.
5. Replace every unverifiable claim with `<!-- TODO: verify <claim> -->` rather than leaving a
   possibly-invented fact in the docs.
6. Return a summary table: claims checked, confirmed, fixed (with before → after), marked TODO.

## Reporting

Aggregate the per-page summaries and show the user: total claims checked, mismatches fixed
(listed individually — these are the bugs that would have shipped), and TODOs added. A run with
zero checked claims on a page that documents a CLI or API is itself a red flag — report it, don't
skip silently.
