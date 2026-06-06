# Code Standards — GGM Chrome Extension

## General Principles

- Write no comments unless the WHY is non-obvious (a portal quirk, a vendor bug workaround,
  an invariant that would surprise a reader). Never comment WHAT the code does.
- No speculative abstractions. Three similar lines is better than a premature helper.
- No error handling for scenarios that cannot happen. Trust that `Grant Number` exists
  on every grant record — validate only at system boundaries (portal response, file upload).
- No backwards-compatibility shims. If something is removed, remove it completely.

## JavaScript / Extension

- ES2022+ features are fine (Chrome MV3 supports modern JS).
- No TypeScript for the extension shell itself — plain JS with JSDoc types for IDE hints.
- `async/await` throughout. No raw Promise chains.
- `const` by default. `let` only when reassignment is necessary. Never `var`.
- Arrow functions for callbacks. Named `function` declarations for top-level functions.
- Destructure objects at the call site, not inside the function body.

## Chrome Extension Rules

- Background service worker: no DOM access, no `window`, no `document`.
- Content script: minimal footprint — only reads the current URL/DOM for grant number,
  sends one message. Does not modify the portal page.
- `chrome.storage.local` for user data only. Never write scraped data there.
- IndexedDB for all scraped data. Wrap all IDB operations in promise helpers — never
  use the raw callback-based API directly in business logic.
- Never use `eval()`, `innerHTML` with unsanitized strings, or `document.write()`.
- CSP in manifest: no `unsafe-inline`, no `unsafe-eval`, no external script sources.
- All resources (fonts, icons) must be bundled in the extension — no CDN at runtime.

## Portal Scraping

- Use `fetch()` with credentials `"include"` — never store or pass cookies manually.
- Parse portal HTML with `DOMParser`, not with regex.
- Read auxNav items by `data-aux-nav-parent-id` attribute — never by positional index.
- Read `aria-label` for invoice/QTR number and status. Split on `": "` once only.
  If `": "` is absent, treat the whole value as the number, status as empty string.
- Normalize QTR numbers immediately on parse (before storing). See architecture.md.
- All string joins use `.trim()` on both sides.

## Data Handling

- Financial expenditure: always sum ALL rows matching a grant's Document Name.
  Never call `.find()` on the financial store for expenditure totals.
- Numbers stored as actual numbers in IndexedDB. Strings in the source
  (e.g. `"$874,604.00"`) must be parsed: `parseFloat(str.replace(/[$,\s]/g, ""))`.
- Dates stored as ISO strings (`YYYY-MM-DD`). Compare as strings for same-day checks,
  parse to Date objects only for arithmetic.
- Empty/null/undefined fields normalize to empty string `""` in IndexedDB records,
  matching the existing JSON schema.

## CSS / Styling

- All colors, spacing, and typography via CSS custom properties defined in `:root`.
  No raw hex values in component CSS — always `var(--token-name)`.
- Dark mode via `[data-theme="dark"]` attribute on `<html>`.
- Component styles scoped by BEM-lite class naming: `.grant-card`, `.grant-card__title`,
  `.grant-card--legacy`.
- No inline `style=` attributes in HTML templates except for dynamically computed values
  (e.g. progress bar width percentages).
- Fonts bundled as woff2 files in the extension — no Google Fonts at runtime.

## File Organization

```
extension/
  manifest.json
  background.js          — service worker: scrape + IDB
  content.js             — minimal portal reader
  dashboard/
    dashboard.html
    dashboard.js
    dashboard.css
  sidepanel/
    sidepanel.html
    sidepanel.js
    sidepanel.css
  shared/
    db.js                — IndexedDB helpers (open, read, write, clear)
    signal.js            — calcSignal(), calcINVStatus(), calcQTRStatus()
    financial.js         — getFinancialSummary() — sums all EA rows per grant
    format.js            — fmt(), num(), str(), esc()
    storage.js           — chrome.storage.local helpers
  assets/
    fonts/
    icons/
  context/               — this folder (Foundation docs, not shipped in extension)
```

## Naming Conventions

- File names: `kebab-case.js`
- Function names: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- CSS classes: `kebab-case`
- IDB store names: `kebab-case` matching the JSON key (`"grants"`, `"invoices"`, etc.)
- `chrome.storage.local` keys: `shso_{descriptor}_{identifier}` — always prefixed `shso_`
  to avoid collisions with other extensions

## No-Go List

- No `alert()` or `confirm()` — use inline UI state.
- No `console.log()` in production paths — use a debug flag.
- No `setTimeout` busy-polling. Use IDB `onsuccess` callbacks or Promise chains.
- No modifying `node_modules` or vendored libs.
- No writing to `CLAUDE.md` or any `context/` file from extension runtime code.
