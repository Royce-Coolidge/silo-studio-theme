# Architecture

**Analysis Date:** 2026-07-03

## Pattern

Shopify **vintage theme** (pre-Online Store 2.0) built on Shopify's official **Minimal** theme (v11.7.20). Follows Shopify's standard theme directory contract — the platform, not the code, dictates the top-level layout (`layout/`, `templates/`, `sections/`, `snippets/`, `assets/`, `config/`, `locales/`). Rendering happens server-side on Shopify's Liquid engine; there is no build step and no server code in this repo.

## Rendering Pipeline

For every request:

1. **`layout/theme.liquid`** — wraps every page. Sets up `<head>` (favicon, canonical URL, social meta via `snippets/social-meta-tags.liquid`), injects theme CSS (`timber.scss.css` then `theme.scss.css`), primes the `window.theme` global (strings, settings, moneyFormat), loads jQuery + lazysizes, and lays out `<body>` with:
   - `{% section 'header' %}`
   - `<main>{{ content_for_layout }}</main>`
   - `{% section 'footer' %}`
   - Deferred script tags including `assets/theme.js` at the very bottom.
2. **`templates/<page-type>.liquid`** — Shopify picks the template based on the request (`index.liquid`, `product.liquid`, `collection.liquid`, `page.liquid` and its variants, `blog.liquid`, `article.liquid`, `cart.liquid`, `search.liquid`, `404.liquid`, `list-collections.liquid`, `gift_card.liquid`, `password.liquid`, and the `customers/*.liquid` account pages). Each template renders one or more `{% section '...' %}` calls plus small inline Liquid.
3. **`sections/*.liquid`** — the actual UI blocks. Each section is self-contained: markup + `<style>` + `{% schema %}` JSON block at the bottom that describes settings/blocks the theme editor exposes to the merchant.
4. **`snippets/*.liquid`** — reusable partials pulled in via `{% include 'name' %}` (this codebase uses the older `include` form, not the newer `render`).

## Section ↔ JavaScript Wiring (contract)

Any section that needs client-side behavior follows a strict three-part contract:

1. **Markup:** the section's root element carries `data-section-id="{{ section.id }}"` and `data-section-type="<slug>"`. Example: `sections/product-template.liquid` starts with `<div ... data-section-id="{{ section.id }}" data-section-type="product-template" ...>`.
2. **Constructor in `assets/theme.js`:** each behavior is defined as `theme.<Name> = (function() { function <Name>(container) { ... } return <Name>; })();`. Constructors receive the section root DOM node.
3. **Registration:** the bottom of `assets/theme.js` (around L1788–L1802) instantiates `theme.Sections` (a thin wrapper over Shopify's `Shopify.theme.sections` runtime) and calls `sections.register('<slug>', theme.<Ctor>)` for every wired section:
   - `product-template` → `theme.Product`
   - `collection-template` → `theme.Collection`
   - `list-collections-template` → `theme.ListCollections`
   - `cart-template` → `theme.Cart`
   - `article-template` → `theme.Article`
   - `header-section` → `theme.Header`
   - `slideshow-section` → `theme.SlideshowSection`
   - `collection-list-section` → `theme.CollectionList`
   - `featured-products-section` → `theme.FeaturedProducts`
   - `map-section` → `theme.Maps`
   - `password-header` → `theme.PasswordHeader`
   - `product-recommendations` → `theme.ProductRecommendations`

The `theme.Sections` wrapper (defined at `assets/theme.js:312`) auto-binds each constructor to `shopify:section:load` / `unload` / `select` / `deselect` events. This is what lets the theme editor re-init a section live when a merchant edits its settings — miss this contract and a new section will render statically but stay dead in the editor.

## SCSS Pipeline

Two entry points, both compiled by Shopify's server-side `.scss.liquid` engine:

- **`assets/timber.scss.liquid`** — base framework (Timber v2.0.0): grid, forms, media-query mixins (`@mixin at-query`), typography primitives, prefix helpers.
- **`assets/theme.scss.liquid`** — theme-specific styles. Injects Liquid values at the top (`{{ accent_family | font_face }}`, color settings, size settings) so every merchant setting change triggers a recompile with new values baked in.

Both are loaded from `layout/theme.liquid:33-34` via `{{ 'timber.scss.css' | asset_url | stylesheet_tag }}`.

## Data Flow — Theme Editor Settings

```
config/settings_schema.json  (source of truth, editable schema)
        │
        ▼  (merchant edits in Shopify admin theme editor)
        │
config/settings_data.json    (current values — do not hand-edit)
        │
        ▼  (Liquid rendering)
        │
{{ settings.<key> }} in .liquid + .scss.liquid files
        │
        ▼  (mirrored for JS live-preview)
        │
window.theme.settings (initialized in layout/theme.liquid:52-60)
```

Only the subset needed by JS is mirrored to `window.theme.settings` — currently `enableWideLayout`, `typeAccentTransform`, `typeAccentSpacing`, `baseFontSize`, `headerBaseFontSize`, `accentFontSize`. Adding a new merchant-editable JS-affecting value means updating both the schema AND this global.

## Data Flow — Internationalization

```
locales/<lang>.json  (32 files; en.default.json is source of truth)
        │
        ▼
{{ 'general.meta.tags' | t }}  in Liquid
        │
        ▼  (subset exposed to JS at render time)
        │
window.theme.strings.addToCart, .soldOut, ...  (layout/theme.liquid:40-51)
```

Any new user-visible string added by Silo customizations must land in `en.default.json` first; other locales fall back if the key is missing.

## Entry Points by Page Type

| Request | Template | Key sections |
|---------|----------|--------------|
| `/` | `templates/index.liquid` | Sections chosen by merchant in editor (uses `content_for_index`) |
| `/products/<handle>` | `templates/product.liquid` | `product-template` → `theme.Product` (variants, zoom, add-to-cart) + `product-recommendations` → `theme.ProductRecommendations` |
| `/collections/<handle>` | `templates/collection.liquid` | `collection-template` → `theme.Collection` (grid, sorting, tag filter) |
| `/collections` | `templates/list-collections.liquid` | `list-collections-template` → `theme.ListCollections` |
| `/cart` | `templates/cart.liquid` | `cart-template` → `theme.Cart` (line-item quantity, notes) |
| `/blogs/<handle>` | `templates/blog.liquid` | `blog-template` |
| `/blogs/<handle>/<article>` | `templates/article.liquid` | `article-template` → `theme.Article` (comments) |
| `/pages/<handle>` | `templates/page.liquid` (+ variants: `.contact`, `.full-width`, `.no-heading`, `.full-width-no-heading`) | — |
| `/search` | `templates/search.liquid` | — |
| `/account/*` | `templates/customers/*.liquid` | Shopify's `shopify_common.js` is conditionally loaded |
| `/password` | `templates/password.liquid` + `layout/password.liquid` | `password-header`, `password-content` |

Every page also gets `header` and `footer` sections from `layout/theme.liquid`.

## Silo Studio Customizations on Top of Stock Minimal

- **Typekit stylesheet** in `layout/theme.liquid:3` (`<link rel="stylesheet" href="https://use.typekit.net/efi6ips.css">`) — added before Shopify's own `<head>` includes. This is currently the only visible customization on top of the stock Minimal theme.

No other Silo modifications are detectable without a diff against upstream Minimal v11.7.20. A future `/gsd:map-codebase --refresh` after Silo adds sections/snippets should note new files.

---

*Architecture analysis: 2026-07-03*
