# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Yu-Gi-Oh! proxy card generator delivered as a **single self-contained `index.html` file** (inline `<style>` + inline vanilla-JS `<script>`, no build step, no framework, no npm dependencies). The full spec lives in `REQUIREMENTS.md` (German) — read it before changing behavior, since requirements like the 3×3 print grid, 63×88mm card size, crop marks, mirrored card backs, and the YDK format are normative, not incidental.

Hard constraints from the spec that any change must preserve:
- Stays a **single self-contained HTML file** — no bundlers, no build step, no external scripts/styles, no analytics/tracking
- Must run by double-clicking the file (`file://` origin) in current Chrome/Firefox/Edge/Safari on Windows/macOS/Linux

## Running / testing

There is no build, lint, or test tooling — open `index.html` directly in a browser (double-click, or `file://...index.html`). The YGOPRODeck API allows CORS from any origin, so `fetch` works fine from a `file://` page. The file is also deployed as a static site via GitHub Pages — see [README.md](README.md) for the live URL.

Manual verification checklist (no automated tests exist):
- Search a card name (debounced ~400ms) → results show thumbnail/name/type → click adds to deck
- Adjust quantity (1–3), remove a card, confirm the live total count
- Export `.ydk`, re-import it, confirm the deck round-trips (including multi-copy counts capped at 3)
- Open print preview → verify 3×3 grid, crop marks, and mirrored card-back pages when the back toggle is on → `window.print()`
- Toggle DE/EN and confirm all visible strings switch
- Resize the window to confirm the two-column → single-column responsive breakpoint

## Architecture

Everything lives in `index.html`. The inline `<script>` is one IIFE organized into clearly-separated sections (in source order):

1. **i18n** — `STRINGS` dict (`de`/`en`), `t(key)` lookup, `applyI18n()` walks `[data-i18n]` / `[data-i18n-placeholder]` elements and re-renders dynamic lists so language switches affect already-rendered content too
2. **API client** — `normalizeCard`, `searchCards`, `resolveCardsByIds` wrap the YGOPRODeck `cardinfo.php` endpoint (search by `fname`, batch-resolve by `id`); a `400` response means "no matches" and is treated as an empty result, not an error
3. **Search** — debounced input handler with `AbortController` to cancel stale in-flight requests; renders into `#search-results`
4. **Deck state** — plain array `deck` of `{ id, name, type, image, imageSmall, quantity }`; `addCardToDeck` / `setQuantity` / `removeCard` / `totalCount` mutate it directly, and every mutation calls `renderDeck()` to redraw `#deck-list` and the live count
5. **YDK import/export** — `exportYdk` builds `#main`/`#extra`/`#side` text and triggers a `Blob` download; `parseYdkMainIds` + `onImportFile` parse the `#main` block, dedupe/cap copies at 3, batch-resolve names via the API, and replace `deck` wholesale
6. **Print preview** — `renderPrintPages` expands the deck by quantity, chunks into groups of 9, and builds one `.print-page` per group via `buildPage(cards, isBack)`; back pages reuse the same card grouping with `CARD_BACK_URL` and a `.mirrored` class (`transform: scaleX(-1)`) so duplex long-edge printing aligns backs to fronts; crop marks are 4 absolutely-positioned `.crop-mark` spans per `.card-slot`
7. **Wiring** — all `addEventListener` calls and the initial `applyI18n()` / `renderDeck()` at the bottom of the IIFE

CSS lives in the `<head>` `<style>` block, organized as: theme variables (dark + gold palette via custom properties) → layout/panels/buttons/lists → toast → print-overlay/preview grid/crop-marks → `@media print` (hides the app shell, shows only `.print-pages`, sets `@page { size: A4; margin: 0 }`).

When adding features, keep new code inside the existing IIFE sections rather than introducing new files or external scripts — the single-file deliverable constraint is the whole point of this project (see `REQUIREMENTS.md` → "Infrastruktur").
