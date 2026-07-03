# Conventions

**Analysis Date:** 2026-07-03

Conventions here are the ones the existing (upstream Minimal + minimal Silo customization) code actually follows. Silo's future changes should stay consistent unless the whole file is being modernized.

## Liquid

**Include syntax — `{% include %}`, not `{% render %}`.**
This codebase uses the older `{% include 'social-meta-tags' %}` form throughout (e.g. `layout/theme.liquid:26`). Shopify has moved on to `{% render %}` (which has isolated scope and is faster), but the existing sections/snippets all rely on the shared-scope behavior of `include`. Do not mix the two mid-file without checking what upstream variables the snippet relies on.

**Whitespace control — used deliberately.**
`{%- ... -%}` appears where trailing/leading whitespace would otherwise leak into markup (`layout/theme.liquid:101-105` for slideshow-info). Sections default to unstripped `{% %}` because their output is typically wrapped in block-level HTML anyway.

**Schema block placement.**
Every section ends with a `{% schema %} ... {% endschema %}` block. In `sections/product-template.liquid`, the schema starts at L266 and runs to L785 — it's often the largest single block in a section file. The schema JSON is the source of truth for what the merchant sees in the theme editor.

**Root element on every wired section carries three attributes:**
```
data-section-id="{{ section.id }}"
data-section-type="<slug>"
[extra data-* attributes for section-specific config]
```
See `sections/product-template.liquid:1` for the canonical example (also carries `itemscope`/`itemtype` for schema.org).

**Filter chains are single-line.**
`{{ page_title | handle }}`, `{{ 'jquery-2.2.3.min.js' | asset_url | script_tag }}` — filters are chained on one line. No multiline pipes.

**Asset URL discipline (critical).**
Every asset reference goes through a filter — `asset_url`, `img_url`, `shopify_asset_url`, or `file_url`. Never a raw path. Assets are served from Shopify's CDN with a versioned URL; a hard-coded path breaks in production. Search the theme and you'll find zero raw asset paths.

## JavaScript (`assets/theme.js`)

**Module pattern — IIFE returning constructor.**
```js
theme.Product = (function() {
  function Product(container) {
    this.$container = $(container);
    // ...
    this.init();
  }
  Product.prototype.init = function() { /* ... */ };
  return Product;
})();
```
Applied to every wired section (`theme.Product`, `theme.Collection`, `theme.Cart`, `theme.Header`, etc.). Constructors receive the section root DOM node from `theme.Sections` (which is triggered by Shopify's section events).

**Global namespace — `window.theme` only.**
The `theme.liquid` layout primes `window.theme` with `.strings`, `.settings`, `.variables`, `.moneyFormat` (L37-66). All theme JS attaches to `theme.*`. No modules, no bundler, no ES imports — the file is one ES5 script with function-scoped `var`.

**jQuery-first.**
`$` is jQuery. All DOM work is jQuery (`.on()`, `.each()`, `.find()`). No `document.querySelector` in theme code.

**Section registration lives at the bottom of the file.**
`assets/theme.js:1788-1803` is the single source of truth for which sections are wired:
```js
$(document).ready(function() {
  var sections = new theme.Sections();
  sections.register('product-template', theme.Product);
  // ...
});
```
Adding a new interactive section means adding a line here — nothing else auto-discovers.

**Vendored libraries live above `theme.Sections`.**
Everything before L312 in `assets/theme.js` is a vendored library (jQuery.equalHeights, enquire.js, Modernizr, FlexSlider, lodash, imagesLoaded, Magnific Popup, Drift). Do not edit these; if a fix is needed, wrap them.

**ES5 only.**
No arrow functions, no `let`/`const`, no template literals, no `class`. Follow this style for consistency inside `theme.js`; if adding new code that warrants modern JS, consider adding it as a separate asset file rather than mixing styles.

## SCSS

**Two file types by convention:**
- **`.scss`** — plain SCSS, static (used only for `assets/gift-card.scss`).
- **`.scss.liquid`** — SCSS that references Liquid values (colors, fonts, sizes from `settings.*`). These are recompiled server-side whenever a merchant edits settings. Both `theme.scss.liquid` and `timber.scss.liquid` use this form.

**Liquid values injected at the very top.**
`assets/theme.scss.liquid` opens with mixin definitions, variable declarations, then font-face declarations pulled straight from `settings.type_*_family` via `{{ ... | font_face }}` (L300-312). All downstream selectors reference the assigned SCSS variables — no direct `settings.*` reads deep in the file.

**Mixin naming — camelCase.**
`@mixin clearfix()`, `@mixin prefix()`, `@mixin prefixFlex()`, `@mixin at-query()`, `@mixin accentFontStack()`. Timber's convention; keep it.

**Prefer Timber mixins over hand-rolled vendor prefixes.**
`@include prefix(property, value);` beats writing `-webkit- -moz- -ms-` variants by hand.

## JSON (schema blocks, locales)

**Locale files are flat-namespaced JSON.**
`locales/en.default.json` uses dot-notation keys (`"products.product.add_to_cart"`) as leaf strings. `{{ 'products.product.add_to_cart' | t }}` is how Liquid consumes them. When adding new user-visible copy, edit `en.default.json` first; every other locale falls back on missing keys.

**Schema blocks use trailing commas Shopify is strict about.**
Since these are parsed as JSON5-ish by Shopify's editor, keep them tidy: no trailing commas after last element in `blocks` / `settings` / `presets`. A malformed schema hides the section from the editor.

## File naming

- **Files:** kebab-case (`product-grid-item.liquid`, `mobile-nav-icons.liquid`).
- **Alt page templates:** `page.<variant>.liquid` — dots are how Shopify recognizes alternate templates.
- **JS constructors:** PascalCase on `theme.` (`theme.Product`, `theme.ProductRecommendations`).
- **Section slugs (registered names + `data-section-type`):** kebab-case (`product-template`, `product-recommendations`).
- **Liquid variables:** snake_case (`{% assign logo_max_width = ... %}`).

## Comments

**Sparse in Liquid.** Most sections rely on the schema block for "what does this section do" documentation.

**JS files include vendor license headers** for each embedded library — do not strip them (jQuery, FlexSlider, Modernizr etc. each retain their MIT/GPL header).

## What NOT to do (based on what upstream avoids)

- Do not inline `<style>` blocks that are unrelated to per-section merchant settings — put shared styles in `theme.scss.liquid`. (Per-section `<style>` blocks that consume `section.settings.*` are legitimate and appear in `sections/header.liquid`.)
- Do not add a second `window.<something>` global. Attach to `window.theme`.
- Do not `<script src="https://cdn.some-third-party.com">` in Liquid. Route third-party scripts through a section or `layout/theme.liquid` after review — external CDNs are a Content Security concern and a silent-breakage risk. The one exception currently in the theme is the Typekit stylesheet in `layout/theme.liquid:3`.
- Do not use `{% render %}` in files that were written with `{% include %}` unless you're refactoring the whole file — mixing changes scope semantics and will surface bugs.

---

*Conventions analysis: 2026-07-03*
