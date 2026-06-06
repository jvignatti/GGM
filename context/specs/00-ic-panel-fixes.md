# Unit 00-IC: Indirect Cost Panel Fixes ✅ COMPLETE

## Fix 1 — IC Panel False-Positive (commit 5e789b1)

### Problem
`icPanel()` showed when `ICR Charged on Grant > 0` (rate alone). Two EE grants had
a 15% rate entered on the IC form but `Total Indirect Costs = $0` — the dollar
approval was never completed. Panel rendered with rate % displayed but all dollar
amounts showing "—". Misleading: looked like IC was active when it wasn't.

### Affected grants
- `EE-2026-St. Johnsb-00022-GR2020` — 15% rate, $0 total IC
- `EE-2026-Woodstock-00010-GR2023` — 15% rate, $0 total IC

### Fix
```javascript
// Before
if (charged <= 0) return "";

// After
if (charged <= 0 || totalIC <= 0) return "";
```
Panel now requires both a non-zero rate AND a non-zero approved dollar amount.

---

## Fix 2 — IC Category Checkmarks (commit 2d7a433)

### Problem
`checkRow()` showed ✅ under "Has Cost" for any budget category with spending,
regardless of whether IC was applied to it. For grants where only Salaries is in
the IC base (most grants), Travel, Supplies, Other Operating, Equipment all showed
✅ with no IC Amount — implying IC was applied when it wasn't.

Confirmed on VDH-00012 (screenshot): Supplies, Travel, Other Operating, Equipment
all ✅ but IC Amount = "—" for all of them. Only Salaries ($56,694) was in IC base.

### Fix
- ✅ now shows only when `indNum > 0` (category is IN the IC base with approved amount)
- Column header renamed "Has Cost" → "In IC Base"

```javascript
// Before
const hasCost = num(costVal) > 0;
// ✅ shown when hasCost

// After
const inBase = indNum !== null && indNum > 0;
// ✅ shown when inBase
```

### Invariant added
INV-9 in architecture.md: IC panel requires both charged rate and approved dollar
amount. A draft rate without dollar approval is not an active IC arrangement.
