# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**El Tres Cafe** — a static digital menu for a real cafe in Alicante, Spain, deployed via GitHub Pages at `eltres.cafe`. No build system, no framework, no package manager. The entire app is a single HTML file with embedded CSS and JS.

## Development

No build or install step. Open `menu.html` directly in a browser, or use any static server:

```bash
python3 -m http.server 8080
# then open http://localhost:8080/menu.html
```

`index.html` is just a meta-redirect to `menu.html`.

## Architecture

Everything lives in [menu.html](menu.html) in three inline blocks:

**`<style>` (lines 7–447)** — All CSS using CSS custom properties (`--brand`, `--bg`, `--text`, etc.). Dark mode is implemented via `[data-theme="dark"]` overrides on `:root` variables, toggled with `localStorage`.

**`<script>` block 1 — DATA (lines 452–872)** — Four global objects that contain all content:
- `CONFIG` — `whatsapp` number and `tables` array (table numbers shown in checkout)
- `LANGS` — language codes to display names/flags (currently `es`, `en`, `ru`, `uk`)
- `UI` — all UI strings keyed by language code
- `MENU` — full menu content keyed by language, containing `categories[]` and `specials`. Each item has: `id`, `name`, `desc`, `price`, `emoji`, optional `image`/`modalImage`/`modalImagePosition`, and `tags[]` (`"popular"`, `"vegetarian"`, `"healthy"`, `"seasonal"`). Mark an item unavailable with `soldOut: true`.

Categories support two layouts: flat (`items[]`) or grouped (`groups[]`, each with its own `items[]`). Groups render as collapsible accordion sections within the category.

**`<script>` block 2 — JS runtime (lines 981–1525)** — Vanilla JS managing:
- `currentLang` (string), `cart` (array of `{item, qty}`), `modalItem`, `modalQtyVal`, `orderType`, `selectedTable` — all module-level state
- `render()` (line 1036) — rebuilds all dynamic HTML from current `currentLang` state; called on every language switch
- Cart drawer → checkout drawer (pickup / table / delivery) → WhatsApp order via `submitOrder()` (line 1449)
- Search/filter, scroll-spy for nav tabs, fly-to-cart animation, haptic feedback via `navigator.vibrate`
- `?table=N` URL parameter pre-selects the table number in checkout (used by QR codes at tables)
- Delivery address autocomplete via Nominatim/OpenStreetMap (`fetchAddrSuggestions`, line 1494)

## Adding/Editing Menu Items

Add or edit entries in the `MENU` object in the first `<script>` block. To add items with photos, place images in [Assets/](Assets/) and reference them as `image:"Assets/filename.jpg"` (card thumbnail) and `modalImage:"Assets/filename.jpg"` (modal view). All four language copies of `MENU` (`es`, `en`, `ru`, `uk`) must be kept in sync manually.

## Deployment

Push to `main` (or the default branch) — GitHub Pages auto-deploys. The custom domain is configured via [CNAME](CNAME) (`eltres.cafe`).
