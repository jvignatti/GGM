# Build Plan — GGM Chrome Extension

## Phase 0: Bug Fixes (existing index.html + master_scrape.py)

Fix confirmed bugs in the running system before building the extension.
These fixes establish correct behavior that the extension will inherit.

### Unit 00-FIN1: Fix financial multi-EA sum in index.html

**The bug:** `financial.find()` returns the first matching financial row for a grant.
EE grants have 4 rows (one per budget category: OP, DUI, DD, Equipment). The dashboard
displays only the first row's expenditure, underreporting by up to $238k.

**Confirmed from live data:** EE-2025-Chittenden-00005-GR1910-1 has 4 rows totaling
$344,739.50 expended. Dashboard shows $106,593.07 (row 1 only). Portal shows $531,946.20
(close to full sum; small delta from match vs. non-match items).

**The fix:** Replace `financial.find(...)` with `financial.filter(...).reduce(sum monthly)`.
Apply in `showDetail()` and `renderGI()` wherever expenditure totals are computed.

**Files:** `GGM/GGM/index.html`

---

## Phase 1: Extension Scaffold

### Unit 01: Manifest, file structure, shared utilities

Set up the extension skeleton with correct MV3 manifest, permissions, and the shared
utility modules that all other units depend on.

**Deliverable:** Extension loads in Chrome (developer mode) with no manifest errors.
Background service worker registers. `db.js`, `format.js`, `signal.js`, `financial.js`,
`storage.js` exist with stub implementations.

### Unit 02: IndexedDB schema

Define and open the `ggm-db` database with all 7 object stores. Write the
`db.js` module with typed promise wrappers for open/get/put/getAll/clear per store.

**Deliverable:** `db.js` can open the database, write a test record, and read it back.

---

## Phase 2: Scraper

### Unit 03: Landing page scraper (invoices + QTRs)

Scrape each grant's landing page via `fetch()` with session cookies. Parse the
`auxNavSubWindow` containers for parentId 5000008 (invoices) and 5000009 (QTRs).
Store results in IndexedDB. Normalize QTR numbers on write.

### Unit 04: Indirect costs scraper

For each grant with an `Indirect Costs URL`, navigate to that page and extract:
ICR rate, MTDC sum, total IC, cost category amounts. Store in `indirect` IDB store.

### Unit 05: EE invoice sub-report scraper (OP/DUI/DD)

For each EE invoice URL, navigate to the OP Report, DUI Report, and DD Report sub-pages.
Extract enforcement metrics. Store aggregated by invoice number in `supervisors` store.

### Unit 06: Retry + carry-forward

Implement the retry loop (2 retries) and carry-forward logic for timed-out grants.
A `null` result sentinel means failure; `[]` means genuine empty. On final failure,
fall back to the existing IDB rows for that grant.

---

## Phase 3: Dashboard Core

### Unit 07: Dashboard shell (HTML + CSS)

Set up `dashboard.html` with the full CSS design system (all tokens, dark mode, all
component classes) matching `ui-context.md`. Header, toolbar, table container, detail
container — HTML structure only, no data yet.

### Unit 08: Grant list + signal rendering

Connect `dashboard.js` to IndexedDB. Render the grant list with Signal badges. Implement
`calcSignal()`, `calcINVStatus()`, `calcQTRStatus()` from `signal.js`. Wire up filters,
search, sort, and pagination.

### Unit 09: Per-grant detail view

Render the full grant detail view: hero card, contacts, financial panel, IC panel,
invoice calendar, QTR calendar. Uses `getFinancialSummary()` from `financial.js`
(multi-EA sum — not find).

### Unit 10: Financial panel (multi-EA sum)

Implement `getFinancialSummary(grantNumber)` in `shared/financial.js`:
- Filter all financial rows where `Document Name === grantNumber`
- Sum `Monthly {Month}` and `Monthly {Month} Match` across all matching rows
- Return `{ actTotal, actMatch, monthlyTotals[], monthlyMatchTotals[] }`

This is the authoritative fix for bug FIN-1 in the extension.

### Unit 11: Indirect cost panel

Read from the `indirect` IDB store. Render IC rate, MTDC, total IC, category breakdown.

### Unit 12: Grantee Intelligence + SHSO score

Render per-org composite score with 6 dimensions. SHSO score input persists to
`chrome.storage.local` (keyed by stable org hash, not raw string). Score survives
re-scrapes.

### Unit 13: Enforcement activity (EE/DRE)

Render the supervisor/enforcement panel for EE and DRE grants using data from the
`supervisors` IDB store. Monthly sparklines, hero KPIs, collapsible detail table.

---

## Phase 4: Side Panel

### Unit 14: Content script

Inject into `grants.vermont.gov/*`. Read the current grant number from the URL and/or
page heading. Send it via `chrome.runtime.sendMessage` to the side panel.

### Unit 15: Side panel

Receive grant number from content script. Look up signal and grantee score from IDB.
Display: grant number, Signal badge, org name, Grantee Score, last-scraped timestamp.
"Open in Dashboard" button opens the full extension dashboard.

---

## Phase 5: Data Import + Polish

### Unit 16: Financial xlsx import

Manual upload of SHSO Financial Report xlsx. Parse with SheetJS (bundled, not CDN).
Write rows to the `financial` IDB store. Show row count and last-import timestamp.

### Unit 17: Dark mode + UI polish

Wire up dark mode toggle to `chrome.storage.local`. Verify all panels render correctly
in both modes. PDF export forces light mode on the captured element.

### Unit 18: PDF export

Implement `exportPDF()` using bundled html2pdf.js. Forces light styles, hides the
export button during capture, restores afterward.

---

## Dependency Graph

```
01 ──► 02 ──► 03 ──► 04
               │     └──► 05
               └──► 06
               │
               ▼
00-FIN1        07 ──► 08 ──► 09 ──► 10 (uses financial.js from 01)
               │      │      └──► 11
               │      └──► 12
               │      └──► 13 (uses supervisors from 05)
               │
               └──► 14 ──► 15
               
               16 (standalone, adds to financial store)
               17, 18 (after 07–13)
```
