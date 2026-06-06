# Project Overview — GGM SHSO Grant Management Dashboard

## One-Paragraph Overview

GGM is a local-only Chrome extension that gives the Vermont Highway Safety Office a
real-time operational view of its ~58 active grants portfolio. It rides the user's
existing authenticated session on grants.vermont.gov, scrapes invoice and quarterly
report status for every active grant, pulls financial and indirect cost data, and
renders a full dashboard — plus a side panel that shows a grant's compliance signal
while browsing the portal. No credentials are stored, no data leaves the device, and
no server is involved at any stage.

## Goals

1. Replace the Python scraper + standalone HTML file pipeline with a single Chrome
   extension that runs entirely in-browser on the user's existing portal session.
2. Show accurate financial data for every grant — including EE grants that have
   multiple expense accounts (OP/DUI/DD/Equipment) — by summing all matching rows.
3. Surface compliance signals (invoice lag, QTR lag, no activity) in a side panel
   while the user browses the portal, without leaving the portal tab.
4. Preserve user-entered manual scores (SHSO Score 1–10) across re-scrapes and
   browser restarts, in a store that the scraper never touches.
5. Complete a full refresh of all ~58 active grants with automatic retry for timeouts,
   carrying forward previous data for any grant that still fails.

## Core User Flow

1. User opens Chrome and navigates to grants.vermont.gov (already logged in).
2. User clicks the GGM extension icon → extension popup/full-page dashboard opens.
3. If data is stale (> 24h) or user clicks "Refresh", extension scrapes the portal:
   - Visits each grant's landing page to read invoices and QTRs from auxNav.
   - Visits each grant's Indirect Costs URL to read ICR rates and amounts.
   - Retries timed-out grants up to 2 times before carrying forward previous data.
4. Dashboard renders: grant list with Signal badges, org Grantee Scores, financial
   summary, invoice/QTR calendar per grant, IC detail, supervisor/enforcement data.
5. While browsing a grant page on the portal, a side panel shows that grant's Signal
   level and the org's current Grantee Score.
6. User enters/adjusts SHSO Score (1–10) for an organization — saved to
   chrome.storage.local, never overwritten by scraping.

## Features

### Dashboard
- Grant list with Signal 0–5 badges, type/FY filters, search
- Per-grant detail: financial summary, monthly expenditure table, IC panel,
  invoice calendar, QTR calendar, enforcement activity (EE/DRE only)
- Grantee Intelligence: per-org composite score with breakdown, FY filter, SHSO input
- Dark mode with persistence
- PDF export of grant detail view
- Prev/Next navigation across filtered grant list

### Scraper (runs inside extension)
- Reads invoice list from `auxNavSubWindow` parentId=5000008
- Reads QTR list from `auxNavSubWindow` parentId=5000009
- Reads indirect cost data from per-grant Indirect Costs URL
- Retries failed grants 2× before falling back to cached data
- Progress indicator during scrape

### Side Panel
- Detects current portal grant number from page URL/title
- Shows: grant Signal level, org Grantee Score, last-scraped timestamp
- Updates on navigation without requiring a full reload

### Financial Report
- Monthly actuals for the SHSO Financial Report remain sourced from the
  ReportBuilder aggregate export (no per-grant equivalent on the portal).
- The extension imports this via a manual file upload (xlsx) on first run or refresh.

### Data Safety
- All data stays in IndexedDB (scraped) and chrome.storage.local (user judgments).
- No external scripts, no CDN dependencies, no telemetry.
- Sensitive Vermont financial data — handled on-device only.

## Explicit Out of Scope (v1)

- No login automation — extension requires user to already be logged in.
- No scraping of the SHSO Financial Report — manual xlsx upload only for v1.
- No scraping of DRE enforcement/supervisor data per-invoice (only available via
  aggregate Supervisors Report export).
- No scheduled background scraping — user-triggered only.
- No multi-user sync — single device, single user.
- No write-back to the portal — read-only extension.
- No EDUC/TRCC invoice sub-report scraping (not yet mapped).

## Success Criteria

- A user with an active grants.vermont.gov session can open the extension, trigger a
  scrape, and see all 58 active FY26 grants with invoice/QTR status within 10 minutes.
- Grants that time out are retried; their previous data is preserved if retries fail.
- Financial remaining balance matches the portal for EE grants with multiple EA rows.
- SHSO Score entered for an org survives a full re-scrape without being overwritten.
- The side panel shows the correct Signal for the grant being viewed on the portal.
- No data is sent to any server at any point.
