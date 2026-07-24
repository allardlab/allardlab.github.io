# Website Improvement TODO

Phase 1 (refactor) and Phase 2 (UI modernization) are both done on the `refactor-and-ui-overhaul` branch. The items below are the visible-change items that did not make it into Phase 2 — they are deliberately scoped out because each is a meaningful feature on its own. The Foundation-vs-CSS-Grid question is resolved: the site is on CSS Grid + Flexbox, no framework.

## Home page (`index.md`)

- Responsive image handling on `PXL_20230221_052352091~2.jpg` (`srcset`, `sizes`, aspect ratio, `object-fit`).
- Add a clear value-proposition line above the fold and CTAs to key pages (Publications, People, open positions).
- Visual hierarchy and visible separators between sections.

## Publications page (`publications.md`)

- Migrate the hand-maintained `<ul class="publist">` into structured `_data/publications.yml`.
- Group entries by year with sticky year headers.
- Style preprints vs. journal papers distinctly.
- Client-side year/keyword filter.
- Highlight recent / featured publications above the fold.

## People page (`people.md`)

- Migrate the hand-maintained list to `_data/people.yml` with `current`, `photo`, `bio`, `links`.
- Real `alt` text on every photo; styled initials avatar when no photo exists (drop favicon placeholders).
- Hover state on people cards.

## Software page (`software.md`)

- Replace the brittle `cdn.jsdelivr.net/github-cards` widget with static cards rendered from `_data/software.yml` (or built at deploy time from the GitHub API). Restyle to match the dark theme.

## Science page (`science.md`)

- ✅ Added an `<h1>` ("Research", via `title:` front matter).
- ✅ Replaced the verbose justified filler with an alternating "zigzag" layout: each paper's Bluesky card pairs with a short research blurb (`.science-row` / `.science-row--flip` in `_08_pages.scss`).
- ✅ Deduped the Bluesky `embed.js` to a single `async` `<script>` at the end of the page (was one per card).
- The three blurbs (`Reading the immune signal`, `Building the cell's skeleton`, `The physics of DNA`) are first-draft copy — have Jun review/replace, especially the DNA one (written from the paper's venue, not its abstract).
- Optional: truly lazy-load the embeds with an IntersectionObserver so `embed.js` only fetches when a card scrolls into view.
- When the Publications data migration lands, consider sourcing these cards from a `_data/*.yml` "recent papers" list instead of hand-pasting blockquotes.

## Cross-page

- Mobile-first audit at 375 / 768 / 1024 / 1440 widths in a real browser.
- Image optimisation: convert large jpg/png to webp with jpg fallback via `<picture>`. `loading="lazy"` is already on all gallery + content `<img>`s.
- Lighthouse pass: Accessibility ≥ 95, Performance ≥ 85 on simulated mobile.
- ~~Fix the pre-existing Sass `/` division deprecation warnings in `_sass/_functions.scss` (replace `/` with `math.div`).~~ **DO NOT DO THIS while production uses the `github-pages` gem.** Attempted 2026-07-24 and it broke the live build: production's Ruby Sass 3.7.4 treats `@use "sass:math"` / `math.div()` as a *fatal syntax error* (reverted in `a96f740`). The Dart Sass slash-division warning only appears in newer local toolchains — it is version drift, not a real defect. This item only becomes valid *if* production is ever moved to Dart Sass (see the CI/CD "deferred alternative" below); until then, keep `/`.

## CI / CD

**Direction decided 2026-07-24: keep production on the `github-pages` gem and align *local* to it — the opposite of the earlier "modernize prod to Jekyll 4" plan below.** The deploy stays on GitHub Pages' proven "Deploy from a branch" builder (Jekyll ~3.10.x + Ruby Sass 3.7.4 in safe mode); local now reproduces that instead of running Jekyll 4 / Dart Sass. This was prompted by a Jekyll-4-only "fix" (`math.div` in SCSS) passing `jekyll serve` locally yet failing the live build — the version drift the earlier plan warned about, biting in the direction we didn't expect.

**Already done (2026-07-24, commit `83f9346`):**

- `Gemfile` pinned to the `github-pages` gem; `Gemfile.lock` regenerated (Jekyll 3.10.0, jekyll-sass-converter 1.5.2, Ruby Sass 3.7.4).
- `_config.yml` `exclude:` now lists `node_modules`, `src`, `.astro`, `package*.json` (Jekyll 3.10 does not auto-skip `node_modules` the way Jekyll 4 does, so local builds choked on stray `.astro` front matter).
- README documents the constraint; the Sass `math.div` TODO item above is marked do-not-do.
- Net effect: `bundle exec jekyll build` locally now matches production. The drift is closed from the local side.

**Still to do (the remaining gap): no pre-merge validation.** `allardlab/main` has no `.github/workflows/`, so a broken commit still only surfaces *after* it's pushed and the Pages builder runs. Add a **build-only check** (no deploy — deployment stays with the existing Pages builder):

1. Add `.github/workflows/build.yml`, triggered `on: pull_request` (base `main`) and `on: push` to `main`. Commit it to the repo so it runs on both `allardlab` (upstream, where the deploy lives) and `origin` (fork, catches issues pre-PR). Build-only ⇒ no secrets, no deploy permissions.
2. Two jobs (run both; each is seconds):
   - **jekyll-build-pages** — `actions/jekyll-build-pages@v1`, the *same* docker action the live Pages deploy runs. Highest fidelity: green here ⇒ the deploy will build.
   - **bundle-build** — `ruby/setup-ruby` (ruby `3.3`; prod is 3.3.4, local 3.3.5 — minor mismatch, warning only) with `bundler-cache: true`, then `bundle exec jekyll build`. Validates that the committed `Gemfile.lock` resolves under the `github-pages` gem.
3. (Optional follow-up) add an HTMLProofer step to catch broken internal links / missing images / missing PDFs — larger scope, keep as a separate non-blocking job.

**Manual step — org admin only (record who/when done):**

- **Settings → Branches → branch-protection rule on `main`:** require the `build` check before merge, so a red build blocks the merge instead of merely advising. (Requires a repo admin; not doable from the repo or without `gh` auth.)

**Deferred alternative — modernize production to Jekyll 4 / Dart Sass (NOT the current direction).** The original plan was to take over deployment with an Actions `deploy` job (`actions/upload-pages-artifact` + `actions/deploy-pages`) building with our own bundle, and flip **Settings → Pages → Source** to "GitHub Actions", so prod runs Jekyll 4.3.3. That would let the SCSS be modernized to `math.div` and drop the `github-pages`-gem pin. It is deliberately *not* being pursued now (more moving parts, and the Pages "Deploy from a branch" builder is a known-good zero-config fallback). If it is ever revisited:

- Reversion is a settings toggle, not a code change: **Settings → Pages → Source → "Deploy from a branch"** (`main` / root) immediately resumes the legacy ~3.10.x builder that runs today. So keep the ability to flip back.
- Doing this would *reverse* the local-alignment work above (un-pin the Gemfile back to `gem 'jekyll'`) and re-enable the `math.div` SCSS change. Don't do half of it — modernizing SCSS without also moving prod off the `github-pages` gem is exactly what broke the build on 2026-07-24.

## Don't-lose-these footguns

If anyone refactors the SCSS again:

- The `body { background; color; … }` block in `_sass/_09_elements.scss` is what gives the site its dark theme. Foundation used to provide this; now we own it. Deleting it would silently regress the site to light mode (the symptom we saw immediately post-Phase-2.E).
- The masthead's `22.9pt / 10pt / 14.9pt` font sizes are manually tuned so the three rows render at approximately equal pixel width in supria-sans. Don't replace with a scaled type-scale or `clamp()` ranges without re-checking visual balance.
