# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

This is a Shopify theme — there is no build step. Development uses Shopify CLI directly.

```bash
# Start local development server (syncs to a connected store)
shopify theme serve

# Run Theme Check linter locally
shopify theme check

# Push theme to store
shopify theme push

# Pull latest theme from store
shopify theme pull
```

CI runs two automated checks on every push (via `.github/workflows/ci.yml`):
- **Theme Check** — linting and validation (`shopify/theme-check-action@v1`)
- **Lighthouse CI** — performance testing against a live store (`shopify/lighthouse-ci-action@v1`)

## Architecture Overview

Standard Shopify theme structure. No framework, no bundler, no Node.js — pure Liquid, HTML, CSS, and vanilla JavaScript.

### Directory roles

| Directory | Role |
|-----------|------|
| `layout/` | Root HTML shells — `theme.liquid` (all pages) and `password.liquid` (locked store) |
| `templates/` | Page-type assignments — almost all are JSON files that compose sections |
| `sections/` | ~45 draggable/configurable blocks rendered by the theme editor |
| `snippets/` | ~50 reusable partials included via `{% render %}` |
| `assets/` | Flat directory of all CSS and JS (no subdirectories) |
| `config/` | `settings_schema.json` (editor UI) + `settings_data.json` (saved values) |
| `locales/` | 50 translation files across 28 languages; `en.default.json` is the source of truth |

### Templates → Sections → Snippets hierarchy

Templates (JSON) declare which sections appear on a page. Sections contain merchant-configurable settings and render snippets for repeated elements (e.g., `card-product.liquid`, `card-collection.liquid`). Snippets are not independently configurable.

### JavaScript architecture

All JS is vanilla and split into small per-feature files loaded as needed:

- `global.js` — focus trapping, accessibility helpers, shared DOM utilities
- `pubsub.js` — lightweight publish/subscribe event bus used across cart/product interactions
- `constants.js` — shared constants
- `theme-editor.js` — Shopify theme editor integration hooks

Feature files follow the pattern `<feature>.js` (e.g., `cart.js`, `product-form.js`, `predictive-search.js`, `facets.js`). Custom elements (Web Components) are used extensively — most interactive components are defined as `customElements.define(...)` classes.

### CSS architecture

- `base.css` — global CSS custom properties (color system, typography scale, spacing), resets, and layout utilities
- `component-*.css` — per-component styles (cards, drawers, price, badges, etc.)
- `section-*.css` — section-specific overrides
- `template-*.css` — template-specific overrides

CSS variables for theming are set as inline styles on `<body>` from `layout/theme.liquid` using values from `settings_data.json`.

### Localization

All user-facing strings must use `{{ 'key' | t }}` with keys defined in `locales/en.default.json`. Schema strings for the theme editor use the separate `locales/*.schema.json` files.

## Code Principles

From the project's contribution guidelines:

- **Web-native only** — no libraries or frameworks; use browser APIs directly
- **Zero layout shift** — avoid DOM manipulation before user interaction; no render-blocking JS
- **Server-rendered** — business logic belongs in Liquid on the server, not JS on the client
- **No polyfills** — use progressive degradation for older browsers instead

## Formatting

Configured via `.prettierrc.json`:
- Print width: 120
- JS: single quotes
- Liquid files: double quotes

VS Code auto-formats on save using Prettier (JS/CSS) and Shopify Theme Check (Liquid). See `.vscode/settings.json`.
