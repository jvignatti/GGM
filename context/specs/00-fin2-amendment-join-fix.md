# Unit 00-FIN2: Amendment Financial Join Fix ✅ COMPLETE

## Goal

Fix `getFinancialRows()` so it resolves financial data for amended grants where the
SHSO Financial Report appends a `-1` sequence suffix to Document Name while the
Grant Summary keeps the base grant number.

## Root Cause

When a grant is formally amended, the Financial Report Document Name changes from
`EDUC-2026-BAADA-00031-GR1997` to `EDUC-2026-BAADA-00031-GR1997-1`. The grants
array, invoices, and QTRs all stay on the base number. Exact-match join fails silently,
showing $0 expended for affected grants.

## Confirmed Affected Grants (as of 2026-06-06)

- EDUC-2026-RutlandCSD-00003-GR2007 → financial DN: GR2007-1
- EDUC-2026-VDH-00023-MU0461 → financial DN: MU0461-1
- EDUC-2026-VDH-00012-MU0459 → financial DN: MU0459-1
- EDUC-2026-BAADA-00031-GR1997 → financial DN: GR1997-1 ($4,405 expended, $5,595 remaining)
- EE-2026-Vergennes-GR2001 → financial DN: GR2001-1 (4 EA rows, $31,820 expended)
- EE-2026-Washington-GR2027 → financial DN: GR2027-1 (4 EA rows, $34,756 expended)
- EE-2026-Winhall-GR2022 → financial DN: GR2022-1 (4 EA rows, $12,028 expended)

## Fix Applied

`getFinancialRows()` in `index.html`:

```javascript
function getFinancialRows(gn) {
  const g = str(gn).trim();
  return financial.filter(r => {
    const dn  = str(r["Document Name"]).trim();
    const rgn = str(r["Grant Number"]).trim();
    return dn === g || rgn === g || dn.startsWith(g + '-');
  });
}
```

## Verify

- [ ] EDUC-2026-BAADA-00031-GR1997 shows $4,405 expended, $5,595 remaining
- [ ] EE-2026-Vergennes shows correct multi-EA sum
- [ ] DRE single-EA grants unchanged
- [ ] No double-counting on amendment grants that also appear as `-1` in grants array
