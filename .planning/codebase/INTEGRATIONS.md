# External Integrations

**Analysis Date:** 2026-07-03

## APIs & External Services

**Maps & Geolocation:**
- Google Maps API - Interactive maps for store location display
  - SDK: `google.maps` (Google's official Maps API)
  - Usage: `theme.Maps` object in `assets/theme.js` (line 1567+)
  - Geocoder: `google.maps.Geocoder()` for address-to-location conversion
  - Marker: `google.maps.Marker()` for location pins
  - Config location: API key configured in theme editor via `sections/map.liquid` (line 335)
  - Implementation: Map renders in `sections/map.liquid` when API key is present

**Typography:**
- Adobe Typekit - Custom font serving
  - CDN: `https://use.typekit.net/efi6ips.css`
  - Loaded in: `layout/theme.liquid` (line 4)
  - Kit ID: efi6ips (Adobe TypeKit project)
  - Scope: Applied globally to all pages via stylesheet link

## Data Storage

**Databases:**
- Shopify Platform Database - All product, customer, order data managed by Shopify
- No custom database connections (e.g., no PostgreSQL, MongoDB, Firebase configuration)

**File Storage:**
- Shopify CDN - Theme assets served via `{{ asset_url }}` filter
- Shopify Media - Product and theme images
- Localized for multiple regions via Shopify's CDN

**Caching:**
- Shopify's built-in page caching
- Browser caching via standard HTTP headers

## Authentication & Identity

**Auth Provider:**
- Shopify Platform - Built-in customer authentication
- Customer Accounts: Optional (can be disabled)
  - If enabled: `shopify_common.js` loads for customer pages (`layout/theme.liquid`, line 82)
- Shop Pay integration available (referenced in settings but not explicitly configured)

**Session Management:**
- Shopify's server-side session handling
- `customer` object available in Liquid (used in `sections/newsletter.liquid`, line 40)

## Monitoring & Observability

**Error Tracking:**
- Not detected - No Sentry, Rollbar, or similar service configured

**Logs:**
- Shopify Admin logs - Available in Shopify admin panel
- Browser console for JavaScript errors (no remote logging)

**Analytics:**
- Not configured in theme itself (would be set up via Shopify apps or Shopify native analytics)

## CI/CD & Deployment

**Hosting:**
- Shopify Platform (Theme is deployed via Shopify CLI or admin upload)
- CDN: Shopify's global CDN for all assets

**CI Pipeline:**
- Not detected in theme repository
- Deployment handled directly through Shopify Theme Editor or CLI

## Environment Configuration

**Required env vars:**
- None - Shopify themes don't use traditional environment variables
- Settings stored in `config/settings_schema.json` and configured via Theme Editor:
  - `enable_wide_layout` (boolean)
  - `color_*` settings (hex colors)
  - `type_*_family` and `type_*_size` (typography)
  - `social_*_link` URLs (Twitter, Facebook, etc.)
  - Google Maps API key (`api_key` in map section)

**Secrets location:**
- Google Maps API Key: Entered in Shopify Theme Editor, stored in theme settings
- Social links: Public URLs (no secrets)
- No `.env` or secrets management file in repository

## Webhooks & Callbacks

**Incoming:**
- Newsletter form: `{% form 'customer' %}` (line 32 in `sections/newsletter.liquid`)
  - Endpoint: Shopify's `/contact` endpoint (Shopify-managed)
  - Method: Form POST
  - Field: `contact[email]`, `contact[tags]` = "newsletter"
  - Response: Shopify creates customer account if email not found

**Outgoing:**
- None detected in theme
- Email notifications handled by Shopify (no custom webhook triggers)

## Map Section Integration

**Component:** `sections/map.liquid`
- Requires: Google Maps API key (configured in settings, line 335)
- Address input: `map_address` setting (line 178)
- Features:
  - Geocoding: Converts address to coordinates via `google.maps.Geocoder()`
  - Map display: Renders in DOM element `#Map-{section.id}` (line 54-57)
  - Marker: Places pin at location
  - Fallback: Shows image if API key missing or map fails to load
- Event listeners: `google.maps.event.addDomListener()` for window resize (line 1664)

## Newsletter Integration

**Component:** `sections/newsletter.liquid`
- Form type: Shopify customer form (`{% form 'customer' %}`)
- Fields:
  - Email: `contact[email]`
  - Tag: `contact[tags]` = "newsletter"
- Storage: Customers stored in Shopify database with marketing consent
- Configuration via Theme Editor:
  - Heading, subtext, background color customizable
- No third-party email service (Mailchimp, Klaviyo, etc.) - native Shopify integration only

## Internationalization

**Locales (32 language files in `locales/` directory):**
- English (default): `en.default.json`
- Regional variants:
  - Portuguese: `pt-BR.json`, `pt-PT.json`
  - Chinese: `zh-CN.json`, `zh-TW.json`
  - Spanish: `es.json`
- Full list:
  - bg-BG, cs, da, de, el, en, es, fi, fr, hi, hr-HR, hu, id, it, ja, ko, lt-LT, ms, nb, nl, pl, pt-BR, pt-PT, ro-RO, ru, sk-SK, sl-SI, sv, th, tr, zh-CN, zh-TW

**Translation system:**
- Shopify's Liquid `t` filter: `{{ 'key.path' | t }}`
- Used throughout templates for user-facing text
- Settings in `config/settings_schema.json` include multilingual labels

## Shopify Platform APIs Used

**Theme-specific Shopify APIs:**
- `shop.money_format` - Store currency formatting (line 65 in `layout/theme.liquid`)
- `shop.name` - Store name in meta titles
- `page_title`, `page_description` - Auto-populated by Shopify
- `current_tags`, `current_page` - Pagination and tag filtering
- `customer` object - Logged-in user data (if accounts enabled)
- `request.page_type` - Current page type (product, index, customers/*, etc.)
- `settings.*` - Theme customization values from `config/settings_schema.json`
- `section.id`, `section.settings` - Section context and configuration

---

*Integration audit: 2026-07-03*
