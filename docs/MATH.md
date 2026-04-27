# Mathematical Methodology
## Scoring · Signal Detection · Risk Logic · Impact Calculations

---

## Overview

The dashboard computes several derived metrics from the raw data. This document defines every equation, scoring model, signal rule, and calculation used. All formulas are applied client-side in the browser when the JSON file is loaded.

---

## 1. Financial Metrics

### 1.1 Budget Utilization Rate

Measures how much of the awarded grant budget has been spent.

```
Utilization Rate = (Total Expenditures / Total Award Amount) × 100
```

- **< 25%** → Low utilization — may indicate slow start or inactivity
- **25% – 75%** → Normal range
- **> 90%** → High utilization — may need budget amendment soon
- **> 100%** → Over-expenditure — critical flag

### 1.2 Remaining Balance

```
Remaining Balance = Total Award Amount − Total Expenditures
```

### 1.3 Indirect Cost Amount

```
Indirect Cost Amount = Direct Cost Base × Indirect Cost Rate
```

Where `Indirect Cost Rate` is the approved rate for the grant's agency.

### 1.4 Burn Rate (Monthly)

Estimated monthly spend rate based on elapsed grant period:

```
Burn Rate = Total Expenditures / Elapsed Months
```

Where:
```
Elapsed Months = (Current Date − Grant Start Date) in months
```

### 1.5 Projected Final Expenditure

```
Projected Final = Burn Rate × Total Grant Duration in Months
```

Used to estimate whether the grant will under-spend or over-spend by end date.

---

## 2. Invoice Scoring

Each grant receives an invoice health score based on its invoice portfolio.

### 2.1 Invoice Submission Rate

```
Invoice Submission Rate = (Submitted Invoices / Expected Invoices) × 100
```

Where:
```
Expected Invoices = Elapsed Grant Months / Invoice Frequency
```
(Invoice frequency is typically monthly or quarterly depending on grant type.)

### 2.2 Invoice Status Score

Each invoice status carries a weight:

| Status | Weight |
|---|---|
| Approved | 1.0 |
| Submitted | 0.7 |
| Pending | 0.4 |
| Missing / Not Filed | 0.0 |

```
Invoice Score = (Σ Status Weights) / Total Expected Invoices × 100
```

Range: 0–100. Higher is better.

### 2.3 Invoice Lag (Days)

```
Invoice Lag = Current Date − Most Recent Invoice Period End Date
```

- **< 30 days** → Current
- **30–60 days** → Watch
- **> 60 days** → Late
- **> 90 days** → Critical

---

## 3. Quarterly Report (QTR) Scoring

Mirrors invoice scoring with QTR-specific logic.

### 3.1 QTR Submission Rate

```
QTR Submission Rate = (Submitted QTRs / Expected QTRs) × 100
```

### 3.2 QTR Status Score

| Status | Weight |
|---|---|
| Approved | 1.0 |
| Submitted | 0.7 |
| Pending | 0.4 |
| Missing | 0.0 |

```
QTR Score = (Σ Status Weights) / Total Expected QTRs × 100
```

### 3.3 QTR Lag

```
QTR Lag = Current Date − Most Recent QTR Period End Date
```

Thresholds: same as Invoice Lag.

---

## 4. SIGNAL Component

The SIGNAL is a composite alert system that flags grants requiring attention. Each signal condition is evaluated independently and can fire simultaneously.

### Signal Conditions

| Signal | Code | Trigger Condition |
|---|---|---|
| Invoice Late | `INV_LATE` | Invoice Lag > 60 days |
| QTR Late | `QTR_LATE` | QTR Lag > 60 days |
| Low Utilization | `LOW_UTIL` | Utilization Rate < 20% AND Elapsed Months > 3 |
| High Utilization | `HIGH_UTIL` | Utilization Rate > 90% |
| Over Budget | `OVER_BUDGET` | Utilization Rate > 100% |
| Near End Date | `NEAR_END` | Days to Grant End < 60 |
| No Activity | `NO_ACTIVITY` | No invoices AND no QTRs AND Elapsed Months > 2 |
| Missing Supervisor | `NO_SUPERVISOR` | Supervisor field is blank |

### Signal Count

```
Signal Count = Number of active signal conditions for a grant
```

A grant with Signal Count = 0 requires no immediate action.
A grant with Signal Count ≥ 3 is flagged as high priority.

---

## 5. Priority Score

A single composite number (0–100) representing the urgency of attention needed for a grant. Higher = more urgent.

```
Priority Score = (W1 × Financial Risk) + (W2 × Compliance Risk) + (W3 × Timeline Risk)
```

Where:

**Financial Risk (0–100):**
```
Financial Risk = MAX(
    Utilization Rate − 90,   ← over-spend pressure
    (20 − Utilization Rate)  ← under-spend pressure
) normalized to 0–100
```

**Compliance Risk (0–100):**
```
Compliance Risk = 100 − MIN(Invoice Score, QTR Score)
```

**Timeline Risk (0–100):**
```
Timeline Risk = MAX(0, (60 − Days to End Date) / 60) × 100
```

**Weights:**

| Component | Weight (W) |
|---|---|
| Financial Risk | 0.35 |
| Compliance Risk | 0.40 |
| Timeline Risk | 0.25 |

**Final:**
```
Priority Score = (0.35 × Financial Risk) + (0.40 × Compliance Risk) + (0.25 × Timeline Risk)
```

Rounded to nearest integer. Range: 0–100.

### Priority Tiers

| Score | Tier | Action |
|---|---|---|
| 75–100 | 🔴 Critical | Immediate attention required |
| 50–74  | 🟠 High | Review this week |
| 25–49  | 🟡 Medium | Monitor closely |
| 0–24   | 🟢 Low | On track |

---

## 6. Portfolio-Level Aggregations

### 6.1 Total Awards

```
Total Awards = Σ Award Amount for all active grants
```

### 6.2 Total Expenditures

```
Total Expenditures = Σ Total Expenditures for all active grants
```

### 6.3 Portfolio Utilization

```
Portfolio Utilization = (Total Expenditures / Total Awards) × 100
```

### 6.4 Grants at Risk Count

```
Grants at Risk = COUNT(grants WHERE Priority Score ≥ 50)
```

### 6.5 Average Priority Score

```
Avg Priority = Σ Priority Scores / Total Active Grants
```

---

## 7. Hours Saved Calculation

See [`IMPACT.md`](IMPACT.md) for the full hours-saved and cost-savings methodology.
