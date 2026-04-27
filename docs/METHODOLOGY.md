# Methodology
## SHSO Grants Operations Dashboard · Data Pipeline Documentation

---

## Overview

The dashboard is powered by a multi-step data pipeline that transforms raw portal exports and scraped data into a unified, structured JSON file. This document describes each step of that process — what data is collected, how it is processed, and how it flows into the dashboard.

No proprietary data, credentials, or internal system references are included in this document.

---

## Pipeline Architecture

```
STEP 1          STEP 2              STEP 3          STEP 4
Portal      →   Automated      →   Merge &     →   Dashboard
Exports         Scraper            Normalize        Rendering
(Excel/CSV)     (Invoices,         (JSON)           (HTML)
                QTRs)
```

---

## Step 1 — Report Exports

Seven structured reports are exported from the grants management portal. Each report covers a distinct operational domain:

| Report | Domain |
|---|---|
| Grant Summary | Core grant metadata — number, type, agency, status, dates, amounts |
| SHSO Financial Report | Budget vs. expenditure tracking per grant |
| Indirect Cost | Indirect cost rates and calculations |
| Supervisors Report | Grant-to-supervisor assignment mapping |
| Grants Invoices | Invoice records per grant (partially populated by Step 2) |
| Grants QTR | Quarterly report records per grant (partially populated by Step 2) |

All exports are saved as CSV or Excel files in a designated local output folder before processing continues.

---

## Step 2 — Automated Scraper

For each grant in the portfolio, the scraper navigates to the grant detail page and extracts:

**Invoice data per grant:**
- Invoice number
- Invoice status (Submitted, Approved, Pending, etc.)
- Invoice period
- Scrape timestamp

**Quarterly report (QTR) data per grant:**
- QTR number
- QTR status
- QTR period
- Scrape timestamp

The scraper processes grants concurrently (configurable concurrency level) to reduce total runtime. Grants that fail due to portal timeouts are logged with their error and can be retried independently.

**Deduplication logic:** Before writing, the scraper checks existing CSV files and only appends records that do not already exist based on a defined comparison key set. This prevents duplicate entries on reruns.

---

## Step 3 — JSON Merge

All six data sources are read and merged into a single structured JSON object:

```
grants_data.json
├── generated         ← ISO timestamp of when the file was created
├── grants[]          ← Grant Summary rows
├── indirect[]        ← Indirect Cost rows
├── financial[]       ← SHSO Financial Report rows
├── invoices[]        ← All invoice records (from scraper)
├── qtrs[]            ← All QTR records (from scraper)
└── supervisors[]     ← Supervisor assignments
```

Each array contains the full row data from its source, normalized to remove null values (replaced with empty strings for consistency).

**Key design decisions:**
- All data types are preserved as strings in the JSON to prevent type coercion errors across systems
- Timestamps are stored in ISO 8601 format
- The JSON is human-readable (indented) to allow manual inspection when needed
- File size is typically 200KB–2MB depending on portfolio size

---

## Step 4 — Dashboard Rendering

The dashboard HTML file reads the JSON client-side when the user drops or selects the file. All processing — filtering, scoring, signal detection, calculations — happens in the browser using JavaScript. No data is transmitted anywhere.

The dashboard joins data across the six arrays using grant number as the primary key.

---

## Data Sources Summary

| Source | Update Frequency | Method |
|---|---|---|
| Grant Summary | Per run | Automated export |
| Financial Report | Per run | Automated export |
| Indirect Cost | Per run | Automated export |
| Supervisors | Per run | Automated export / manual xlsx |
| Invoices | Per run | Automated scraper |
| QTRs | Per run | Automated scraper |

---

## Known Limitations

- **Portal timeouts:** The grants portal occasionally returns timeout errors for individual grants. These are logged and can be retried. They do not affect the rest of the dataset.
- **Manual email submissions:** Some invoice or QTR submissions occur via email outside the portal. These are not captured by the scraper and represent a known data gap.
- **Supervisor report format:** The supervisors report may be in either CSV or Excel format depending on the export. Both are handled, but column names must remain consistent.
- **Static snapshot:** Each JSON file is a point-in-time snapshot. The dashboard does not auto-refresh. Re-run the pipeline to get current data.
- **Grant number as join key:** All cross-source joins depend on grant number being consistent across all reports. Any formatting inconsistency in grant numbers will cause join failures.

---

## Iteration History

### v1 — Manual Excel Baseline
All reporting done manually by cross-referencing multiple Excel files. No unified view. Estimated 6–10 hours per reporting cycle.

### v2 — Export Automation
Automated the export of the seven portal reports. Eliminated manual download steps. Output: structured CSV/Excel files in a consistent format.

### v3 — Scraper Addition
Added the invoice and QTR scraper. For the first time, invoice and quarterly report status was captured systematically for all grants rather than checked individually.

### v4 — JSON Merge
Combined all data sources into a single normalized JSON. Enabled the dashboard to be built as a standalone file rather than requiring live database access.

### v5 — HTML Dashboard (Current)
Built the interactive dashboard. Added scoring, signal detection, risk prioritization, filtering, and CSV export. The full pipeline became executable as a single command.

### v6 (Planned)
Web-based intake form, real-time dashboard refresh, IT-compliant deployment, potential integration with grants portal API if made available.
