# Dashboard Documentation
## SHSO Grants Operations Dashboard · Section Guide

---

## Overview

The dashboard is a single HTML file that loads entirely in your browser. It has no backend server, no login required, and no internet connection needed after the file is opened. All data stays on your computer.

When you first open the file, you will see a data upload area. Once you drop the `grants_data.json` file onto it, the dashboard populates instantly.

---

## Navigation

The dashboard is organized into tabs across the top:

| Tab | Purpose |
|---|---|
| **Overview** | Portfolio-level KPIs and health summary |
| **Grants** | Full grant list with filtering and scoring |
| **Invoices** | Invoice status across all grants |
| **Quarterly Reports** | QTR status across all grants |
| **Financial** | Budget vs. expenditure analysis |
| **Supervisors** | Grant-to-supervisor assignment view |
| **Signals** | Alert dashboard — grants flagged for attention |

---

## Section: Overview

The landing tab. Shows the state of the full portfolio at a glance.

**KPI Cards (top row):**

| Card | What It Shows |
|---|---|
| Total Grants | Count of all grants in the loaded dataset |
| Total Awards | Sum of all grant award amounts |
| Total Expenditures | Sum of all reported expenditures |
| Portfolio Utilization | Expenditures ÷ Awards as a percentage |
| Grants at Risk | Count of grants with Priority Score ≥ 50 |
| Active Signals | Total number of signal flags across all grants |

**Portfolio Health Bar:**
A stacked bar showing the distribution of grants by priority tier (Critical / High / Medium / Low).

**Signal Summary:**
A count of each signal type currently active across the portfolio. Useful for identifying systemic issues (e.g., many grants with late invoices simultaneously).

---

## Section: Grants

A filterable, sortable table of all grants.

**Columns:**

| Column | Description |
|---|---|
| Grant Number | Unique identifier |
| Grant Type | Category of grant |
| Agency | Receiving agency |
| Award Amount | Total awarded budget |
| Expenditures | Total reported spend |
| Utilization % | Expenditures ÷ Award |
| Priority Score | Composite urgency score (0–100) |
| Signals | Count of active alert conditions |
| End Date | Grant performance period end |
| Supervisor | Assigned SHSO supervisor |

**Filters available:**
- Grant type
- Agency
- Priority tier (Critical / High / Medium / Low)
- Supervisor
- Signal type
- Free-text search

**Sort:** Click any column header to sort ascending or descending.

**Export:** The Export CSV button downloads the currently filtered view.

---

## Section: Invoices

Shows all invoice records scraped from the grants portal, one row per invoice.

**Columns:**

| Column | Description |
|---|---|
| Grant Number | Parent grant |
| Invoice Number | Unique invoice identifier |
| Invoice Status | Current portal status |
| Invoice Period | Billing period covered |
| Scrape Timestamp | When this record was last captured |

**Filters:** Grant number, status, period.

**Color coding:**
- Green = Approved
- Yellow = Submitted / Pending
- Red = Missing or overdue

---

## Section: Quarterly Reports

Identical structure to the Invoices section, for QTR records.

**Key difference:** QTRs cover a quarter's worth of programmatic activity, not financial invoicing. A grant can have invoices on time and QTRs late, or vice versa.

---

## Section: Financial

Budget and expenditure analysis per grant.

**Columns:**

| Column | Description |
|---|---|
| Grant Number | Grant identifier |
| Award Amount | Total approved budget |
| Total Expenditures | Reported spend to date |
| Remaining Balance | Award − Expenditures |
| Utilization % | Expenditures ÷ Award |
| Indirect Cost | Calculated indirect cost amount |
| Burn Rate | Estimated monthly spend rate |
| Projected Final | Projected total spend by grant end |

**Flags:**
- Over 100% utilization → red flag
- Projected Final > Award Amount → warning
- Remaining Balance < 5% of Award → near-limit warning

---

## Section: Supervisors

Shows which SHSO supervisor is assigned to each grant.

**Use cases:**
- Confirm all grants have a supervisor assigned
- See workload distribution across supervisors
- Filter the grant list by supervisor for individual review sessions

**Missing supervisor flag:** Any grant without a supervisor assignment is highlighted. This is also a SIGNAL condition.

---

## Section: Signals

The operational alert center. Shows every grant that has at least one active signal condition.

**Signal types explained:**

| Signal | Meaning |
|---|---|
| `INV_LATE` | Invoice is more than 60 days past period end |
| `QTR_LATE` | QTR is more than 60 days past period end |
| `LOW_UTIL` | Spending significantly below expected pace |
| `HIGH_UTIL` | Spending at 90%+ of award with time remaining |
| `OVER_BUDGET` | Expenditures exceed award amount |
| `NEAR_END` | Grant ends within 60 days |
| `NO_ACTIVITY` | No invoices or QTRs recorded and grant is active |
| `NO_SUPERVISOR` | No supervisor assigned |

**How to use this section:**
Sort by Signal Count (descending) to see the highest-risk grants first. Use the signal type filter to focus on one issue at a time across the portfolio.

---

## Data Load Indicator

The top of every tab shows the data timestamp from the JSON file — the exact date and time the pipeline was last run. This tells you how current your data is.

If the timestamp is more than one week old, consider re-running the pipeline before making decisions based on the dashboard.

---

## What the Dashboard Cannot Do

- It cannot refresh data automatically — you must re-run the pipeline and reload the JSON
- It cannot write back to the portal or any system
- It cannot send notifications or emails
- It does not store any data between sessions — every browser close clears the loaded data
- It cannot access the portal directly — it only reads the JSON you provide
