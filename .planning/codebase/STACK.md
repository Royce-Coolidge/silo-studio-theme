# Technology Stack

**Analysis Date:** 2026-07-03

## Languages

**Primary:**
- Liquid - Shopify template language for dynamic content, used throughout `layout/`, `sections/`, `snippets/`
- SCSS (`.scss.liquid`) - Styling with Liquid variable interpolation in `assets/timber.scss.liquid` and `assets/theme.scss.liquid`
- JavaScript - Client-side behavior in `assets/theme.js` and vendored libraries

**Secondary:**
- JSON - Theme configuration in `config/settings_schema.json`, locale strings in `locales/*.json`

## Runtime

**Environment:**
- Shopify Liquid Engine (server-side template rendering)
- Shopify Storefront API (theme data access)
- Browser JavaScript runtime (ES5/ES6)

**Package Manager:**
- None - This is a Shopify theme repository without `package.json`, `Gemfile`, or other dependency manifests
- No build step - SCSS and JavaScript are served and compiled server-side by Shopify

## Frameworks

**Core:**
- Shopify Minimal Theme v11.7.20 - Official Shopify theme base (configured in `config/settings_schema.json`)
- Timber v2.0.0 - Shopify SCSS framework (see header in `assets/theme.scss.liquid`)

**Styling:**
- SCSS with Timber mixins (`@mixin clearfix()`, `@mixin prefix()`, `@mixin prefixFlex()`)

## Key Dependencies

**Vendored JavaScript Libraries (in `assets/theme.js`):**
- jQuery 2.2.3 - DOM manipulation (loaded via `{{ 'jquery-2.2.3.min.js' | asset_url | script_tag }}` in `layout/theme.liquid`)
- Modernizr 2.8.2 - Feature detection (custom build for touch, csstransforms, csstransforms3d, fontface)
- enquire.js v2.1.2 - Media query management in JavaScript
- imagesLoaded v4.1.1 - Image loading detection with EventEmitter
- jQuery FlexSlider v2.7.1 - Image carousel/slider functionality
- Magnific Popup v1.0.0 - Lightbox modal for images (2015-03-30 release)
- Drift - Product image zoom functionality
- lodash - JavaScript utility library
- lazysizes - Lazy image loading (async-loaded via `{{ 'lazysizes.min.js' | asset_url }}`)

**Shopify Assets (server-provided):**
- `option_selection.js` - Product variant selection (loaded conditionally for product pages)
- `shopify_common.js` - Common Shopify utilities (loaded conditionally for customer pages)

## Configuration

**Environment:**
- Settings are configured through Shopify Theme Editor
- `config/settings_schema.json` (2199 lines) defines all editable theme settings:
  - Layout settings (wide layout toggle)
  - Color scheme (topbar, body, footer, buttons, text, borders)
  - Typography (body font, header font, accent font, sizes 12-36px)
  - Favicon image upload
  - Social media links (Twitter, Facebook, etc.)

**Build:**
- No build configuration files (no Webpack, Gulp, Rollup, Vite, etc.)
- No `tsconfig.json`, `.babelrc`, `jest.config.js`
- No `.nvmrc` or Node version pinning detected

## Platform Requirements

**Development:**
- Shopify CLI or theme development tools (for syncing/testing)
- Text editor or IDE
- Web browser for theme preview

**Production:**
- Shopify Plus or standard Shopify store (renders via Shopify's Liquid engine)
- Modern browser support: IE9+, modern Chrome/Firefox/Safari

## Asset Pipeline

**Compiled Assets:**
- `timber.scss.css` - Generated from `assets/timber.scss.liquid` (compiled server-side)
- `theme.scss.css` - Generated from `assets/theme.scss.liquid` (compiled server-side)
- Loaded in `layout/theme.liquid` via: `{{ 'timber.scss.css' | asset_url | stylesheet_tag }}`

**Script Loading Order:**
1. jQuery 2.2.3 (required by other libraries)
2. lazysizes async (for lazy image loading)
3. option_selection.js (conditional: product pages only)
4. theme.js (main theme bundle with all vendored libraries)

---

*Stack analysis: 2026-07-03*
