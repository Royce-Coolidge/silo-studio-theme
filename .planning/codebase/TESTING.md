# Testing

**Analysis Date:** 2026-07-03

## Current State

**There is no automated test suite in this repository.**

No `package.json`, no `Gemfile`, no `jest.config.*`, no `playwright.config.*`, no `.github/workflows/`, no test folders, no fixture files. Verifying a change today means opening the theme in a browser and clicking through it.

## Manual Testing Workflow

The realistic developer loop for a Shopify vintage theme:

1. **Local hot-reload preview.** `shopify theme dev --store <shop>.myshopify.com` — pulls current live theme, watches local files, syncs changes to a preview URL on the dev store. This is the primary "does my change work" check. Requires the Shopify CLI installed (`brew install shopify-cli` or npm) and the developer authenticated against a store.
2. **Shopify theme editor.** For anything driven by `{% schema %}` settings, previewing means opening the theme editor in Shopify admin and confirming a merchant can toggle the setting and see the effect. Section-level JS wiring must survive `shopify:section:load` / `shopify:section:unload` events fired by the editor — a section that only initializes on `DOMContentLoaded` will be broken here.
3. **`shopify theme check`** — Shopify's official Liquid + JSON linter. Not run automatically anywhere in this repo, but it's the closest thing to a test that exists for vintage themes. Suggested for CI when the team is ready to set it up.
4. **Cross-locale spot check.** Any Liquid string changed must be checked against `locales/en.default.json` (source of truth) and at minimum one other locale to confirm the `| t` filter still resolves. A missing key silently falls back to `en.default.json` on production but shows as `translation missing: ...` in the theme editor.

## What has been checked historically

Unknown. The repo has no CI logs, no test outputs, no `git log` (fresh `git init`), and no `CHANGELOG.md`. Silo's operational QA process on top of stock Minimal is not documented in this repo.

## Frameworks used

- **None.** No unit-test framework, no integration-test runner, no visual-regression tool, no accessibility linter, no performance budget.

## Mocking / fixtures

- **None.** Liquid is Shopify-side; mocking a store's product/collection/customer data locally is not attempted in this repo. The `shopify theme dev` workflow uses live store data.

## Coverage gaps

The largest testable surface without coverage today:

- **`sections/product-template.liquid`** + **`theme.Product`** — variant selection, price rendering, add-to-cart, product zoom. Everything a merchant sells depends on this.
- **`sections/cart-template.liquid`** + **`theme.Cart`** — quantity updates, notes, checkout hand-off.
- **`sections/header.liquid`** + **`theme.Header`** — sticky-nav, mobile-nav, search bar.
- **All 32 `locales/*.json` files** vs. actual keys used in Liquid — a real string coverage tool would show which locales are missing which keys.
- **Checkout flow** — templates/customers/*.liquid (login, register, reset password) receive minimal customization from Silo and are the most sensitive customer-facing screens for account management.

## Recommendations if automated testing becomes a v-next priority

The pragmatic path (not implemented here):

1. **Playwright against a Shopify dev store** — script cart-add, variant-select, checkout-start flows against a preview URL. Runs against a real Shopify backend so it exercises the same Liquid the merchant sees. This is the option most Shopify agencies actually adopt.
2. **`shopify theme check` in a pre-commit hook** — cheapest lint gate. Catches broken schemas and Liquid syntax errors before they reach Shopify.
3. **Locale key audit script** — a tiny Node script that greps `| t` calls in `.liquid` files and diffs the key set against `locales/en.default.json`. Would prevent "translation missing" in the editor.
4. **Lighthouse CI on the dev-store preview** — the largest existing performance risk (jQuery + FlexSlider + Magnific + Drift + synchronous Typekit) is measurable but nobody is measuring it.

None of the above exists today. Any test work is greenfield.

---

*Testing analysis: 2026-07-03*
