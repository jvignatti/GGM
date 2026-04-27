# SHSO Grants Operations Dashboard
### Highway Safety Office · Vermont · Season 2025–2026

[![Status](https://img.shields.io/badge/status-active-brightgreen)]()
[![Data](https://img.shields.io/badge/data-internal--only-red)]()
[![Dashboard](https://img.shields.io/badge/dashboard-HTML--standalone-blue)]()

---

## What This Is

An interactive, standalone HTML dashboard built to give the Highway Safety Office a real-time operational view of its grants portfolio — without relying on external software, subscriptions, or live database connections.

The dashboard consolidates data from multiple internal reporting sources into a single interface. It supports filtering, scoring, signal detection, risk prioritization, and export — all running locally in a browser with no server required.

**This repository contains:**
- The HTML dashboard file (open in any browser)
- Full methodology and mathematical documentation
- Script descriptions (no code)
- How-to-use guides
- Data safety and privacy documentation

**This repository does not contain:**
- Any real grant data
- Employee names, addresses, or contact information
- Internal URLs, system credentials, or API references
- Source code for the automation scripts
- The JSON data file (must remain internal)

---

## Quick Start

1. Obtain `grants_data.json` from your authorized internal source
2. Open `dashboard.html` in Chrome, Edge, or Safari
3. Click **Load Data** and drop the JSON file onto the upload area
4. The dashboard loads instantly — nothing is sent to any server

Full instructions → [`docs/HOW_TO_USE.md`](docs/HOW_TO_USE.md)

---

## Repository Structure

```
shso-grants-dashboard/
│
├── index.html                  ← Interactive dashboard (open in browser / GitHub Pages)
│
├── README.md                   ← This file
│
└── docs/
    ├── METHODOLOGY.md          ← Full data methodology
    ├── MATH.md                 ← Equations, scoring, signal logic
    ├── DASHBOARD.md            ← Dashboard sections explained
    ├── SCRIPTS.md              ← Script inventory (names + descriptions)
    ├── HOW_TO_USE.md           ← User guide
    ├── IMPACT.md               ← Hours saved, cost savings, project value
    ├── DATA_SAFETY.md          ← Privacy and data handling policy
    └── DISCLAIMER.md           ← Legal and organizational disclaimer
```

---

## Project Background

The SHSO grants portfolio involves dozens of active grants across multiple agencies, each with invoices, quarterly reports, financial summaries, and supervisor assignments. Before this project, operational visibility required manually cross-referencing multiple Excel reports, with no unified view of status, risk, or priority.

This project built a pipeline that:
1. Automates the export of existing reports from the grants portal
2. Scrapes invoice and quarterly report status for each grant
3. Merges all sources into a single structured JSON
4. Renders that JSON into an interactive dashboard

The result is a tool that surfaces what needs attention, scores risk, and tracks financial exposure — in a format any team member can open without technical knowledge.

---

## Data Safety Summary

The HTML dashboard file is safe to share publicly. It contains no data.

The `grants_data.json` file contains sensitive operational and financial data belonging to the State of Vermont. It must never be uploaded to GitHub or shared outside the organization.

Full policy → [`docs/DATA_SAFETY.md`](docs/DATA_SAFETY.md)

---

## Documentation Index

| Document | Purpose |
|---|---|
| [`METHODOLOGY.md`](docs/METHODOLOGY.md) | How data is collected, merged, and processed |
| [`MATH.md`](docs/MATH.md) | All equations, scores, signals, and risk calculations |
| [`DASHBOARD.md`](docs/DASHBOARD.md) | Every section of the dashboard explained |
| [`SCRIPTS.md`](docs/SCRIPTS.md) | Script inventory with descriptions |
| [`HOW_TO_USE.md`](docs/HOW_TO_USE.md) | Step-by-step user guide |
| [`IMPACT.md`](docs/IMPACT.md) | Time saved, cost savings, project value |
| [`DATA_SAFETY.md`](docs/DATA_SAFETY.md) | Privacy, data governance, security policy |
| [`DISCLAIMER.md`](docs/DISCLAIMER.md) | Legal and organizational disclaimer |

---

*Built and maintained by the SHSO Operations Team · Vermont*
*All data belongs to the State of Vermont*
