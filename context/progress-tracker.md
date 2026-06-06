# Progress Tracker

## Current Phase

**Phase 0 — Foundation & Bug Fixes (complete)**
**Phase 1 — Extension Scaffold (not started)**

## Current Status

The standalone dashboard (`index.html`) is stable and production-quality for EDUC
and DRE grants as of 2026-06-06. All known data bugs and display inconsistencies
have been fixed. Foundation documentation is complete. Chrome extension not started.

---

## ⚠️ REQUIRED BEFORE PRODUCTION — Full Pipeline Re-Run

Once all dashboard bugs are fixed and the canonical schema is stable, the entire
pipeline must be re-run from scratch. The current `grants_data.json` (and the
`old_gears_data.json` used as the legacy base) were generated before these fixes:

- **FIN-1**: Multi-EA financial sum — FY25 EE grants show understated expenditures
  (first EA row only). All FY25 EE financial figures are wrong in the current JSON.
- **FIN-2**: Amendment `-1` join — FY25 amended grants may have missing financial
  data depending on when the Financial Report suffix changed.
- **Budget field** (`Apv Total` priority) — any FY25 EDUC grant with both fields
  populated was showing `Apv Program Total` (federal + match) as approved budget.
- **Vendor inconsistencies** (trailing spaces, QTR normalization) — handled at
  runtime but should be normalized at write time before extension build.

**Re-run order:**
1. Finish remaining bug fixes (SIG-1, GI-1, canonical schema design)
2. Re-run `master_scrape.py` full (Step 1 exports + Step 2 scraper + Step 3 merge)
3. Verify FY25 EE financial figures match portal for a sample of grants
4. Regenerate EDUC FY25 vs FY26 Word report from corrected data
5. Use new clean JSON as authoritative base going forward

**Do not build the Chrome extension against the current JSON.**

---

## Completed Work

### Pipeline (master_scrape.py — in scripts/, not in git repo)
- [x] Step 1: Excel exports (8 reports) — working
- [x] Step 2: Async scraper with concurrency=4 — working
- [x] **Retry fix (2026-06-05)**: Timed-out grants retry 2× and carry forward
      previous data on persistent failure. Sentinel `None` vs `{}`. All 58 FY26
      grants now successfully scraped on each run.
- [x] Step 3: JSON merge + legacy merge — working

### Dashboard (index.html) — all fixes as of 2026-06-06

| Fix | Commit | Description |
|---|---|---|
| Dark mode to header | 6119ba8 | Moved toggle from hidden toolbar to always-visible header |
| MONTHS crash | 6119ba8 | `showDetail()` crashed after FIN-1 removed local MONTHS; fixed via global FIN_MONTHS |
| **FIN-1** | 6119ba8 | `financial.find()` → `getFinancialRows()` + `getFinancialSummary()` sums ALL EA rows. EE grants have 4 rows; using first only understated expenditures by up to $238k |
| **FIN-2** | 6119ba8 | `startsWith(gn + '-')` fallback catches grants where Financial Report appends `-1` suffix. Affects all 7 amendment grants. |
| **Budget field** | 6b5b624 | `Apv Total` (federal award only) takes priority over `Apv Program Total` (federal+match). Fixes inflated approved budget display. |
| IC panel false-positive | 5e789b1 | Panel now requires `charged > 0 AND totalIC > 0`. Rate-only grants (2 EE grants: St.Johnsb, Woodstock) no longer show empty IC panel. |
| IC category checkmarks | 2d7a433 | ✅ only shows when category is IN the IC base (has IC amount > 0). Column renamed "Has Cost" → "In IC Base". |

### Foundation Docs
- [x] CLAUDE.md
- [x] context/project-overview.md
- [x] context/architecture.md — portal HTML map, 11 invariants, join keys, vendor patterns
- [x] context/code-standards.md
- [x] context/ai-workflow-rules.md
- [x] context/ui-context.md — full color token tables, signal/badge specs
- [x] context/progress-tracker.md (this file)
- [x] context/specs/00-build-plan.md — 18-unit extension build plan
- [x] context/specs/00-fin1-financial-sum-fix.md ✅ complete
- [x] context/specs/00-fin2-amendment-join-fix.md ✅ complete

### Analysis & Reporting
- [x] EDUC FY25 vs FY26 Word report — 19 orgs, 48 grants, scored 1–10
      Saved locally: `EDUC_FY25_FY26_Report.docx` (not in repo — contains data)
      **Note:** Needs re-run after full pipeline re-run (FY25 EE financials were wrong)

---

## Open Bugs (not yet fixed)

| ID | Severity | Description | File/Location |
|---|---|---|---|
| SIG-1 | Medium | "Invoice In Process" shows ✅ in calendar but excluded from signal — contradiction visible to user | `index.html` ~line 1507 |
| GI-1 | Medium | QTR5 required for Grant Completion in Grantee Score but compliance signals only count QTR1–4 — understates completed-grant scores | `index.html` ~line 1973 |
| ORG-1 | Low | SHSO score localStorage key uses raw org name string — special chars (`&`, `/`, spaces) create inconsistent keys | `index.html` ~line 1983 |

---

## Known Vendor Data Inconsistencies (documented in architecture.md)

| INV | Description | Impact |
|---|---|---|
| INV-1 | EE grants have 4 financial rows per EA — must sum all | ✅ Fixed FIN-1 |
| INV-6 | Trailing spaces in grant numbers (`Vergennes `) | Handled via .trim() |
| INV-7 | Extension must never store credentials | Extension constraint |
| INV-8 | "Bennington" org name shared by CSD and Town of Bennington in new GEARS | Deferred — EE out of scope |
| INV-9 | IC panel: rate can be entered without dollar approval | ✅ Fixed |
| INV-10 | `Apv Match Total = $0` is valid for no-match grants | Documented |
| INV-11 | Financial Report appends `-1` to Document Name on amended grants | ✅ Fixed FIN-2 |
| QTR bug | Portal emits `DREQTR1` without dash for some DRE QTRs | Normalized on scrape |

---

## Open Questions for Next Session

- [ ] **Canonical schema redesign**: User wants to make the system vendor-proof.
  Normalize all fields at write time (in merge_json) so the dashboard reads clean
  canonical fields, not raw portal field names with defensive fallbacks everywhere.
  See discussion in session 2026-06-06 — design agreed in principle, not implemented.
- [ ] **list_to_work.xlsx**: Do the 7 amendment grants have different landing page
  URLs under the `-1` number? If so, scraper needs updating before next run.
- [ ] **SIG-1 fix**: Small — Invoice In Process should be excluded from calendar
  display to match signal calculation.
- [ ] **GI-1 fix**: Small — QTR5 should not be required for Grant Completion score.
- [ ] **EDUC report re-run**: After full pipeline re-run, regenerate Word report.

---

## Next Unit to Build

**SIG-1** — Invoice "In Process" calendar fix (small, ~15 min)
**GI-1** — QTR5 Grant Completion alignment (small, ~15 min)
**Spec 01** — Grant Report Generator button (spec written, ready to implement)
**Canonical schema** — vendor-proof field normalization at merge time

Spec 01 is fully written: `context/specs/01-grant-report-generator.md`
Answer the two open questions in the spec before starting implementation.

---

## Build Plan (Chrome Extension)

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
