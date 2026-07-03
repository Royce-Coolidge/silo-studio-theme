# Concerns

**Analysis Date:** 2026-07-03

Concerns are grouped by severity of impact on future work. Every claim is anchored to a file/line so future planning can confirm before acting.

## HIGH — Framework era (vintage vs. Online Store 2.0)

**Fact.** `config/settings_schema.json` declares `"theme_name": "Minimal"`, `"theme_author": "Shopify"`, `"theme_version": "11.7.20"`. Minimal is a Shopify **vintage** theme — released before Online Store 2.0 (June 2021). All page templates are `.liquid` (e.g. `templates/product.liquid`), not `.json`.

**Implication.**
- Merchants cannot drag-and-drop sections onto arbitrary page types. Only `templates/index.liquid` (via `content_for_index`) accepts editor-managed sections.
- No support for **theme blocks**, **metaobjects-as-blocks**, or **app blocks** as OS 2.0 defines them. Shopify apps that ship "one-click add to your theme" integrations increasingly require OS 2.0 — merchants may hit walls.
- Shopify still supports vintage themes but adds no new features to them. Any Silo customization that pushes the theme far will drift further from a viable OS 2.0 migration target.

**Prevention.** If Silo plans significant new merchandising features, evaluate migrating to Dawn (OS 2.0 baseline) or Shopify's newer "Sense" family before deep customization. Extending vintage indefinitely accumulates migration cost.

## HIGH — Aged client-side JavaScript

**Fact.** `assets/theme.js` is a single ~1878-line file that inlines:
- jQuery 2.2.3 (loaded separately from `assets/jquery-2.2.3.min.js`; released 2016, EOL — jQuery 3.x has been out since 2016 and security backports for 2.x are not guaranteed)
- **FlexSlider v2.7.1** — unmaintained since ~2016; last commit years ago
- **Modernizr 2.8.2** — 2014 build; almost every feature check in it is trivially true on modern browsers
- **enquire.js 2.1.2** — unmaintained; superseded by native `window.matchMedia`
- **Magnific Popup 1.0.0** — 2015 release
- **Drift** — product image zoom, uncertain maintenance
- Full **lodash** build inline — hundreds of kilobytes when only a handful of functions (`_.assignIn`) appear to be used

**Implication.**
- Security: dependencies have not received modern XSS/prototype-pollution audits.
- Bundle size: everything ships together in `theme.js`, loaded on every page.
- Compatibility: jQuery 2 and Modernizr 2 will drift out of test coverage on newer browsers; niche mobile browsers may break first.

**Prevention.** Before adding features on top of `theme.js`, either (a) commit to modernizing the JS layer (drop jQuery, use native APIs; replace FlexSlider with a modern accessible carousel), or (b) keep changes minimal and layer new behavior into a separate script rather than growing the monolith.

## MEDIUM — Render-blocking Typekit link (Silo customization)

**Fact.** `layout/theme.liquid:3` — the very first `<head>` element (before Shopify's `content_for_header`, before charset, before viewport) is:
```
<link rel="stylesheet" href="https://use.typekit.net/efi6ips.css">
```
No `rel="preload"`, no `media="print" onload="this.media='all'"` swap, no `font-display` control. If Adobe Fonts is slow or the kit is retired, the entire theme's first paint is blocked or its typography breaks silently.

**Implication.**
- LCP degrades whenever Adobe's CDN is slow.
- Kit `efi6ips` is a single point of failure with no fallback.
- The link is above Shopify's own recommended `<head>` primitives — unusual placement.

**Prevention.** Move the Typekit link below Shopify's `content_for_header` and either (a) use `<link rel="preconnect">` + `<link rel="stylesheet" media="print" onload="...">` for async loading, (b) self-host the woff2 files via `assets/` and use `@font-face` in `theme.scss.liquid`, or (c) fall back to the theme's `settings.type_*_family` fields (which the schema already supports).

## MEDIUM — SCSS recompilation on every editor change

**Fact.** `assets/theme.scss.liquid` reads `settings.*` at compile time (colors, sizes, font families around L300-312). Every time a merchant changes a color in the theme editor, Shopify recompiles the SCSS.

**Implication.** Merchant-visible latency in the theme editor is worse than it needs to be. Not a production concern (compiled CSS is cached), but a real UX drag for anyone iterating on the theme.

**Prevention.** For values that don't need to become part of the compiled CSS (e.g. font family selection where CSS variables would suffice), consider emitting CSS custom properties from an inline `<style>` in `theme.liquid` rather than baking them into `theme.scss.css`. Vintage-era themes rarely used CSS variables; modern practice is to prefer them.

## MEDIUM — Icon delivery via icon-font

**Fact.** `assets/icons.eot`, `.ttf`, `.woff`, `.svg`, plus `assets/icons.json`. This is Fontello-style icon-font delivery. `assets/svg-definitions.liquid` (via `snippets/svg-definitions.liquid`) also exists, suggesting a partial migration to SVG sprites was underway upstream but not completed.

**Implication.**
- Icon fonts are known to cause a11y issues — screen readers may read arbitrary glyphs; browsers occasionally fail to load the font and substitute a fallback showing a mystery character.
- Two icon systems shipping in parallel is confusing for future contributors and doubles the payload.

**Prevention.** Standardize on the SVG sprite path. Convert remaining icon-font uses to `<svg><use xlink:href="#icon-name"></use></svg>` referencing `snippets/svg-definitions.liquid`. Remove `.eot`/`.ttf`/`.woff`/`.svg` icon-font assets once no Liquid references them.

## MEDIUM — Localization drift risk

**Fact.** 32 locale files under `locales/` (`bg-BG`, `cs`, `da`, `de`, `el`, `en.default`, `es`, `fi`, `fr`, `hi`, `hr-HR`, `hu`, `id`, `it`, `ja`, `ko`, `lt-LT`, `ms`, `nb`, `nl`, `pl`, `pt-BR`, `pt-PT`, `ro-RO`, `ru`, `sk-SK`, `sl-SI`, `sv`, `th`, `tr`, `zh-CN`, `zh-TW`). All inherited from stock Minimal.

**Implication.** Anything Silo adds behind a `{{ '...' | t }}` filter will be missing from every locale except `en.default.json`. Shopify's theme editor will surface a "translation missing" warning for merchants operating in those locales.

**Prevention.** Either (a) restrict Silo to markets whose locales they will maintain (probably `en.default`), or (b) add a locale-key audit script (see `TESTING.md` recommendations) that ensures every key referenced in Liquid exists at least in `en.default.json`, and only ship translation to locales where the market actually operates.

## MEDIUM — Fresh git repository (no history)

**Fact.** `git init` was run at the start of this planning session; there is no prior git history. Silo's customizations on top of stock Minimal cannot be diffed against upstream without acquiring a reference copy of Minimal v11.7.20 separately.

**Implication.**
- No way to see what Silo previously changed vs. what came from Shopify.
- Any regression cannot be `git bisect`ed.
- Attribution of past decisions is impossible.

**Prevention.** Import a reference copy of stock Minimal v11.7.20 (Shopify's GitHub or Shopify Partners download) into a separate branch named `upstream/minimal-v11.7.20`. Diff the current tree against it — the resulting patch is the Silo customization surface. Continue with that reference branch present for future upstream syncs.

## LOW — No CSP / limited security posture

**Fact.** Vintage themes cannot declare a CSP header from theme code; Shopify controls response headers. The only theme-level lever is what's loaded in `<head>`.

**Implication.** Third-party dependencies (Typekit, any future Google Maps for `sections/map.liquid`, any newsletter integration) are trusted implicitly. If any third-party CDN is compromised, the theme executes untrusted code.

**Prevention.** Minimize third-party `<script>`s. Prefer self-hosting fonts and libraries where practical. Shopify offers CSP configuration at the Plus tier; if Silo is on Plus, that's the right layer.

## LOW — No documented Google Maps API key handling

**Fact.** `sections/map.liquid` + `theme.Maps` in `assets/theme.js` exist for merchant-configured store-locator maps.

**Implication.** If the `map` section is used in production, a Google Maps API key is expected to be in the section settings (via `{% schema %}`) — the source of the key and its restriction policy are unknown from this repo alone.

**Prevention.** During the next customization pass, audit `sections/map.liquid` schema, confirm where the key is expected, and document key restrictions (HTTP referrer allowlist) for the merchant.

## LOW — CLAUDE.md and .planning/ tracking

**Fact.** `CLAUDE.md` was written in this session and lives at the repo root. `.planning/` is currently untracked and unignored.

**Implication.** Depending on `commit_docs` preference (currently defaulted to `true`), planning docs will end up in `git status`.

**Prevention.** This is handled by the `/gsd:new-project` workflow's config step later — not an immediate action.

---

*Concerns analysis: 2026-07-03*
