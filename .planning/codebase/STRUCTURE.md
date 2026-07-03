# Structure

**Analysis Date:** 2026-07-03

The directory layout is dictated by Shopify — a theme must have these top-level folders and file-naming conventions or it will not upload. There is no place to add "outside" directories at the theme root.

## Top-level layout

```
silo-studio-co-uk-new/
├── assets/       Static assets + compiled entry points
├── config/       Merchant-editable theme settings
├── layout/       Page wrappers (theme.liquid, password.liquid)
├── locales/      i18n JSON — 32 language files
├── sections/     Editor-configurable UI blocks (with {% schema %})
├── snippets/     Reusable partials (included via {% include %})
├── templates/    Page-type templates + templates/customers/ subfolder
├── .planning/    GSD planning docs (added by this workflow, not part of the theme)
└── CLAUDE.md     Repo-level guidance for Claude Code (added during /init)
```

No `package.json`, no bundler config, no test folder, no `README.md`, no CI config. The repo is theme sources only; Shopify compiles server-side on upload.

## `assets/`

Compiled entry points (linked from `layout/theme.liquid`) and static media. Only two files in this directory are custom-authored SCSS entry points; the rest are either vendored libraries or icon assets.

| File | Purpose |
|------|---------|
| `theme.js` | Main JS bundle (~1878 lines). Vendored libs at the top (jQuery plugins, FlexSlider, Modernizr, enquire.js, lodash, imagesLoaded, Magnific Popup, Drift), then `theme.*` module constructors, then `sections.register(...)` at the very bottom. |
| `theme.scss.liquid` | Theme SCSS entry (~3110 lines). Injects Liquid values (colors, fonts, sizes) at the top so editor changes recompile. |
| `timber.scss.liquid` | Timber base framework SCSS (~2653 lines). Grid, forms, mixins. |
| `gift-card.scss.liquid` + `gift-card.scss` | Standalone stylesheet for `templates/gift_card.liquid` (Shopify serves gift cards on a special layout). |
| `jquery-2.2.3.min.js` | Vendored jQuery, loaded explicitly by `layout/theme.liquid:74`. |
| `lazysizes.min.js` | Vendored lazysizes, loaded async. |
| `icons.eot`, `icons.ttf`, `icons.woff`, `icons.svg`, `icons.json` | Icon-font (Fontello-style) — vintage-theme icon delivery. |
| `ico-select.svg` + `ico-select.svg.liquid` | Select-box arrow icon. |
| `password-page-background.jpg` | Default background for the storefront password page. |

## `config/`

Two files, both required by Shopify:

- **`settings_schema.json`** — the schema definition for what's editable in the theme editor (layout toggle, color scheme, typography sizes, favicon, social links). ~2199 lines. Contains theme identity: `"theme_name": "Minimal"`, `"theme_author": "Shopify"`, `"theme_version": "11.7.20"`.
- **`settings_data.json`** — current merchant values. Do not hand-edit for things a merchant should own; put those in the schema.

## `layout/`

- **`theme.liquid`** — wraps every normal storefront page. Header, `content_for_layout`, footer, script tags. Also the location of the Silo Studio Typekit customization (line 3).
- **`password.liquid`** — wraps the storefront password page only.

## `templates/`

One `.liquid` file per Shopify page type. This is a vintage theme, so templates are `.liquid` (not `.json`), meaning merchants cannot drag-drop sections onto arbitrary pages.

| Template | Purpose |
|----------|---------|
| `index.liquid` | Home page. Uses `content_for_index` (sections chosen in editor). |
| `product.liquid` | Product detail page → renders `product-template` section. |
| `collection.liquid` | Collection listing → `collection-template`. |
| `list-collections.liquid` | All-collections index → `list-collections-template`. |
| `cart.liquid` | Cart page → `cart-template`. |
| `blog.liquid` | Blog index → `blog-template`. |
| `article.liquid` | Article detail → `article-template`. |
| `page.liquid` | Standard content page → renders `page` in header/content flow. |
| `page.contact.liquid` | Alt template — contact form. |
| `page.full-width.liquid` | Alt template — no side padding. |
| `page.no-heading.liquid` | Alt template — no page title. |
| `page.full-width-no-heading.liquid` | Combined variant. |
| `search.liquid` | Search results. |
| `404.liquid` | Not-found page. |
| `gift_card.liquid` | Gift card display (uses `gift-card.scss` standalone). |
| `password.liquid` | Storefront password page (used with `layout/password.liquid`). |
| `customers/account.liquid` | Logged-in customer dashboard. |
| `customers/activate_account.liquid` | Account activation flow. |
| `customers/addresses.liquid` | Address book. |
| `customers/login.liquid` | Login form. |
| `customers/order.liquid` | Individual order detail. |
| `customers/register.liquid` | Registration form. |
| `customers/reset_password.liquid` | Password reset. |

## `sections/`

22 sections. Each is a `.liquid` file that emits markup + a `{% schema %}` block. Sections marked with (JS) below are registered in `assets/theme.js` and receive a constructor:

| Section | Registered slug | Constructor |
|---------|-----------------|-------------|
| `header.liquid` | `header-section` | `theme.Header` (JS) |
| `footer.liquid` | — | — |
| `product-template.liquid` | `product-template` | `theme.Product` (JS) |
| `collection-template.liquid` | `collection-template` | `theme.Collection` (JS) |
| `collection-list.liquid` | `collection-list-section` | `theme.CollectionList` (JS) |
| `list-collections-template.liquid` | `list-collections-template` | `theme.ListCollections` (JS) |
| `cart-template.liquid` | `cart-template` | `theme.Cart` (JS) |
| `article-template.liquid` | `article-template` | `theme.Article` (JS) |
| `blog-template.liquid` | — | — |
| `slider.liquid` | `slideshow-section` | `theme.SlideshowSection` (JS) |
| `featured-collection.liquid` | — | — |
| `featured-blog.liquid` | — | — |
| `featured-product.liquid` | `featured-products-section` | `theme.FeaturedProducts` (JS) |
| `featured-video.liquid` | — | — |
| `feature-row.liquid` | — | — |
| `gallery.liquid` | — | — |
| `map.liquid` | `map-section` | `theme.Maps` (JS) — requires Google Maps API key |
| `newsletter.liquid` | — | Renders `snippets/newsletter-form.liquid` |
| `rich-text.liquid` | — | — |
| `product-recommendations.liquid` | `product-recommendations` | `theme.ProductRecommendations` (JS) |
| `custom-html.liquid` | — | Merchant-editable HTML block |
| `password-content.liquid`, `password-header.liquid` | `password-header` for the latter | `theme.PasswordHeader` (JS) |

## `snippets/`

Small partials included by sections/templates. Common groupings:

- **Product cards / collections:** `product-grid-item.liquid`, `collection-grid-item.liquid`, `product-unit-price.liquid`, `collection-sorting.liquid`, `collection-tags.liquid`, `search-result.liquid`, `search-result-grid.liquid`.
- **Site chrome:** `site-nav.liquid`, `mobile-nav.liquid`, `mobile-nav-icons.liquid`, `breadcrumb.liquid`, `pagination-custom.liquid`, `search-bar.liquid`.
- **Blog:** `blog-sidebar.liquid`, `featured-blog.liquid`, `onboarding-featured-blog.liquid`, `comment.liquid`, `tags-article.liquid`.
- **Media/social:** `social-links.liquid`, `social-meta-tags.liquid`, `social-sharing.liquid`, `svg-definitions.liquid`, `bgset.liquid`, `image-style.liquid`.
- **Marketing:** `newsletter-form.liquid`.

## `locales/`

32 JSON files. `en.default.json` is the source of truth — every other locale falls back on missing keys.

Available languages: `bg-BG`, `cs`, `da`, `de`, `el`, `en.default`, `es`, `fi`, `fr`, `hi`, `hr-HR`, `hu`, `id`, `it`, `ja`, `ko`, `lt-LT`, `ms`, `nb`, `nl`, `pl`, `pt-BR`, `pt-PT`, `ro-RO`, `ru`, `sk-SK`, `sl-SI`, `sv`, `th`, `tr`, `zh-CN`, `zh-TW`.

## Naming conventions

- **File names:** kebab-case (`product-grid-item.liquid`, `mobile-nav-icons.liquid`).
- **Liquid variables:** snake_case (`{% assign logo_max_width = ... %}`).
- **JS constructors on `window.theme`:** PascalCase (`theme.Product`, `theme.Header`).
- **Section registration slugs:** kebab-case matching the `data-section-type` attribute (`product-template`, `collection-list-section`).
- **SCSS files that need Liquid interpolation:** `<name>.scss.liquid` (mandatory — the pipeline recognizes the double extension).
- **Alternate page templates:** `page.<variant>.liquid` — Shopify treats these as merchant-selectable alt templates for the same page type.

## Key locations for common tasks

- **Add a new interactive section:** create `sections/<name>.liquid` with `data-section-type="<slug>"` on the root, add a `{% schema %}` block, then add a `theme.<Name>` module + `sections.register('<slug>', theme.<Name>)` line at the bottom of `assets/theme.js`.
- **Add a new merchant setting:** edit `config/settings_schema.json`. If JS needs to react live in the editor, mirror it in `window.theme.settings` inside `layout/theme.liquid:52-60`.
- **Add a translated string:** add the key to `locales/en.default.json`; other locales fall back automatically.
- **Change global colors/fonts:** merchant does it through the theme editor (backed by `config/settings_schema.json`); developers should not hard-code values in SCSS.

---

*Structure analysis: 2026-07-03*
