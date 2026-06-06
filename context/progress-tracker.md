# Progress Tracker

## Current Phase

**Phase 0 — Foundation & Bug Fixes (complete)**
**Phase 1 — Extension Scaffold (not started)**

## Current Status

The old pipeline (Python scraper + standalone index.html) is fully operational
and producing correct data as of 2026-06-06. All known dashboard bugs have been
fixed. Foundation documentation is complete. Chrome extension has not been started.

---

## Completed Work

### Pipeline (master_scrape.py — in scripts/)
- [x] Step 1: Excel exports (8 reports) — working
- [x] Step 2: Async scraper with concurrency=4 — working
- [x] **Retry fix (2026-06-05)**: Timed-out grants retry 2× and carry forward
      previous data on persistent failure. Sentinel changed from `{}` to `None`.
      All 58 FY26 grants now successfully scraped on each run.
- [x] Step 3: JSON merge + legacy merge — working

### Dashboard (index.html — in GGM/GGM/)
- [x] Grant list with Signal badges, filters, search, pagination
- [x] Per-grant detail: financial, IC, invoices, QTRs, enforcement activity
- [x] Grantee Intelligence with composite score
- [x] Dark mode
- [x] PDF export
- [x] Legacy grant support (toggle, filter, badges, 50/50 scoring)
- [x] Amendment / supplemental badges
- [x] Prev/Next navigation across filtered list
- [x] **FIN-1 fix (2026-06-05)**: `financial.find()` replaced with
      `getFinancialRows()` + `getFinancialSummary()` — sums ALL matching financial
      rows per grant. EE grants with 4 EA rows now show correct expenditures.
      Confirmed on EE-2025-Chittenden-GR1910-1: $768k → $529k remaining.
- [x] **FIN-2 fix (2026-06-06)**: Financial join now uses `startsWith(gn + '-')`
      fallback to catch grants where the Financial Report appends a sequence suffix
      (e.g. `GR1997-1` in the report vs `GR1997` in the Grant Summary).
      Confirmed on EDUC-2026-BAADA-00031-GR1997: $0 → $4,405 expended, $5,595 remaining.
      Affects all 7 amendment grants: EDUC RutlandCSD, VDH×2, BAADA; EE Vergennes, Washington, Winhall.
- [x] **MONTHS fix (2026-06-05)**: `showDetail()` crashed with
      `ReferenceError: MONTHS is not defined` after FIN-1 fix removed the local
      declaration. Fixed by using global `FIN_MONTHS` constant.
- [x] **Dark mode toggle (2026-06-05)**: Moved from inside `view-list` (hidden
      before data loads) to the header — always accessible.

### Foundation Docs (2026-06-05)
- [x] CLAUDE.md — entry point
- [x] context/project-overview.md
- [x] context/architecture.md — portal HTML map, 7 invariants, join keys
- [x] context/code-standards.md
- [x] context/ai-workflow-rules.md
- [x] context/ui-context.md — full color token tables, signal/badge specs
- [x] context/progress-tracker.md (this file)
- [x] context/specs/00-build-plan.md — 18-unit extension build plan
- [x] context/specs/00-fin1-financial-sum-fix.md — **COMPLETE**

### Analysis & Reporting (2026-06-05)
- [x] EDUC FY25 vs FY26 Word report — 19 orgs, 48 grants, scored 1–10 per year
      + general score. Saved to EDUC_FY25_FY26_Report.docx (not in repo — contains data).

---

## Open Bugs

| ID | Severity | Description | File | Notes |
|---|---|---|---|---|
| SIG-1 | Medium | "Invoice In Process" shows ✅ in calendar but excluded from signal — visible contradiction | `index.html:~1507` | Quick fix |
| GI-1 | Medium | QTR5 required for Grant Completion in Grantee Score but not in compliance signals — understates completed-grant scores | `index.html:~1973` | Quick fix |
| ORG-1 | Low | SHSO score localStorage key uses raw org name string — special chars create inconsistent keys | `index.html:~1983` | Fix in extension (use stable hash) |

---

## Open Questions

- [ ] Do the 7 amendment grants have different landing page URLs under the `-1`
      number? If so, `list_to_work.xlsx` needs updating before next scrape.
- [ ] Does the Financial Report `-1` suffix pattern apply to DRE/EE/TRCC grants
      beyond the 7 confirmed cases, or only when a grant is formally amended?
- [ ] Monthly financial actuals from per-invoice Financial Reimbursement pages
      (portal) vs. SHSO Financial Report aggregate — viable for v1 extension?

---

## Next Unit to Build

**SIG-1 fix** — Invoice "In Process" display consistency (small, do first)
**GI-1 fix** — QTR5 Grant Completion alignment (small, do second)

Then: **Unit 01 — Extension scaffold** (manifest, background, db.js, shared utils)
Spec file: `context/specs/01-extension-scaffold.md` (write before starting)

---

## Build Plan (extension phases)

| Unit | Name | Status |
|---|---|---|
| 00-FIN1 | Fix financial sum in index.html | ✅ Complete |
| 00-FIN2 | Fix amendment financial join (startsWith) | ✅ Complete |
| SIG-1 | Fix Invoice In Process calendar vs signal | 🔲 Next |
| GI-1 | Fix QTR5 Grant Completion alignment | 🔲 Next |
| 01 | Extension scaffold: manifest, background, db.js, shared/ | 🔲 |
| 02 | IndexedDB schema + migration | 🔲 |
| 03 | Scraper: landing page (invoices + QTRs) | 🔲 |
| 04 | Scraper: indirect costs per grant | 🔲 |
| 05 | Scraper: EE invoice sub-reports (OP/DUI/DD) | 🔲 |
| 06 | Retry logic + carry-forward in scraper | 🔲 |
| 07 | Dashboard shell: HTML + CSS | 🔲 |
| 08 | Dashboard: grant list + signal rendering | 🔲 |
| 09 | Dashboard: per-grant detail view | 🔲 |
| 10 | Dashboard: financial panel (multi-EA sum) | 🔲 |
| 11 | Dashboard: IC panel | 🔲 |
| 12 | Dashboard: Grantee Intelligence + SHSO score | 🔲 |
| 13 | Dashboard: enforcement activity (EE/DRE) | 🔲 |
| 14 | Content script: read grant number from portal URL | 🔲 |
| 15 | Side panel: signal + grantee score display | 🔲 |
| 16 | Financial xlsx import (manual upload) | 🔲 |
| 17 | Dark mode + UI polish | 🔲 |
| 18 | PDF export | 🔲 |
