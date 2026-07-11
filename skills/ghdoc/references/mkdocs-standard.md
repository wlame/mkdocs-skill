# MkDocs Documentation Standard for GitHub / Open-Source Repositories

**Status:** Recommended baseline  
**Date checked:** 2026-07-11  
**Primary recommendation:** MkDocs 1.6.x + Material for MkDocs 9.7.x + curated plugin/profile set  
**Audience:** Engineering teams building a unified documentation experience across multiple repositories/products

---

## 1. Executive summary

For a unified documentation system across GitHub repositories and open-source style tools, the most practical MkDocs stack today is:

```text
MkDocs 1.6.x
Material for MkDocs 9.7.x
mike for versioning
mkdocstrings for API/reference docs
Git metadata plugins for ownership and freshness
redirects + tags + image/lightbox + privacy/social/offline plugins as needed
```

This combination gives the best balance of:

- mature ecosystem and community knowledge;
- polished product documentation UI;
- good search and navigation;
- Markdown-first authoring;
- GitHub integration;
- generated API/reference docs;
- versioned documentation;
- reusable organization-wide configuration;
- low adoption friction for engineering teams.

The main strategic caveat is that **both halves of the stack changed hands in 2025–2026**:

- **Material for MkDocs entered maintenance mode in November 2025.** Critical bug and security
  fixes continue (committed through ~Nov 2026), but no new features — the team is focusing on
  **Zensical**, its ground-up Rust-based successor.
- **MkDocs itself effectively forked in March 2026.** After ~2 years of upstream inactivity, the
  original author privately started an incompatible "MkDocs 2.0" that would drop plugin support;
  the resulting governance conflict led the most active maintainer (Oleh Prypin / oprypin) to launch
  **ProperDocs** — a drop-in continuation fork of MkDocs 1.6.x (same config, same plugin API,
  `pip install properdocs`). Background: https://fpgmaas.com/blog/collapse-of-mkdocs/

Therefore, the recommended strategy is:

> Use pinned MkDocs 1.6.x + Material 9.7.x now as a stable baseline, keep the generator an
> **engine parameter** (see "Engines and succession" below), and re-evaluate ProperDocs/MaterialX
> and Zensical on the quarterly migration test.

---

## Engines and succession

The standard treats the site generator as data — one row per engine, consumed by all generated
files (mkdocs.yml theme name, requirements, workflow, just recipes):

| Engine | pip packages | CLI binary | Config file | Theme name | Status (2026-07) |
|---|---|---|---|---|---|
| `mkdocs` (default) | `mkdocs==1.6.1` + `mkdocs-material==9.7.6` | `mkdocs` | `mkdocs.yml` | `material` | Baseline. Frozen upstream but pinned, proven, universally documented. |
| `properdocs` | `properdocs==1.6.7` + `mkdocs-materialx==10.1.8` | `properdocs` | `properdocs.yml` (reads `mkdocs.yml` as deprecated fallback) | `materialx` | Opt-in fallback. Drop-in fork pair; smoke-tested (build --strict passes, 2026-07-11); **mike versioning unverified** (jimporter/mike#259); young project (first release 2026-03). |
| Zensical | — | — | — | — | Watch only. Official successor from the Material team (Rust rewrite, Nov 2025); feature parity still closing; not plugin-compatible enough for the baseline profile yet. |

**Expected eventual direction:** ProperDocs + MaterialX (or Zensical, if parity lands first) will
replace the frozen baseline. Because ProperDocs is a drop-in fork, switching is a name swap in
generated files, not a content migration. Re-check on each quarterly migration test (section 21):

- ProperDocs: release cadence and issue response since 2026-03; mike compatibility resolution;
  mkdocstrings confirmation.
- MaterialX: tracking of upstream Material security fixes after Material's ~Nov 2026 fix window closes.
- Zensical: baseline plugin-profile parity (tags, redirects, git metadata, mkdocstrings, versioning).

**Migrate the default engine when:** Material's security-fix window lapses with open CVEs, OR
ProperDocs/MaterialX shows 6+ months of sustained maintenance and the mike + mkdocstrings boxes
are checked, OR Zensical reaches full baseline-profile parity.

---

## 2. Terminology in MkDocs

MkDocs usually uses these terms:

| Concept | MkDocs name | Config section | Example |
|---|---|---|---|
| Visual layout/UI | Theme | `theme:` | `material` |
| Build-time behavior | Plugin | `plugins:` | `search`, `mkdocstrings`, `mike`, `redirects` |
| Markdown syntax extension | Markdown extension | `markdown_extensions:` | `admonition`, `pymdownx.superfences` |
| Jinja/theme customization | Theme override | `theme.custom_dir` / `overrides/` | custom header/footer/templates |
| Static assets | Extra CSS/JS/assets | `extra_css`, `extra_javascript`, `docs/assets/` | organization branding |
| Arbitrary extra config | Extra data | `extra:` | version selector, analytics, social links |

Official MkDocs documentation describes the same high-level extension model: themes, plugins, and Markdown extensions.

---

## 3. Evaluation criteria

For an organization-wide documentation standard, I would evaluate themes/plugins by these criteria:

| Criterion | Why it matters |
|---|---|
| Mature ecosystem | Teams need examples, fixes, Stack Overflow/GitHub discussions, and predictable behavior. |
| Good default UX | Documentation should be useful without every team becoming frontend experts. |
| GitHub fit | Repository link, edit links, source links, issue links, GitHub Pages support, and versioning matter. |
| Search quality | Internal docs become hard to use without fast search. |
| Navigation scalability | Large docs need sections, tabs, index pages, breadcrumbs, TOC, and version selectors. |
| API/reference generation | Repositories often need generated API docs from source code. |
| Content portability | Markdown should remain mostly portable and not depend on excessive custom macros. |
| CI reproducibility | Builds must be deterministic and strict. |
| Maintenance risk | Docs platform should not become a fragile dependency. |
| Migration path | A central template must make future migration possible. |

---

## 4. Theme recommendation

### Recommended theme: Material for MkDocs

```yaml
theme:
  name: material
```

Material for MkDocs is the strongest current choice because it provides a feature-rich technical documentation UI on top of MkDocs:

- responsive design;
- polished navigation;
- local search;
- repository links;
- edit/view source actions;
- code copy buttons;
- code annotations;
- tabs;
- admonitions;
- Mermaid diagram support through Markdown extensions;
- social cards;
- privacy/offline helpers;
- tags;
- blog support;
- version selector integration.

### Why not the built-in themes?

MkDocs ships with simpler built-in themes such as `mkdocs` and `readthedocs`. They are useful for small projects, but for a unified product documentation standard they usually lack enough UX polish, navigation features, and ecosystem expectations.

### Why not switch directly to Zensical now?

Zensical is the official successor path from the Material for MkDocs team and is designed for compatibility with existing `mkdocs.yml` projects. However, for a conservative organization-wide standard, Material for MkDocs remains the safer immediate baseline because:

- most existing MkDocs plugin examples target MkDocs + Material;
- team onboarding material is easier to find;
- plugin behavior is better known;
- migration risk can be tested later using the same template.

### Theme decision

Use:

```text
Primary baseline: Material for MkDocs 9.7.x
Approved fallback engine: ProperDocs + MaterialX (opt-in via engine=properdocs)
Watch list: Zensical
Avoid for baseline: heavily custom themes, unmaintained themes, theme forks without clear maintenance history
```

---

## 5. Community feedback synthesis

The general feedback pattern from users and maintainers is:

### Positive feedback

Users generally choose Material for MkDocs because it is:

- easy to make documentation look professional without custom frontend work;
- Markdown-first;
- feature-rich compared with default MkDocs themes;
- familiar in the Python/open-source ecosystem;
- good for GitHub Pages and Read the Docs style publishing;
- powerful enough for both library docs and product docs.

### Common complaints / risks

The most common concerns are:

- **maintenance-mode risk:** Material for MkDocs is no longer the long-term innovation path;
- **MkDocs ecosystem uncertainty:** MkDocs 1.x, MkDocs 2.0 discussions, Zensical, ProperDocs, and MaterialX have created some fragmentation;
- **large navigation performance/UX:** very large flat navigation trees can become hard to manage;
- **plugin fragility:** too many plugins can make builds slower and more brittle;
- **theme override debt:** deep overrides of Material internals can make upgrades painful;
- **macro abuse:** heavy templating can make Markdown harder to read and migrate.

### Practical interpretation

Material for MkDocs is still the best **current** choice for a reusable organization-wide documentation baseline, but you should treat it like a stable LTS platform:

- pin versions;
- keep overrides minimal;
- keep Markdown portable;
- document all plugin usage;
- run strict CI builds;
- maintain a quarterly migration test against successors.

---

## 6. Recommended plugin profiles

Use multiple profiles instead of one huge configuration. Every repository should start with the baseline profile, then opt into specialized profiles.

---

### 6.1 Baseline profile: every repository

```yaml
plugins:
  - search
  - tags
  - privacy
  - git-revision-date-localized:
      enable_creation_date: true
      type: date
  - git-authors
  - redirects:
      redirect_maps: {}
  - glightbox
```

| Plugin | Purpose | Recommendation |
|---|---|---|
| `search` | Local client-side search | Enable everywhere. |
| `tags` | Tagging and discovery | Enable everywhere for large documentation estates. |
| `privacy` | Self-host external assets where possible | Enable for public/company docs. |
| `git-revision-date-localized` | Show page freshness | Enable where Git history is available. |
| `git-authors` | Show ownership/contributors | Useful for internal ownership and open-source attribution. |
| `redirects` | Preserve old URLs | Enable early, before docs grow. |
| `glightbox` | Image zoom/lightbox | Useful for diagrams, screenshots, UI docs. |

---

### 6.2 API/reference documentation profile

For Python libraries/services, add:

```yaml
plugins:
  - mkdocstrings:
      handlers:
        python:
          options:
            show_source: true
            show_root_heading: true
            show_signature_annotations: true
  - autorefs
```

For generated reference pages, add:

```yaml
plugins:
  - gen-files:
      scripts:
        - docs/scripts/gen_ref_pages.py
  - literate-nav:
      nav_file: SUMMARY.md
  - section-index
```

| Plugin | Purpose | Recommendation |
|---|---|---|
| `mkdocstrings` | Generate API docs from source/docstrings | Use for Python packages and libraries. |
| `mkdocstrings-python` | Python handler for mkdocstrings | Use with `mkdocstrings`. |
| `autorefs` | Cross-page heading/object references | Use with generated reference docs. |
| `gen-files` | Generate docs pages during build | Use when reference pages should not be committed. |
| `literate-nav` | Maintain nav in Markdown | Useful for generated API docs and large docs. |
| `section-index` | Make section index pages work naturally | Useful with `navigation.indexes`. |

---

### 6.3 Versioned product documentation profile

For products with released versions, use `mike`:

```yaml
extra:
  version:
    provider: mike
```

Recommended deployment model:

```bash
mike deploy --push --update-aliases 1.4 latest
mike set-default --push latest
```

Why use `mike`:

- old docs remain deployed as static files;
- old versions are not rebuilt every time;
- you can maintain `latest`, `stable`, or version aliases;
- Material integrates with the version selector UI.

Use versioning for:

- SDKs;
- CLI tools;
- APIs with versioned behavior;
- products with multiple supported releases.

Avoid versioning for:

- very small internal docs;
- continuously deployed services where docs should always reflect `main`;
- early projects without stable releases.

---

### 6.4 Optional content profile

```yaml
plugins:
  - blog
  - offline
  - macros
  - minify_html
  - monorepo
```

| Plugin | Purpose | Recommendation |
|---|---|---|
| `blog` | Release notes, engineering posts, announcements | Use for product changelogs or docs news. |
| `offline` | Offline-browsable docs bundle | Use for air-gapped/customer-delivered docs. |
| `macros` | Variables and build-time templating | Use carefully; avoid making Markdown unreadable. |
| `minify_html` | Smaller generated HTML | Use in CI if it does not break output. |
| `monorepo` | Combine docs from multiple packages | Use only when repo structure requires it. |

---

## 7. Recommended Markdown extensions

Use this baseline:

```yaml
markdown_extensions:
  - admonition
  - attr_list
  - md_in_html
  - footnotes
  - tables
  - toc:
      permalink: true
  - pymdownx.details
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span
  - pymdownx.inlinehilite
  - pymdownx.snippets:
      check_paths: true
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.emoji:
      emoji_index: !!python/name:material.extensions.emoji.twemoji
      emoji_generator: !!python/name:material.extensions.emoji.to_svg
```

### Authoring guidance

Use Markdown extensions for documentation clarity, not decoration.

Recommended usage:

- `admonition` for notes, warnings, tips, and danger blocks;
- `details` for collapsible advanced sections;
- `superfences` for Mermaid diagrams and nested code blocks;
- `tabbed` for OS/package-manager/language variants;
- `snippets` for shared reusable fragments;
- `tasklist` for checklists;
- `toc.permalink` for stable links to headings.

Avoid:

- overusing tabs for completely different topics;
- hiding critical information inside collapsed sections;
- complex HTML in Markdown;
- too many custom CSS classes;
- diagrams that cannot be maintained as text.

---

## 8. Recommended `mkdocs.yml` baseline

```yaml
site_name: Product Docs
site_description: Unified documentation
site_url: https://docs.example.com/

repo_name: org/repo
repo_url: https://github.com/org/repo
edit_uri: edit/main/docs/

strict: true
use_directory_urls: true

theme:
  name: material
  language: en
  logo: assets/logo.svg
  favicon: assets/favicon.svg
  features:
    - navigation.instant
    - navigation.tracking
    - navigation.tabs
    - navigation.sections
    - navigation.indexes
    - navigation.top
    - search.suggest
    - search.highlight
    - search.share
    - content.code.copy
    - content.code.annotate
    - content.action.edit
    - content.action.view
    - toc.follow

plugins:
  - search
  - tags
  - privacy
  # social cards are optional — they require mkdocs-material[imaging] plus system
  # cairo libraries in CI; enable deliberately, never by default:
  # - social:
  #     enabled: !ENV [CI, false]
  - git-revision-date-localized:
      enable_creation_date: true
      type: date
  - git-authors
  - redirects:
      redirect_maps: {}
  - glightbox

markdown_extensions:
  - admonition
  - attr_list
  - md_in_html
  - footnotes
  - tables
  - toc:
      permalink: true
  - pymdownx.details
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span
  - pymdownx.inlinehilite
  - pymdownx.snippets:
      check_paths: true
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.emoji:
      emoji_index: !!python/name:material.extensions.emoji.twemoji
      emoji_generator: !!python/name:material.extensions.emoji.to_svg

extra:
  version:
    provider: mike
```

---

## 9. Recommended dependency management

Use pinned dependencies for reproducible organization-wide builds.

### Minimal `requirements-docs.in`

```txt
mkdocs==1.6.1
mkdocs-material==9.7.6

mkdocstrings==1.0.5
mkdocstrings-python==2.0.5
mkdocs-autorefs==1.4.4
mkdocs-gen-files==0.6.1
mkdocs-literate-nav==0.6.3
mkdocs-section-index==0.3.12

mkdocs-git-revision-date-localized-plugin==1.5.3
mkdocs-git-authors-plugin==0.10.0
mkdocs-redirects==1.2.3
mkdocs-glightbox==0.5.2
mkdocs-macros-plugin==1.5.0
mike==2.2.0
```

Every entry is pinned — an unpinned line in a "pinned" file silently defeats the whole
reproducibility goal. (Versions verified against PyPI 2026-07-11.)

### Better production approach

Use `pip-tools` or another lockfile generator:

```bash
pip-compile requirements-docs.in -o requirements-docs.txt
pip-sync requirements-docs.txt
```

Then CI should install from the locked file:

```bash
pip install -r requirements-docs.txt
mkdocs build --strict
```

### Why pin versions?

For a centralized documentation platform, unpinned versions create avoidable risk:

- plugin updates can break builds;
- theme HTML/CSS changes can break overrides;
- CI output can differ between repositories;
- generated API docs can change format unexpectedly;
- migration testing becomes harder.

---

## 10. GitHub Actions baseline

Two workflow templates ship with this standard — use exactly one, chosen by the versioning model.
**The GitHub Pages "Source" setting must match the workflow**; a mismatch builds green and never
publishes:

| Model | Template | Publishing mechanism | Pages Source setting |
|---|---|---|---|
| Unversioned (default) | `examples/docs-workflow.yml` | `actions/upload-pages-artifact` + `actions/deploy-pages` | **GitHub Actions** |
| Versioned (mike) | `examples/docs-versioned-workflow.yml` | `mike deploy --push` to the `gh-pages` branch on `v*` tags | **Deploy from a branch → gh-pages** |

Shared rules for both:

- `fetch-depth: 0` on checkout — Git metadata plugins (authors, revision dates) need full history;
- install from the locked `requirements-docs.txt`;
- `mkdocs build --strict` gates every push and pull request;
- least-privilege permissions: the artifact flow needs only `pages: write` + `id-token: write` on
  the deploy job (`contents: read` elsewhere); only the mike flow needs `contents: write` because
  it pushes a branch;
- an external link check (`lychee`) runs non-blocking — internal links are already covered by
  `--strict`, and external sites break for reasons unrelated to the repo;
- do not use `mkdocs gh-deploy` in the artifact flow — it belongs to the branch-based model only.

For versioned docs, the release flow mike performs on a `v1.4.x` tag:

```bash
mike deploy --push --update-aliases 1.4 latest
mike set-default --push latest
```

---

## 11. Recommended repository layout

```text
docs/
  index.md
  getting-started.md
  concepts/
    index.md
  guides/
    index.md
  reference/
    index.md
  operations/
    index.md
  troubleshooting/
    index.md
  changelog.md
  contributing.md
  assets/
    logo.svg
    favicon.svg
    diagrams/
  snippets/
    common-warning.md
  scripts/
    gen_ref_pages.py
mkdocs.yml
requirements-docs.in
requirements-docs.txt
```

### Required sections for most repositories

| Section | Purpose |
|---|---|
| `index.md` | Landing page, project purpose, quick links. |
| `getting-started.md` | Fast path for first successful use. |
| `concepts/` | Stable conceptual model. |
| `guides/` | Task-oriented how-to content. |
| `reference/` | APIs, CLI flags, config options, schemas. |
| `operations/` | Deployment, observability, backups, SLOs, runbooks. |
| `troubleshooting/` | Known failures, diagnostics, fixes. |
| `changelog.md` | User-visible changes. |
| `contributing.md` | Contribution workflow. |

---

## 12. Documentation principles for the organization

### 12.1 Separate documentation types

Keep these content types distinct:

| Type | Question answered | Example |
|---|---|---|
| Tutorial / getting started | How do I succeed the first time? | Install and run local dev environment. |
| Concept | How does this work? | Architecture, lifecycle, data model. |
| Guide / how-to | How do I complete a task? | Configure SSO, rotate keys, create a plugin. |
| Reference | What are the exact options? | CLI flags, API methods, config schema. |
| Troubleshooting | What went wrong and how do I fix it? | Error messages, logs, known issues. |
| Operations | How do we run it safely? | Backup, restore, monitoring, deployment. |

### 12.2 Standard page structure

For guides:

```markdown
# Task title

## Goal

What the reader will accomplish.

## Prerequisites

- Required permissions
- Required tools
- Required environment

## Steps

1. Do this.
2. Do that.
3. Verify the result.

## Verification

Expected output, logs, API response, or UI state.

## Troubleshooting

Common failures and fixes.

## Related pages

- Link to concept
- Link to reference
```

For reference pages:

```markdown
# Reference title

## Overview

## Syntax / schema / API

## Parameters

## Examples

## Compatibility notes

## Related pages
```

### 12.3 README policy

For GitHub repositories:

- keep `README.md` short;
- include project purpose, quick start, status, and links;
- move long installation/configuration/API docs to the MkDocs site;
- avoid duplicating large docs between README and `docs/`.

---

## 13. Navigation strategy

Avoid one huge flat nav. Use a predictable hierarchy:

```yaml
nav:
  - Home: index.md
  - Getting started: getting-started.md
  - Concepts:
      - concepts/index.md
      - Architecture: concepts/architecture.md
      - Data model: concepts/data-model.md
  - Guides:
      - guides/index.md
      - Installation: guides/installation.md
      - Configuration: guides/configuration.md
  - Reference:
      - reference/index.md
      - CLI: reference/cli.md
      - API: reference/api.md
  - Operations:
      - operations/index.md
      - Deployment: operations/deployment.md
      - Monitoring: operations/monitoring.md
  - Troubleshooting: troubleshooting/index.md
  - Changelog: changelog.md
```

For large API reference trees, prefer generated navigation with `literate-nav` instead of manually editing a giant YAML file.

---

## 14. Branding and customization rules

Use central branding, but keep theme overrides shallow.

Recommended:

```yaml
extra_css:
  - assets/stylesheets/extra.css

extra_javascript:
  - assets/javascripts/extra.js
```

Good customizations:

- logo;
- favicon;
- color palette;
- typography if required by brand;
- custom admonition icons;
- minor spacing/layout fixes;
- organization footer links.

Avoid:

- replacing core Material templates unless necessary;
- heavy JavaScript behavior;
- deeply coupled CSS selectors from internal theme markup;
- per-repository theme hacks;
- incompatible navigation patterns between products.

Organization-wide principle:

> Put customization in the shared docs template, not in individual repositories.

---

## 15. Multi-repository strategy

There are three common models.

### Model A: One docs site per repository

Best for:

- open-source libraries;
- independent tools;
- services with separate lifecycles.

Pros:

- simple ownership;
- easy CI;
- versioning maps naturally to repo releases.

Cons:

- users need a portal to discover all products;
- cross-repo links require care.

### Model B: Central documentation portal

Best for:

- product families;
- internal platforms;
- developer portals.

Pros:

- unified discovery;
- consistent IA/navigation;
- centralized ownership.

Cons:

- harder CI;
- can become a bottleneck;
- content ownership may blur.

### Model C: Hybrid

Recommended for most organizations:

- each repository owns its docs;
- a central portal indexes products and links to repo docs;
- shared template enforces look and structure;
- optional aggregation for top-level concepts and platform docs.

---

## 16. Versioning strategy

Use versioned docs only where product behavior differs by release.

Recommended aliases:

```text
latest  -> newest release or mainline release
stable  -> recommended production release, optional
1.4     -> exact minor release
1.3     -> previous supported minor release
```

Suggested rules:

- docs for released versions should not be silently rebuilt from `main`;
- old docs should remain static;
- unsupported versions should display a warning banner;
- default version should be explicit;
- deprecation and end-of-support status should be visible.

---

## 17. Search and discoverability

Baseline:

```yaml
plugins:
  - search
  - tags
```

Recommended content practices:

- use predictable page titles;
- include common synonyms in prose;
- write meaningful headings;
- add tags to important docs;
- include error messages exactly as users see them;
- create troubleshooting pages for frequent support issues.

Example tags:

```yaml
tags:
  - installation
  - configuration
  - authentication
  - troubleshooting
  - api
  - cli
  - operations
```

---

## 18. Security, privacy, and compliance

Recommended settings:

```yaml
plugins:
  - privacy
```

Guidelines:

- avoid loading third-party scripts by default;
- self-host assets where practical;
- review analytics requirements centrally;
- do not put secrets, internal hostnames, or private credentials in docs;
- scan generated docs artifacts if publishing from private repositories;
- treat Mermaid diagrams and generated HTML as publishable source.

For public documentation, review:

- screenshots;
- sample tokens;
- internal URLs;
- architecture diagrams;
- logs;
- support contacts;
- embedded metadata in images.

---

## 19. Quality gates

Every repository should run:

```bash
mkdocs build --strict
```

Recommended checks:

- no broken internal links;
- no missing nav pages;
- no build warnings;
- generated API docs build successfully;
- Mermaid diagrams render;
- old redirects work;
- docs build from a clean checkout;
- GitHub Actions uses locked dependencies;
- docs preview is available for pull requests.

Optional additional checks:

- Markdown linting;
- Vale style linting;
- link checker for external links;
- spell checking;
- screenshot testing for heavily customized themes.

---

## 20. Governance model

For a unified organization standard, define:

### Platform owners

Responsible for:

- shared template;
- dependency pins;
- theme customizations;
- CI workflow;
- migration testing;
- documentation principles;
- plugin allowlist.

### Repository owners

Responsible for:

- content accuracy;
- product-specific docs;
- release/version updates;
- redirects after page moves;
- generated API docs configuration.

### Docs review checklist

Before merging docs changes:

- Does this belong in tutorial, guide, concept, reference, operations, or troubleshooting?
- Is the page linked from navigation?
- Is the title searchable?
- Are commands copy-pasteable?
- Are examples realistic?
- Are warnings explicit?
- Are version-specific statements marked?
- Are internal/private details removed?
- Does `mkdocs build --strict` pass?

---

## 21. Migration and risk strategy

### Current risk

Material for MkDocs is in maintenance mode. MkDocs ecosystem discussions also indicate potential future changes that may not be backward-compatible with existing plugin/theme assumptions.

### Recommended mitigation

Use a migration-ready template:

- keep Markdown portable;
- avoid deep theme overrides;
- centralize CSS/JS;
- maintain a plugin inventory;
- pin dependencies;
- test quarterly against Zensical and ProperDocs/MaterialX;
- document which Material features are required by your organization.

### Quarterly migration test

Run the shared template against:

```text
1. Current pinned MkDocs + Material baseline
2. Zensical compatibility build
3. ProperDocs + MaterialX compatibility build
```

Migration should be considered only when these work:

- theme rendering;
- navigation;
- search;
- tags;
- redirects;
- version selector;
- API docs generation;
- Mermaid diagrams;
- social/offline/privacy functionality if required;
- custom CSS and branding;
- CI build and deployment.

### Migration trigger

Migrate when one of these becomes true:

- Material security support is no longer sufficient;
- Zensical reaches feature parity for your required plugin/profile set;
- MkDocs 1.x ecosystem becomes too risky;
- ProperDocs/MaterialX proves more stable for your required compatibility profile;
- organizational requirements need features not available in the Material baseline.

---

## 22. Alternatives considered

| Alternative | Fit | Notes |
|---|---:|---|
| MkDocs built-in themes | Low/Medium | Good for simple docs, not enough for a polished org-wide product docs standard. |
| Read the Docs theme | Medium | Familiar, but less feature-rich and less modern than Material. |
| Sphinx | Medium/High | Excellent for Python and complex reference docs; heavier authoring model than MkDocs. |
| Docusaurus | Medium/High | Strong React ecosystem and versioning; less Markdown-simple and less Python-native than MkDocs. |
| VitePress / VuePress | Medium | Good modern frontend stack; less aligned with existing MkDocs/Python tooling. |
| Docsify | Low/Medium | Simple runtime docs, but weaker for strict CI/static build governance. |
| Zensical | Watch | Official successor from the Material team (Rust rewrite, Nov 2025); feature parity with the baseline plugin profile still closing. |
| ProperDocs + MaterialX | Approved fallback engine | Drop-in continuation fork of MkDocs 1.6.x + Material (Mar 2026); available today via `engine=properdocs`; mike versioning unverified. See "Engines and succession". |

---

## 23. Final recommended standard

Use this as the organization-wide standard:

```text
Theme:
  Material for MkDocs 9.7.x

Core engine:
  MkDocs 1.6.x

Baseline plugins:
  search
  tags
  privacy
  git-revision-date-localized
  git-authors
  redirects
  glightbox

API docs profile:
  mkdocstrings
  mkdocstrings-python
  autorefs
  gen-files
  literate-nav
  section-index

Versioning:
  mike

Optional:
  blog
  offline
  macros
  minify_html
  monorepo

Build policy:
  mkdocs build --strict
  locked dependencies
  full Git history in CI
  shallow theme overrides only
  quarterly migration tests
```

---

## 24. Implementation checklist

### Phase 1: Template

- [ ] Create central docs template repository.
- [ ] Add baseline `mkdocs.yml`.
- [ ] Add locked `requirements-docs.txt`.
- [ ] Add GitHub Actions workflow.
- [ ] Add logo/favicon/assets.
- [ ] Add example pages for each content type.
- [ ] Add docs authoring guide.
- [ ] Add redirect policy.
- [ ] Add versioning policy.

### Phase 2: Pilot

- [ ] Apply template to 2-3 representative repositories.
- [ ] Include one small CLI/library.
- [ ] Include one service/product.
- [ ] Include one repository with generated API docs.
- [ ] Test local dev, CI, publishing, and pull-request previews.
- [ ] Collect author feedback.

### Phase 3: Organization rollout

- [ ] Publish standard and migration guide.
- [ ] Add docs build to all repositories.
- [ ] Enforce `mkdocs build --strict`.
- [ ] Add owners for each docs area.
- [ ] Add docs quality checklist to PR template.
- [ ] Create central product documentation portal/index.

### Phase 4: Maintenance

- [ ] Review plugin updates monthly or quarterly.
- [ ] Refresh pinned dependencies intentionally.
- [ ] Run migration compatibility tests quarterly.
- [ ] Track Zensical and ProperDocs/MaterialX maturity.
- [ ] Keep documentation template changes backward-compatible.

---

## 25. References

- MkDocs official site: https://www.mkdocs.org/
- MkDocs configuration guide: https://www.mkdocs.org/user-guide/configuration/
- MkDocs theme guide: https://www.mkdocs.org/user-guide/choosing-your-theme/
- MkDocs customization guide: https://www.mkdocs.org/user-guide/customizing-your-theme/
- Material for MkDocs: https://squidfunk.github.io/mkdocs-material/
- Material for MkDocs Python Markdown extensions: https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown-extensions/
- Material for MkDocs versioning with mike: https://squidfunk.github.io/mkdocs-material/setup/setting-up-versioning/
- Material for MkDocs maintenance-mode issue: https://github.com/squidfunk/mkdocs-material/issues/8523
- Material for MkDocs 2025 blog archive / maintenance-mode note: https://squidfunk.github.io/mkdocs-material/blog/archive/2025/
- Zensical compatibility: https://zensical.org/compatibility/
- Zensical plugin compatibility: https://zensical.org/compatibility/plugins/
- mike: https://github.com/jimporter/mike
- mkdocstrings: https://mkdocstrings.github.io/
- mkdocstrings recipes: https://mkdocstrings.github.io/recipes/
- mkdocstrings Python handler on PyPI: https://pypi.org/project/mkdocstrings-python/
- mkdocs-autorefs on PyPI: https://pypi.org/project/mkdocs-autorefs/
- mkdocs-gen-files on PyPI: https://pypi.org/project/mkdocs-gen-files/
- mkdocs-git-revision-date-localized-plugin on PyPI: https://pypi.org/project/mkdocs-git-revision-date-localized-plugin/
- mkdocs-redirects: https://github.com/mkdocs/mkdocs-redirects
- mkdocs-glightbox: https://github.com/blueswen/mkdocs-glightbox
- mkdocs-macros-plugin: https://mkdocs-macros-plugin.readthedocs.io/
- mkdocs-monorepo-plugin: https://github.com/backstage/mkdocs-monorepo-plugin
- MaterialX: https://github.com/jaywhj/mkdocs-materialx
- ProperDocs: https://properdocs.org/ · https://github.com/ProperDocs/properdocs
- ProperDocs organization discussions: https://github.com/orgs/ProperDocs/discussions
- ProperDocs fork background: https://fpgmaas.com/blog/collapse-of-mkdocs/
- mike + ProperDocs compatibility (open): https://github.com/jimporter/mike/issues/259
