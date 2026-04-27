# Project Impact
## Hours Saved · Cost Savings · Short and Mid-Term Value

---

## Baseline — Before This Project

Before the dashboard and pipeline existed, the operational reporting workflow required:

| Task | Manual Time per Cycle |
|---|---|
| Download 7 reports individually from portal | 30–45 minutes |
| Open and cross-reference reports in Excel | 2–3 hours |
| Check invoice status for each grant manually | 3–5 hours |
| Check QTR status for each grant manually | 2–4 hours |
| Build a summary view for supervisor review | 1–2 hours |
| Identify which grants needed attention | 1–2 hours |
| **Total per reporting cycle** | **9.5–17 hours** |

Reporting cycles occurred approximately **weekly** during active grant periods.

---

## After This Project

With the pipeline and dashboard operational:

| Task | Automated Time |
|---|---|
| Run `master_scrape.py` (unattended) | 10–25 minutes |
| Review dashboard and identify priorities | 15–30 minutes |
| Export filtered views as needed | 2–5 minutes |
| **Total per reporting cycle** | **~30–60 minutes** |

---

## Hours Saved Calculation

```
Hours Saved per Cycle = Baseline Hours − Automated Hours

Conservative:  9.5 − 1.0  = 8.5 hours saved per cycle
Realistic:    17.0 − 1.0  = 16.0 hours saved per cycle
```

```
Annual Cycles (estimated active reporting weeks) = 30

Annual Hours Saved (Conservative) = 8.5 × 30  = 255 hours
Annual Hours Saved (Realistic)    = 16.0 × 30 = 480 hours
```

---

## Cost Savings Estimate

Using a blended staff rate of $35–$55/hour (program analyst range, inclusive of benefits):

| Scenario | Hours Saved | Rate | Annual Savings |
|---|---|---|---|
| Conservative | 255 hours | $35/hr | $8,925 |
| Realistic | 480 hours | $45/hr | $21,600 |
| High estimate | 480 hours | $55/hr | $26,400 |

These figures represent staff time that can be redirected to programmatic work, grant oversight, and compliance review rather than manual data assembly.

---

## What Was Built (Development Value Estimate)

If this project had been contracted externally:

| Deliverable | Estimated External Cost |
|---|---|
| Portal export automation | $3,000–$5,000 |
| Grant scraper (concurrent, with dedup) | $4,000–$8,000 |
| Multi-source data pipeline and JSON merge | $2,000–$4,000 |
| Interactive branded HTML dashboard | $3,000–$6,000 |
| Scoring model and signal logic | $2,000–$4,000 |
| Full documentation suite | $1,500–$3,000 |
| **Total estimated external value** | **$15,500–$30,000** |

---

## Short-Term Impact (Season / Year 1)

- **Visibility:** For the first time, the full grant portfolio has a single operational view
- **Accountability:** Grants with missing invoices, late QTRs, or no supervisor are surfaced automatically
- **Decision support:** Priority scores reduce the time needed to decide where to focus attention
- **Audit readiness:** The JSON file serves as a timestamped snapshot of portfolio status at any point in time
- **Baseline establishment:** This is Season 1 of data. Every future season builds on this foundation.

---

## Mid-Term Impact (Years 2–3)

- **Trend analysis:** With multiple JSON snapshots, it becomes possible to track how grant health changes over time
- **Predictive signals:** Burn rate and projected final expenditure data supports earlier intervention before over/under-spend becomes critical
- **IT integration:** With IT partnership, the pipeline can be scheduled to run automatically rather than manually triggered
- **Web intake:** The current Excel-based intake process can be replaced with a web form that feeds directly into the data pipeline, eliminating manual entry errors
- **Portal API:** If the grants portal exposes an API, the scraper can be replaced with direct data pulls — faster, more reliable, and less fragile
- **Department expansion:** The same methodology can be applied to other grant portfolios or operational reporting needs across the agency

---

## Intangible Value

These benefits are real but harder to quantify:

- **Reduced cognitive load** — staff no longer need to hold portfolio state in their heads or maintain personal tracking spreadsheets
- **Consistency** — every team member works from the same data, updated at the same time
- **Institutional knowledge** — the documentation in this repository captures methodology that previously existed only as tribal knowledge
- **Scalability** — the pipeline and dashboard work whether there are 20 grants or 200
- **Trust** — stakeholders can see exactly how numbers are calculated, which builds confidence in reporting

---

## Hours Saved Building This vs. External Development

| Approach | Estimated Time |
|---|---|
| External contractor (no domain knowledge) | 155–355 hours |
| Internal build (with domain knowledge) | One focused development cycle |

Domain knowledge — knowing the portal, the grant types, the reporting cadence, and the operational context — is the single largest accelerator. No external contractor can replicate that without a significant discovery phase.
