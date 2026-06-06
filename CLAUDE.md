## Application Building Context

Read the following files in order before implementing or making any architectural decision:

1. `context/project-overview.md` — product definition, goals, features, and scope
2. `context/architecture.md` — system structure, boundaries, storage model, and invariants
3. `context/ui-context.md` — theme, colors, typography, and component conventions
4. `context/code-standards.md` — implementation rules and conventions
5. `context/ai-workflow-rules.md` — development workflow, scoping rules, and delivery approach
6. `context/progress-tracker.md` — current phase, completed work, open questions, and next steps

Update `context/progress-tracker.md` after each meaningful implementation change.

If implementation changes the architecture, scope, or standards documented in the
context files, update the relevant file before continuing.

---

## Project Snapshot — Read This First

**What it is:** A local-only Manifest V3 Chrome extension (in design) that replaces
a Python scraper + standalone HTML dashboard. It rides the user's existing logged-in
session on grants.vermont.gov (Vermont SHSO grants portal), scrapes all grant data
in-browser, stores it locally in IndexedDB, and renders the GGM dashboard inside
the extension. A side panel shows a grant's Signal/risk while browsing the portal.

**What it is not:** A server. A cloud app. A tool that stores credentials. A tool
that sends any data anywhere.

**Current state (2026-06-06):**
- The old pipeline (`master_scrape.py` + `index.html` + `grants_data.json`) is
  operational and being actively maintained while the extension is designed.
- `index.html` is the live dashboard — stable, all known bugs fixed.
- The Chrome extension has NOT been started yet.
- A full pipeline re-run is required before building the extension (see progress-tracker.md).

**Data is sensitive Vermont financial data — no server, no cloud, no telemetry.**

---

## Critical Rules to Know Before Writing Any Code

1. **Financial data must sum ALL matching rows per grant.** Use `getFinancialRows(gn)`
   then sum. Never use `.find()` on the financial array. EE grants have 4 rows.
   See INV-1 in architecture.md.

2. **Amended grants have a `-1` suffix in the Financial Report but not in the grants
   array.** `getFinancialRows()` handles this with `startsWith(gn + '-')`.
   See INV-11 in architecture.md.

3. **Approved budget = `Apv Total` (federal award only).** `Apv Program Total`
   includes match and must not be used as the budget figure.

4. **IC panel shows only when `ICR Charged on Grant > 0 AND Total Indirect Costs > 0`.**
   A rate without an approved dollar amount is a draft, not an approval.

5. **All string joins use `.trim()` on both sides.** Grant numbers contain trailing
   spaces from the portal (e.g., `"Vergennes "`). See INV-6 in architecture.md.

6. **SHSO manual scores and UI preferences live in `chrome.storage.local` only.**
   The scraper never touches that store. Scraped data lives in IndexedDB only.
   These two stores must never be mixed.

7. **Read `context/progress-tracker.md` for the full list of open bugs** (SIG-1,
   GI-1, ORG-1) before starting any new work — they may affect what you're building.

---

## File Locations

```
GGM/GGM/                        ← git repo root (this folder)
├── CLAUDE.md                   ← this file
├── index.html                  ← live standalone dashboard
├── context/
│   ├── project-overview.md
│   ├── architecture.md         ← read this for portal HTML map + all invariants
│   ├── code-standards.md
│   ├── ai-workflow-rules.md
│   ├── ui-context.md
│   ├── progress-tracker.md     ← start here to understand current state
│   └── specs/
│       ├── 00-build-plan.md    ← 18-unit Chrome extension build plan
│       ├── 00-fin1-*.md        ← complete
│       └── 00-fin2-*.md        ← complete
└── docs/                       ← public methodology docs (no code)

C:/Users/JC/Desktop/SOV/SHSO/scripts/   ← NOT in git repo
├── master_scrape.py            ← live pipeline script
├── grants_data.json            ← current data (6.1MB, generated 2026-06-04)
├── old_gears_data.json         ← legacy FY19-FY24 data (read-only)
└── list_to_work.xlsx           ← 58 active grant numbers + landing page URLs
```
