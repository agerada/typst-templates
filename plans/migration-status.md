# Migration Status

Progress report for migrating Quarto Typst templates to upstream packages.

## Templates

| Template | Status | Upstream Package | Notes |
|----------|--------|------------------|-------|
| fiction | ✅ Done | @preview/wonderous-book:0.1.2 | No wrapper needed |
| dept-news | 🔄 Pending PR | @local/dashing-dept-news:0.1.1 | Helper function for hero-image conversion |
| ieee | ✅ Done | @preview/charged-ieee:0.1.4 | Pure import, added missing bibliography + template.typ |
| ams | ✅ Done | @preview/unequivocal-ams:0.1.2 | Pure import, no helpers needed |
| letter | ✅ Done | @preview/appreciated-letter:0.1.0 | Pure import, paper size now via Quarto's `papersize` |
| poster | ✅ Done | @local/typst-poster:0.1.1 | Pure import, API modernization applied |

## Completed

### fiction → @preview/wonderous-book:0.1.2
- **Fork point**: `7cec678` (2023-04-26) "fix: update syntax to new query selector (#6)"
- **Quarto changes**: None (only whitespace)
- **Migration**: Simple import, no wrapper needed
- **Breaking change**: `paper` parameter renamed to `paper-size` in upstream

### dept-news → @local/dashing-dept-news:0.1.1
- **Fork point**: `81fad3d` (2023-03-27) "Templates" (initial commit)
- **Quarto changes**:
  - Figure caption style: decorative `[-- caption --]` → academic `Figure 1: caption` (intentional preference)
  - `blockquote` function: removed `by` parameter (dead code - Markdown blockquotes use native `quote` element)
- **Upstream improvements**:
  - API modernization (`style`/`locate` → `context`)
  - `blockquote` replaced with proper `show quote:` rule (supports attribution)
  - hero-image takes `image` content instead of `path` string
- **Upstream contribution**: Added `figure-caption-style` parameter (branch: `quarto-figure-caption-option`)
- **Migration**: Helper function `adapt-hero-image` for path→image conversion; calls upstream `newsletter` directly
- **TODO**: PR to typst/templates, then switch to @preview

### ams → @preview/unequivocal-ams:0.1.2
- **Fork point**: `a2123e3` (2023-04-24) "Update template.typ (#5)"
- **Quarto changes**: None intentional - all differences were inherited bugs or missed improvements
- **History analysis**:
  - Theorem numbering: Quarto had sequential `"1"`, but original upstream had section-prefixed. A refactor accidentally broke it, Quarto forked during broken period, upstream later fixed it.
  - Bibliography style: Quarto had `apa`, upstream intentionally changed to `springer-mathphys` (more appropriate for math).
- **Upstream improvements adopted**:
  - API modernization (`locate` → `context`)
  - Correct theorem numbering (section-prefixed)
  - Appropriate bibliography style (`springer-mathphys`)
  - Better figure/proof handling
- **Migration**: Pure import, `bibliography: bibliography("$bibliography$")` inline conversion in typst-show.typ

### ieee → @preview/charged-ieee:0.1.4
- **Fork point**: `81fad3d` (2023-03-27) "Templates" (initial commit)
- **Quarto changes**: Only typo fix ("metdata" → "metadata")
- **Upstream improvements adopted**:
  - API modernization (`locate` → `context`)
  - Font: "STIX Two Text" → "TeX Gyre Termes" (matches IEEE LaTeX)
  - `bibliography-file` → `bibliography` (content parameter)
  - New `figure-supplement` parameter
  - Better figure/table handling, multi-column layout
  - Equation reference formatting
- **Bug fixes during migration**:
  - Added missing `bibliography` parameter to typst-show.typ (was never wired up)
  - Added `template.typ` and `cite-method: natbib` to prevent duplicate bibliography
- **Migration**: Pure import, inline bibliography conversion

### letter → @preview/appreciated-letter:0.1.0
- **Fork point**: `81fad3d` (2023-03-27) "Templates" (initial commit)
- **Quarto changes**: Added `paper: "us-letter"` hardcode
- **Upstream changes**: None (template unchanged since initial commit)
- **Breaking change**: Paper size no longer hardcoded to US Letter; uses Typst default (A4). Users wanting US Letter should add `papersize: us-letter` to YAML.
- **Migration**: Pure import, no helpers needed

### poster → @local/typst-poster:0.1.1
- **Fork point**: `ff54bcf` (2023-04-13) "Add support for keywords #5"
- **Quarto changes**: None (identical to upstream)
- **Upstream status**: Original repo (pncnmnp/typst-poster) unmaintained, not published to @preview
- **Applied fixes**:
  - Cherry-picked PR #6 from oluceps: `locate` → `context` API modernization
  - Fixed deprecated `show par: set block(spacing:)` → `set par(spacing:)`
  - Added `typst.toml` package manifest
- **Migration**: Pure import, no helpers needed
- **TODO**: Consider publishing to @preview or finding maintainer

## Pending PRs

- [ ] typst/templates: `figure-caption-style` parameter for dashing-dept-news (branch: `quarto-figure-caption-option` in typst-templates-upstream)
