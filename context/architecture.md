# Architecture — GGM Chrome Extension

## Stack

| Layer | Technology | Role |
|---|---|---|
| Extension shell | Manifest V3 Chrome Extension | Packaging, permissions, CSP |
| Background | Service Worker (`background.js`) | Scrape orchestration, IndexedDB writes |
| Dashboard UI | Extension full-page (`dashboard.html` + JS) | Main grant directory view |
| Side panel | `sidepanel.html` + JS | Per-grant signal while browsing portal |
| Content script | `content.js` | Reads current portal URL/grant number |
| Storage — scraped | IndexedDB (`ggm-db`) | All portal-scraped data |
| Storage — user | `chrome.storage.local` | SHSO scores, UI prefs, never scrape-overwritten |
| Styling | Vanilla CSS with CSS custom properties | Design tokens, dark mode |
| No external deps | — | No CDN, no npm packages in extension bundle |

## System Boundaries

```
Extension popup / full-page
  └── dashboard.html
        reads IndexedDB (scraped data)
        reads chrome.storage.local (user prefs, SHSO scores)
        writes chrome.storage.local (user prefs, SHSO scores)

Background service worker
  └── background.js
        drives scraping via fetch() against grants.vermont.gov
        reads/writes IndexedDB (scraped data)
        NEVER touches chrome.storage.local user data

Content script
  └── content.js (injected into grants.vermont.gov pages)
        reads current URL and page DOM for grant number
        sends message to side panel
        NEVER accesses storage directly

Side panel
  └── sidepanel.html
        reads IndexedDB (read-only)
        reads chrome.storage.local (read-only)
        receives messages from content script
```

## Storage Model

### IndexedDB — `ggm-db`

Seven object stores, keyed as shown:

| Store | Key | Contents |
|---|---|---|
| `grants` | `Grant Number` | Grant metadata (all fields from Grant Summary) |
| `invoices` | `Invoice Number` | Scraped invoices, linked by `Parent Grant Number` |
| `qtrs` | `QTR Number` | Scraped QTRs, linked by `Parent Grant Number` |
| `indirect` | `Grant Number` | IC rates and amounts per grant |
| `financial` | composite: `[Document Name, EA]` | Monthly expenditure rows — multiple per grant |
| `supervisors` | `Document Name` | Enforcement metrics, linked by invoice number |
| `meta` | `key` | `last_scraped` timestamp, `scrape_version`, etc. |

### chrome.storage.local — user data

```
shso_score_{orgKey}        number 1-10   SHSO manual score per org
shso_theme                 "light"|"dark"
shso_legacy_visible        boolean
```

`orgKey` is a stable hash of the org name (SHA-1 first 8 chars), not the raw string,
to avoid key corruption from special characters.

## Portal HTML Structure (grants.vermont.gov)

This is the authoritative map from verified scraping sessions.

### Grant Landing Page
```
auxNavSubWindow[data-aux-nav-parent-id="5000008"]
  .auxNavSubRow > a[aria-label="InvoiceNumber: Status"]
  .auxNavSubRow (no <a>) → text = period label

auxNavSubWindow[data-aux-nav-parent-id="5000009"]
  .auxNavSubRow > a[aria-label="QTRNumber: Status"]
```

**Left sidebar** also contains links to:
- `Budget Summary {TYPE}` — approved budget totals (NOT monthly actuals)
- `Indirect Costs` — navigates to the Indirect Costs form page

### Indirect Costs Page (per grant)
Form inputs with portal-generated IDs (`pgf...`). Scrape by text-proximity to labels:
- "Indirect Cost Rate Charged on this Grant" → adjacent input = ICR %
- "Sum of Indirect Cost Base Categories" → adjacent value = MTDC
- "Total Indirect Costs" → adjacent value = total IC $
- Budget cost category inputs (Salaries, Supplies, etc.) — in the Budget Costs table

### EE Invoice Sub-Reports
Each EE invoice URL has navigation to sub-report pages:
- **OP Report** — patrol hours, vehicles stopped, DUI arrests, checkpoints, violations
- **DUI Report** — same structure, DUI-budget line items
- **DD Report** — same structure, DD-budget line items
These pages contain all enforcement metrics that appear in the Supervisors Report.

### DRE Invoice Pages
- **Financial Reimbursement** — current-period expenditures by budget category
- No per-invoice enforcement sub-reports for DRE. DRE metrics come only from the
  aggregate Supervisors Report export.

### QTR Number Vendor Bug
The portal emits `DREQTR1-YYYY-...` (no dash) for some DRE QTRs. The scraper must
normalize: if `qtr_num.startsWith("DRE") && !qtr_num.startsWith("DRE-")`, prepend
`DRE-` and drop the first 3 chars. This normalization is required on write in the
scraper; the dashboard also re-applies it defensively on read.

## Invariants

These rules must never be violated. They encode bugs found through live testing.

**INV-1: Financial data is always summed across all matching rows.**
`financial.find()` returns one row. For EE grants, there are typically 4 rows
(OP, DUI, DD, Equipment — each with its own EA code). The dashboard must collect
ALL rows where `Document Name === grantNumber` and sum monthly amounts across all
of them. Using only the first row underreports expenditures by up to $238k per grant.

**INV-2: Scraped data and user data live in separate stores that never cross.**
The background scraper writes only to IndexedDB. SHSO scores and UI preferences live
only in chrome.storage.local. A full re-scrape must never touch chrome.storage.local.

**INV-3: Timed-out grants carry forward their previous data.**
If a grant's scrape returns an error or timeout, the result sentinel is `null` (not
`[]`). The merge step must fall back to the existing IndexedDB rows for that grant.
Writing an empty array is silent data loss.

**INV-4: Invoice compliance counts exclude "Invoice In Process" status.**
Signal calculation filters out invoices with status `"Invoice In Process"`. The
calendar display must do the same — showing a ✅ for an In-Process invoice while
the signal flags it as missing creates a visible contradiction for the user.

**INV-5: QTR4 is the last QTR that counts for compliance. QTR5 (Final Report)
must not be required for Grant Completion in the Grantee Score.**
QTR5 was removed from all compliance counting. The Grant Completion dimension of
the Grantee Score must match — requiring QTR5 silently lowers scores for completed
grants that lack a Final Report.

**INV-6: Grant numbers with trailing spaces are real data from the portal.**
The portal generates grant numbers like `EE-2026-Vergennes -00003` with a trailing
space in the org component. All joins must use `.trim()` on both sides before
comparing strings.

**INV-7: The extension never stores or transmits credentials.**
The extension relies on the browser's existing authenticated session cookies. No
username, password, or session token is ever read, stored, or logged.

**INV-8: Two distinct EE organizations share the same "Bennington" org name in the Grant Summary export.**
The portal's Grant Summary truncates both "Bennington County Sheriff's Department" and
"Town of Bennington" to `Organization = "Bennington"` in new GEARS (FY25+FY26).
Affected grants: GR1992, GR2011 (FY26) and GR1897-1, GR1906 (FY25).
Legacy data correctly uses the full org names. Current impact: Grantee Intelligence
groups both orgs into one "Bennington" card, mixing scores, invoices, and financials.
SHSO Score saved for "Bennington" applies to both entities incorrectly.
Not fixing now — EE grants are out of scope for current work. Revisit when
EE Grantee Intelligence is needed. Correct org names must come from the portal
or a manual mapping table; the Grant Summary export cannot be trusted for this.

**INV-9: IC panel must require both a charged rate AND approved dollar amount.**
Some grants have `ICR Charged on Grant > 0` (rate was entered on the IC form) but
`Total Indirect Costs = $0` because the dollar approval never happened. The IC panel
must gate on `charged > 0 AND totalIC > 0`. Using rate alone causes the panel to
render with no meaningful content. Confirmed on EE-2026-St.Johnsb-GR2020 and
EE-2026-Woodstock-GR2023 (15% rate entered, $0 approved amount).

**INV-10: Some grants have no match requirement — Apv Match Total = $0 is valid data.**
The Grant Summary export sets `Apv Match Total = $0.00` for grants that carry no
match obligation (e.g. EDUC-2026-Milton-00018-GR2015). The dashboard correctly hides
the Match Approved / Match Actual / Match Remaining row when `apvMatch === 0`. This
is not a bug — do not attempt to infer a match amount from other fields for these grants.
Confirmed vendor behavior: $0 match grants exist across EDUC type at minimum.

**INV-9: Financial row lookup uses startsWith fallback for amended grants.**
When a grant is formally amended, the SHSO Financial Report switches its Document
Name to `<GrantNumber>-1` (e.g. `EDUC-2026-BAADA-00031-GR1997-1`) while the Grant
Summary, invoices, and QTRs stay on the base number (`EDUC-2026-BAADA-00031-GR1997`).
`getFinancialRows(gn)` must check: exact Document Name match OR exact Grant Number
field match OR Document Name starts with `gn + '-'`. Confirmed on 7 amendment grants
as of 2026-06-06: EDUC-RutlandCSD-GR2007, VDH-MU0461, VDH-MU0459, BAADA-GR1997,
EE-Vergennes-GR2001, EE-Washington-GR2027, EE-Winhall-GR2022. This pattern will
recur on every future amendment.

## Data Join Keys (confirmed from live data)

| Source | Join to grants | Field name | Notes |
|---|---|---|---|
| invoices | `Parent Grant Number` | exact match | trailing-space safe with .trim() |
| qtrs | `Parent Grant Number` | exact match | QTR normalization applied first |
| financial | `Document Name` | multiple rows — SUM all matches | Grant Number field = short code (e.g. GR1910), not full grant number |
| indirect | `Grant Number` | one row per grant | |
| supervisors | `Document Name` = Invoice Number | chain: grant→invoices→supervisors | |

## Invoice Number Format (all types confirmed)

```
TYPE-INV-MonAbbrev-YYYY-OrgShort-NNNNN
  split("-")[2] = month abbreviation ("Oct", "Nov", ...)

Supplemental (EXT):
TYPE-INV-Ext-YYYY-OrgShort-NNNNN
  split("-").some(s => s.toLowerCase() === "ext") = true
```

## QTR Number Format

```
TYPE-QTR1-YYYY-OrgShort-NNNNN   (FY26 normalized form)
DREQTR1-YYYY-OrgShort-NNNNN     (portal vendor bug — normalize on scrape)
  split("-")[1] = QTR label ("QTR1", "QTR2", ...)
```

## Grant Number Format

```
TYPE-YYYY-OrgShort-NNNNN-EACODE       standard
TYPE-YYYY-OrgShort-NNNNN-EACODE-1     amendment (suffix -1, -2, etc.)
TYPE-YYYY-OrgShort-NNNNN-EACODE-A1    amendment (suffix -A1, -A2, etc.)
DD-YYYY-OrgShort-NNNNN-EACODE         legacy old GEARS (type DD, not EE)
```

Amendment detection regex: `/-(?:A?\d+)$/` — matches `-1`, `-2`, `-A1`, `-A2`.
Does NOT match standard grant numbers ending in EA codes like `-GR1910` or `-LU0242`.
