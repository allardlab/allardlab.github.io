# allardlab.github.io

Source for [allardlab.github.io](https://allardlab.github.io), the Allard Lab website. Jekyll + Sass; layout on CSS Grid + Flexbox; no CSS framework, no client JS except a five-line nav-toggle script.

## Build environment: GitHub Pages / `github-pages` gem

The live site is built by **GitHub Pages using the [`github-pages`](https://pages.github.com/versions/) gem**, which pins **Jekyll 3.10.x** and the **legacy Ruby Sass 3.7.4** engine — not standalone Jekyll 4 / Dart Sass. The `Gemfile` is deliberately pinned to `github-pages` so that `bundle exec jekyll ...` locally reproduces the production build and catches errors before they reach the deploy.

Consequences to remember:

- **Do not "modernize" the SCSS.** Ruby Sass 3.7.4 does not support the Dart Sass module system: `@use`/`@forward` and `math.div()` are fatal syntax errors in production. Keep `/` for division (see `_sass/_functions.scss`). If a newer local toolchain warns about slash-division, that warning is drift, not a bug — ignore it.
- **Keep the `Gemfile` on `github-pages`.** Switching to `gem 'jekyll'` pulls Jekyll 4 + Dart Sass and makes local builds diverge from production (this masks the errors above).
- `node_modules`, `src`, and `.astro` are gitignored Node/Astro tooling dirs that never deploy, but Jekyll 3.10 (unlike Jekyll 4) does not auto-exclude `node_modules`, so they are listed under `exclude:` in `_config.yml` to keep local builds working.

## Repo layout

- [`STATUS.md`](STATUS.md) — architecture: source-of-truth vs compiled, frameworks in use, where the displayed styles come from, design tokens, layout primitives, the page → layout → container-width map, contrast ratios.
- [`APPEARANCE.md`](APPEARANCE.md) — page-by-page description of the rendered site at desktop and mobile.
- [`STYLE.md`](STYLE.md) — design system reference: colour palette, typography scale, layout components, breakpoints, accessibility.
- [`PLAN.md`](PLAN.md) — record of the Phase 1 / Phase 2 refactor + UI overhaul work that produced the current branch.
- [`TODO.md`](TODO.md) — visible-change work that was deliberately scoped out of Phase 2 (publications/people data migration, Lighthouse pass, etc.).

## Quickstart

Local build with the production config:

```
bundle exec jekyll serve
```

Local build with the development overrides in `_config_dev.yml` (use this when you want development-only Jekyll settings to apply on top of `_config.yml`):

```
bundle exec jekyll serve --config _config.yml,_config_dev.yml
```

## Troubleshooting

If the build behaves oddly, clear the cache:

```
rm -rf .jekyll-cache
```

## Photo conversion

To bulk-resize photos for the web:

```
mogrify -resize 600x -path half_res full_res/*.jpg
```
