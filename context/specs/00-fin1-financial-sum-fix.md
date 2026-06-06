# Unit 00-FIN1: Fix Financial Multi-EA Sum

## Goal

Replace every `financial.find()` call in `index.html` with a sum across all matching
financial rows, so that EE grants with multiple expense accounts (OP, DUI, DD, Equipment)
display correct expenditure totals. After this fix, the dashboard remaining balance for
EE-2025-Chittenden-00005-GR1910-1 must change from $768,010.93 to ~$529,864.50.

## Design

No visual changes. Numbers will change. The financial panel layout stays identical.
The monthly table will show the summed values across all EA rows for the current month.

## Implementation

### Root cause

`financial` is an array of rows. Each row has a `Document Name` field equal to the
full grant number. EE grants have 4 rows (one per fund source: OP/DUI/DD/Equipment).
`financial.find()` at line 1247 of `index.html` returns only the first match.

The financial row `Grant Number` field is NOT the full grant number — it is a short
EA code like `GR1910`. Joins must use `Document Name`, not `Grant Number`.

### Change 1: Add `getFinancialRows(gn)` helper

Immediately after the `num()` / `str()` / `esc()` helpers (~line 1843), add:

```javascript
function getFinancialRows(gn) {
  return financial.filter(r => str(r["Document Name"]).trim() === str(gn).trim());
}

function getFinancialSummary(gn) {
  const rows = getFinancialRows(gn);
  const MONTHS = ["October","November","December","January","February","March",
                  "April","May","June","July","August","September"];
  let actTotal = 0, actMatch = 0;
  const monthly = {}, monthlyMatch = {};
  MONTHS.forEach(m => {
    let mv = 0, mm = 0;
    rows.forEach(r => {
      mv += num(r["Monthly " + m] || r["Monthly" + m] || 0);
      mm += num(r["Monthly " + m + " Match"] || r["Monthly" + m + "Match"] || 0);
    });
    monthly[m] = mv;
    monthlyMatch[m] = mm;
    actTotal += mv;
    actMatch += mm;
  });
  // Return a single synthetic row shape for backwards-compat with monthlyTable()
  const synth = rows[0] ? { ...rows[0] } : null;
  if (synth) {
    MONTHS.forEach(m => {
      synth["Monthly " + m] = monthly[m];
      synth["Monthly " + m + " Match"] = monthlyMatch[m];
    });
  }
  return { rows, synth, actTotal, actMatch };
}
```

### Change 2: Replace `finRow` lookup in `showDetail()`

Current (line ~1246):
```javascript
const finRow = financial.find(r => str(r["Grant Number"]) === gn || str(r["Document Name"]) === gn);
```

Replace with:
```javascript
const { synth: finRow, actTotal, actMatch } = getFinancialSummary(gn);
```

Remove the two lines below it that independently compute `actTotal` and `actMatch`
by iterating `MONTHS` over `finRow` — those are now provided by `getFinancialSummary`.

### Change 3: Replace `finRow` lookup in `renderGI()`

There are two places in `renderGI()` where financial data is looked up per grant
(lines ~1886 and ~1936). Both use:
```javascript
const finRow = financial.find(f => str(f["Grant Number"])===gn || str(f["Document Name"])===gn);
```
and then iterate months over `finRow`.

Replace each with:
```javascript
const { synth: finRow, actTotal: actual } = getFinancialSummary(gn);
```
Adjust the variable names to match what each block uses for its local sum.

### Change 4: Fix `financial.find()` in the grant history table inside `renderGI()`

Same pattern at ~line 2139. Replace with `getFinancialSummary(gn).synth`.

## Dependencies

None — this is a pure JavaScript change within `index.html`.

## Verify when done

- [ ] Open dashboard with `grants_data.json`. Navigate to EE-2025-Chittenden-00005-GR1910-1.
- [ ] Financial Summary shows Expended ≈ $344,739.50 (not $106,593.07).
- [ ] Remaining shows ≈ $529,864.50 (not $768,010.93).
- [ ] Monthly table shows the summed values across all EA rows (October should be
      ~$34,493 = $11,575 + $9,743 + $7,293 + $5,881).
- [ ] A DRE grant (single EA row) shows identical numbers before and after the fix.
- [ ] An EDUC grant shows identical numbers before and after the fix.
- [ ] Grantee Intelligence for Chittenden org shows updated score (financial utilization
      will change for past-FY grants).
- [ ] No console errors.
- [ ] `context/progress-tracker.md` updated: FIN-1 marked complete.
