# Collateral Evolution of Quarto Typst Templates

You are in a directory `tt-work` containing
* typst-templates/ a repo containing 6 typst templates for Quarto
* typst-templates-upstream/ clone of https://github.com/typst/templates (contains multiple templates)
* extra space for you to fork other repos as needed (add here)


The last significant commits were more than 2 years ago, and Typst moves fast. 5 of them are not working.

These were built by forking Typst templates and adapting them; IIUC there was no attempt to upstream changes made on this side.

Our goal is to determine what changes were made to the original repo, then find a modern repo, and migrate to it. In some cases, those may be the same repo, but I think most have moved and/or have better-maintained ("more modern") forks.

Instead of copying the contents in, we'll import from upstream and contribute any needed changes back. **The goal is zero or minimal wrapper code** - if we need a feature, we upstream it rather than wrapping.

## Migration Strategy

For each template:

### Phase 1: Analysis

1. **Find the repo.** Each template contains a README.md with a link to the original repo. Note: templates may have been renamed upstream (e.g. `fiction` → `wonderous-book`, `dept-news` → `dashing-dept-news`).

2. **Find the fork point.** Clone the repo into tt-work. Use `git log --oneline --follow -- path/to/lib.typ` and diff against candidate commits to find when Quarto forked. The template file may be named `template.typ` or `lib.typ` depending on age. Record the commit hash, date, and message in migration-status.md.

3. **Identify Quarto's changes.** Diff the fork point against Quarto's version. For each change, **trace the git history** to determine:
   - Is it an intentional feature/preference? (e.g., academic vs decorative caption style)
   - Is it dead code or accidental? (e.g., unused function with removed parameter)
   - Did Quarto inherit a bug that upstream later fixed? (Use `git show <commit>:path/to/file` to trace)
   - Is it a bug fix that should be upstreamed?

   **Important**: Don't assume differences are intentional Quarto changes. Trace the history on both sides to understand *why* they differ.

4. **Identify upstream's improvements.** Compare the fork point to modern upstream. Upstream likely has:
   - API modernization (`locate(loc => ...)` → `context`, `style(styles => ...)` → `context`)
   - Bug fixes and new features
   - Parameter renames (check for breaking changes)

5. **Evaluate each difference.** For Quarto changes that aren't in upstream:
   - Can we adopt upstream's approach instead? (preferred)
   - Should we contribute our change upstream as an option?
   - Is it truly Quarto-specific and needs a wrapper?

### Phase 2: Upstream Contributions

6. **Contribute changes upstream.** For features we need that upstream lacks:
   - Create a branch in typst-templates-upstream
   - Implement as a parameter/option (preserve upstream defaults)
   - Commit with clear message
   - PR to upstream repo

### Phase 3: Migration

7. **Rewrite typst-template.typ.** Import from `@local/...` (during development) or `@preview/...` (once upstreamed):
   - Import the upstream function directly (don't wrap it)
   - Add helper functions for parameter conversions if needed
   - Example: `#let adapt-hero-image(h) = (image: image(h.path), caption: h.caption)`

8. **Update typst-show.typ** to call the upstream function directly:
   - Use helper functions for parameter conversions
   - Set Quarto-preferred defaults inline
   - Example: `hero-image: adapt-hero-image(...), figure-caption-style: "academic"`

9. **Test.** Run `TYPST_PACKAGE_PATH=~/.local/share/typst/packages quarto render template.qmd -M keep-typ:true`

10. **Update README.md.** Change the description from "Based on..." to "Quarto format for [package-name](https://typst.app/universe/package/package-name), a Typst template by the Typst team." Remove any outdated notes (e.g., Quarto pre-release requirements).

## Technical Notes

- **@local packages**: Symlink development directory to `~/.local/share/typst/packages/local/<name>/<version>/`
- **Quarto's typst**: Needs `TYPST_PACKAGE_PATH` env var to find @local packages
- **Template renames**: All upstream templates have creative names now:
  - `fiction` → `wonderous-book`
  - `dept-news` → `dashing-dept-news`
  - `ams` → `unequivocal-ams`
  - `ieee` → `charged-ieee`
  - `letter` → `appreciated-letter`

### Bibliography handling

Templates with bibliography support need special handling to prevent Quarto from injecting a duplicate `#bibliography()` call:

1. **Add `_extension.yml` settings:**
   ```yaml
   contributes:
     formats:
       typst:
         template: template.typ          # custom template
         cite-method: natbib             # prevents duplicate bibliography
   ```

2. **Create `template.typ`** (copy from AMS) - this omits the `$biblio.typ()$` partial that would add a second bibliography.

3. **Convert `bibliography-file` → `bibliography`** in typst-show.typ:
   ```
   $if(bibliography)$
     bibliography: bibliography("$bibliography$"),
   $endif$
   ```
   Modern upstream templates take `bibliography` as content, not `bibliography-file` as a path string.

### Quarto native options

Quarto has native support for some options that templates shouldn't hardcode:
- **`papersize`**: Don't hardcode paper sizes (e.g., `paper: "us-letter"`). Users can set `papersize: us-letter` in YAML.

### .gitignore

The `.gitignore` uses `/*/template.typ` to ignore Quarto-generated files in template root directories while still tracking source `template.typ` files in `_extensions/*/` directories.

## Lessons Learned

### Quarto changes are often minimal or dead code
The fiction template had zero functional changes from upstream - just whitespace. The dept-news template had a modified `blockquote` function, but it was dead code since Markdown blockquotes use Typst's native `quote` element, not a custom function.

**Lesson**: Don't assume Quarto changes are intentional. Trace how the code is actually used.

### Upstream may have already solved the problem better
The dept-news template had a custom `blockquote(body)` function. Upstream replaced this with a proper `show quote:` rule that handles Markdown blockquotes correctly AND supports attribution. The Quarto version was actually broken (couldn't show attribution).

**Lesson**: Before wrapping or preserving Quarto changes, check if upstream's approach is actually superior.

### Quarto may have inherited bugs that upstream later fixed
The AMS template's theorem numbering differed: Quarto had sequential ("Theorem 1, 2, 3") while upstream had section-prefixed ("Theorem 2.1"). Tracing the git history revealed:
1. Original upstream had section-prefixed (correct AMS style)
2. A refactor accidentally lost it
3. Quarto forked during the "broken" period
4. Upstream later restored the correct behavior

**Lesson**: Always trace git history on both sides. A difference isn't necessarily an intentional Quarto choice - we may have just forked at an unfortunate time.

### Upstream features as options, not forks
When Quarto needed a different figure caption style, we added a `figure-caption-style` parameter upstream rather than maintaining a fork. The upstream default is preserved, and Quarto sets its preferred default in the wrapper.

**Lesson**: Contribute options upstream rather than forking. This reduces maintenance burden and benefits the ecosystem.

### Use helper functions, not wrapper functions
Instead of wrapping the entire upstream function (which duplicates all parameters), use small helper functions for specific conversions and call the upstream function directly in typst-show.typ.

**typst-template.typ** - imports + helpers only:
```typst
#import "@local/dashing-dept-news:0.1.1": newsletter, article
#let adapt-hero-image(h) = (image: image(h.path, width: 14cm), caption: h.caption)
```

**typst-show.typ** - calls upstream directly:
```typst
#show: newsletter.with(
  hero-image: adapt-hero-image(...),
  figure-caption-style: "academic",
)
```

**Lesson**: Don't wrap the whole function. Use targeted helpers and call upstream directly. This way new upstream parameters are automatically available.

### Check that parameters are actually wired up
The IEEE template had a `bibliography-file` parameter defined in `typst-template.typ`, but `typst-show.typ` never passed it! The bibliography functionality was completely broken for 2+ years. AMS (created 4 days later) correctly wired up bibliography.

**Lesson**: When analyzing templates, check that typst-show.typ actually passes all the parameters that typst-template.typ accepts. Missing wiring = broken functionality.

## Status

See [migration-status.md](migration-status.md) for current progress.