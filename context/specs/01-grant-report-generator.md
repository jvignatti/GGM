# Spec 01: Grant Performance Report Generator

## Goal

Add a "Generate Report" button to the dashboard toolbar that lets the user select
one or more grant types (EDUC, EE, DRE, TRCC) and one or more fiscal years (FY25,
FY26, etc.), then downloads a fully formatted Word (.docx) report — identical in
content and scoring logic to `EDUC_FY25_FY26_Report.docx` — generated entirely in
the browser from the already-loaded grants_data.json.

No server. No Python. No new files on disk. Click → .docx downloads.

---

## Design

### Button placement
A new **"📊 Report"** button in the toolbar-right area, next to "Reset filters".
Visible only after data is loaded (same condition as the rest of the toolbar).

### Report modal / inline controls
Clicking the button opens a compact inline panel below the toolbar (not a modal
overlay — keeps UX consistent with the rest of the dashboard):

```
┌─────────────────────────────────────────────────────────────────┐
│  📊 Generate Performance Report                           [✕]   │
│                                                                   │
│  Grant Type   [✅ EDUC] [☐ EE] [☐ DRE] [☐ TRCC]               │
│                                                                   │
│  Fiscal Year  [✅ FY26] [✅ FY25] [☐ FY24] [☐ FY23] ...        │
│                                                                   │
│  Include      [✅ Active grants] [☐ Legacy grants]              │
│                                                                   │
│  [Generate & Download Report]     Estimated: ~19 orgs, ~48 grants│
└─────────────────────────────────────────────────────────────────┘
```

- Grant type chips: multi-select toggle buttons, at least one required
- Fiscal year chips: built dynamically from the loaded data's Sub Code values,
  sorted descending (FY26 first), multi-select, at least one required
- "Estimated" counter updates live as selections change
- Generate button disabled until at least one type and one year selected

### Report content (per-org, per selected type/year)
Identical structure to the Python report:
1. **Cover page** — report title, selected types, selected years, date
2. **Portfolio summary table** — all orgs, FY scores side by side
3. **Per-org detail page** — one page per org:
   - Score chips (FY scores + General)
   - FY25 metrics table (if selected) — invoice %, QTR %, utilization vs budget
   - FY26 metrics table (if selected) — invoice %, QTR %, utilization vs pace
   - Missing invoices / missing QTRs flagged in red
   - Score rationale + year-over-year trend arrow
4. **Appendix** — ranked table by general score

### Scoring logic
Same as `build_educ_report.py` — ported to JavaScript:
- Invoice compliance: 40% weight
- QTR compliance: 30% weight
- Financial utilization: 30% weight (FY26 scored vs expected pace, FY25 vs full year)
- FY26 "perfect" = current to date (months elapsed + QTRs due as of today)
- General score = 40% FY25 + 60% FY26 when both present, else single-year score
- No match data factored in (match is separate from federal award)

### Financial data
Uses `getFinancialSummary(gn)` — the fixed multi-EA sum. Approved budget = `Apv Total`.

---

## Implementation

### New dependency
`docx` library (docx.js) — generates proper .docx files in the browser.
Load via CDN, same pattern as xlsx.js already in the dashboard:
```html
<script src="https://unpkg.com/docx@8.5.0/build/index.js"></script>
```
Exposes `window.docx` with `Document`, `Paragraph`, `Table`, `TableRow`,
`TableCell`, `TextRun`, `HeadingLevel`, `AlignmentType`, etc.

### New files / sections in index.html

**1. Report controls panel (HTML)**
A hidden `<div id="report-panel">` inserted after the toolbar, shown/hidden by
the Report button. Contains the type/year chip toggles and generate button.

**2. `buildReport(types, years, includeLegacy)` function**
Main orchestrator. Collects all grants matching the selection, groups by org,
computes scores using existing `getFinancialSummary()`, `countInvSub()`,
`countQTRSub()`, `countDueMonths()`, `countDueQTRs()`. Returns a structured
results object (same shape as `build_educ_report.py`'s `all_results`).

**3. `renderDocx(results, types, years)` function**
Builds the docx Document using the `docx` library. Sections:
- Cover page paragraph
- Summary table (one row per org, columns: Org, FY scores, Inv%, QTR%, Util%)
- Per-org sections (one per org, page break between each)
- Appendix ranked table

**4. `downloadDocx(doc, filename)` function**
Uses `docx.Packer.toBlob(doc)` → creates object URL → triggers download.

**5. Live estimate counter**
On each chip toggle, count `grants.filter(r => types.includes(r["Grant Type"])
&& years.includes(str(r["Sub Code"])) && (includeLegacy || !r["legacy"]))` and
display org count + grant count.

### Scoring helpers (new, shared)
Extract from existing `renderGI()` into reusable functions so both the Grantee
Intelligence view and the report generator use the same logic:
- `computeFinUtil(grants, financial)` → 0–1
- `computeInvCompliance(grants, invoices)` → `{rate, missing[], sub[]}`
- `computeQTRCompliance(grants, qtrs)` → `{rate, missing[], sub[]}`
- `scoreGrant(finUtil, invRate, qtrRate, fy)` → 1–10

These replace the inline score logic currently duplicated between `renderGI()`
and `build_educ_report.py`.

---

## Dependencies

- `docx@8.5.0` (unpkg CDN) — Word document generation in the browser
  No other new dependencies. All scoring uses existing dashboard functions.

---

## Verify when done

- [ ] "📊 Report" button appears in toolbar after data is loaded
- [ ] Clicking opens the report panel; clicking again or ✕ closes it
- [ ] Selecting EDUC + FY26 shows correct estimated org/grant count
- [ ] Multi-select works: EDUC + EE together includes both types
- [ ] Selecting no types or no years disables the Generate button
- [ ] Clicking Generate downloads a .docx file
- [ ] Downloaded file opens in Word without errors
- [ ] Cover page shows correct type(s), year(s), and today's date
- [ ] Summary table matches portal data for a known grant
  (e.g. EDUC-2026-Chittenden: approved = $96,500, not $120,625)
- [ ] BAADA shows $4,405 expended (amendment join working in report)
- [ ] EE orgs score correctly with multi-EA financial sum
- [ ] Per-org page shows correct missing invoices and QTRs
- [ ] Score for LocalMotio FY25 matches Python report (9.7)
- [ ] Dark mode dashboard still renders correctly after adding the library
- [ ] No console errors
- [ ] `context/progress-tracker.md` updated

---

## Resolved Decisions

- **Format:** Word (.docx) only for v1. PDF already covered by per-grant export.
- **SHSO Score:** Include from localStorage. If not entered for an org, redistribute
  its 15% weight proportionally across the other 5 dimensions (same logic as
  Grantee Intelligence view).
- **Filename:** `SHSO_[PROGRAM/PROGRAMS]_FY[YEARS]_REPORT_[DATE].docx`
  - Single type: `SHSO_EDUC_FY2025_FY2026_REPORT_2026-06-06.docx`
  - Multiple types: `SHSO_EDUC_EE_FY2026_REPORT_2026-06-06.docx`
  - All types: `SHSO_ALL_FY2026_REPORT_2026-06-06.docx`
  - Date format: `YYYY-MM-DD`
