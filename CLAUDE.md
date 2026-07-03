# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **Shopify theme** built on the official **Minimal** theme by Shopify (v11.7.20, see `config/settings_schema.json`). The repository is theme sources only — there is no `package.json`, no bundler, no Shopify CLI config file, and no `.git` directory here. Shopify compiles the SCSS/Liquid on their servers.

## Commands

There is no local build step. Assets like `theme.scss.liquid` compile to `theme.scss.css` server-side when uploaded — never link a plain `.scss` file, always request the `.css` version through `asset_url`.

Typical workflow uses the Shopify CLI against a linked store (not currently configured in-repo — ask the user for the store URL / theme ID before running):

- `shopify theme dev --store <shop>.myshopify.com` — hot-reloading local preview against a live store
- `shopify theme push --theme <id>` — upload changes to a specific theme
- `shopify theme pull --theme <id>` — download the current live theme to compare
- `shopify theme check` — lint Liquid + JSON schemas

There is no test framework in this theme. "Verifying" a change means previewing it in `shopify theme dev` or in the Shopify theme editor.

## Architecture

### Standard Shopify layout
`layout/theme.liquid` wraps every page. Rendering order for a normal page:
1. `layout/theme.liquid` → header, injects CSS/JS, then `{{ content_for_layout }}`
2. `templates/*.liquid` (page-type template, e.g. `product.liquid`) which renders one or more `{% section '...' %}` tags
3. `sections/*.liquid` — the actual UI blocks, each with a `{% schema %}` JSON block that exposes settings to the theme editor
4. `snippets/*.liquid` — reusable partials pulled in via `{% include %}` / `{% render %}`

`config/settings_schema.json` defines global theme-editor settings; `config/settings_data.json` holds the current merchant's values. Never hand-edit `settings_data.json` for anything the merchant should own — put it in the schema instead.

### Section ↔ JS wiring
Sections that need JS behavior follow a strict contract:
1. The section's root element carries `data-section-id="{{ section.id }}"` and `data-section-type="<slug>"` (e.g. `product-template.liquid` uses `data-section-type="product-template"`).
2. `assets/theme.js` defines each behavior as `theme.<Name> = (function() { ... })()` returning a constructor.
3. The bottom of `theme.js` (~L1788) registers each type against a slug: `sections.register('product-template', theme.Product);`. Any new interactive section must add both a `data-section-type` on the markup and a matching `sections.register()` call — otherwise it will not initialize (and specifically will not re-init inside the theme editor when the merchant edits the section).

`theme.Sections` (theme.js:312) is a thin wrapper around Shopify's `Shopify.theme.sections` runtime and handles editor re-load events (`shopify:section:load` / `unload` / `select` etc.).

### Styles
Two SCSS entry points, both compiled by Shopify's `.scss.liquid` pipeline:
- `assets/timber.scss.liquid` — base framework (grid, forms, typography primitives) inherited from Shopify's Timber
- `assets/theme.scss.liquid` — theme-specific styles; **injects Liquid values** (theme editor colors, font-family via `font_face`, font sizes) at the top so changing a color in the editor recompiles CSS with the new value

Both are loaded from `layout/theme.liquid` via `{{ 'timber.scss.css' | asset_url | stylesheet_tag }}`.

### Vendored JS (do not edit)
`assets/theme.js` bundles jQuery plugins and vendor libs inline: FlexSlider, enquire.js, Modernizr, lodash, imagesLoaded, Magnific Popup, Drift zoom, and a small `equalHeights` plugin. Theme code sits below these (from ~L300). Prefer extending the existing `theme.*` modules over adding new script tags.

### Globals set up in `layout/theme.liquid`
- `window.theme.strings` — translated UI strings pulled from `locales/*.json` at render time
- `window.theme.settings` — a subset of merchant settings mirrored to JS so the editor's live-preview stays in sync
- `window.theme.moneyFormat` — used by `Shopify.formatMoney` for price rendering
- A stylesheet link to Typekit (`https://use.typekit.net/efi6ips.css`) is loaded before everything else — this is a Silo-Studio customization on top of stock Minimal.

### i18n
All user-visible strings live in `locales/*.json`. `en.default.json` is the source of truth; other locales fall back to it. Reference strings from Liquid with `{{ 'key.path' | t }}` and from JS via `window.theme.strings.<name>` (which must first be exposed in `layout/theme.liquid`).

## Working on this codebase

- When adding a new theme-editor-controlled option, both the section's `{% schema %}` block AND (if the value needs to affect JS live-preview) `window.theme.settings` in `layout/theme.liquid` need updating.
- The stock Minimal theme documentation is at https://docs.shopify.com/manual/more/official-shopify-themes/minimal — useful for understanding intended behavior before changing it.
- Assets referenced by Liquid must go through a filter (`asset_url`, `img_url`, `shopify_asset_url`) — direct paths will break in production because assets are served from a CDN.
