# Design: Org View — Multi-select FY Filter + Column Sort

**Date:** 2026-06-07  
**File:** `index.html`  
**Status:** Approved — ready for implementation

---

## Summary

Two changes to the Organization (Grantee Intelligence) view:

1. **Multi-select FY filter** — the "View:" FY buttons currently allow only one year active at a time. Change to toggle behavior so multiple years can be selected simultaneously. Scores and metrics recalculate from only the selected years' grants.
2. **Org table column sort** — the Grant History table headers (Type, FY, Status, Budget Used) are styled as sortable but have no click handlers. Wire them up with ascending/descending toggle sort.

---

## State

Add two new module-level variables near `orgGrantsCache` / `orgNavList`:

```js
let giSelectedFYs = [];   // active FY selections; empty = All Years
let giSortCol     = null; // null = unsorted; values: 'type','fy','status','budget'
let giSortDir     = "asc";
```

Both are reset inside `showGI()` every time a new org is opened, so state does not bleed between orgs.

```js
// Inside showGI(), before calling renderGI:
giSelectedFYs = [];
giSortCol     = null;
giSortDir     = "asc";
```

---

## Change 1: Multi-select FY Filter

### Filtering logic

Replace the current single-string filter in `renderGI`:

```js
// Before:
const filtered = fyFilter
  ? orgGrants.filter(r => str(r["Sub Code"]||r["Year"]) === fyFilter)
  : orgGrants;

// After:
const filtered = giSelectedFYs.length
  ? orgGrants.filter(r => giSelectedFYs.includes(str(r["Sub Code"]||r["Year"])))
  : orgGrants;
```

The `fyFilter` parameter is kept in the function signature for backward compatibility but is no longer used.

### FY button behavior

**"All Years" button onclick:**
```js
giSelectedFYs = []; renderGI('${esc(orgName)}', orgGrantsCache, '${fyList.join(",")}', '');
```

**Individual FY button onclick:**
```js
(function(fy){ const i=giSelectedFYs.indexOf(fy); if(i===-1) giSelectedFYs.push(fy); else giSelectedFYs.splice(i,1); renderGI('${esc(orgName)}',orgGrantsCache,'${fyList.join(",")}',''); })('${fy}')
```

### Active state

- "All Years" is highlighted (`var(--accent)` background, white text) when `giSelectedFYs.length === 0`.
- Each FY button is highlighted when its year string is in `giSelectedFYs`.
- Multiple FY buttons can be highlighted simultaneously.
- "All Years" and any individual FY button are never highlighted at the same time.

### Score/metrics behavior

All scoring (Financial Utilization, Invoice Compliance, QTR Compliance, Program Consistency, Grant Completion) flows from `filtered`. With multiple FYs selected, scores aggregate across those years exactly as they would for "All Years" but scoped to the selection. No additional scoring changes needed.

---

## Change 2: Org Table Column Sort

### New function: `renderGITable()`

Extracts the table body render logic from `renderGI` into a standalone function. Called by both `renderGI` (initial render) and `giSort` (sort-triggered re-render).

```js
function renderGITable() {
  const filtered = giSelectedFYs.length
    ? window.orgGrantsCache.filter(r => giSelectedFYs.includes(str(r["Sub Code"]||r["Year"])))
    : window.orgGrantsCache;

  let rows = [...filtered];

  if (giSortCol) {
    rows.sort((a, b) => {
      let av, bv;
      if (giSortCol === 'type') {
        av = str(a["Grant Type"]).toLowerCase();
        bv = str(b["Grant Type"]).toLowerCase();
        return giSortDir === "asc" ? av.localeCompare(bv) : bv.localeCompare(av);
      }
      if (giSortCol === 'fy') {
        av = parseInt(a["Sub Code"]||a["Year"]||0);
        bv = parseInt(b["Sub Code"]||b["Year"]||0);
        return giSortDir === "asc" ? av - bv : bv - av;
      }
      if (giSortCol === 'status') {
        av = str(a["Status"]).toLowerCase();
        bv = str(b["Status"]).toLowerCase();
        return giSortDir === "asc" ? av.localeCompare(bv) : bv.localeCompare(av);
      }
      if (giSortCol === 'budget') {
        const getApv = r => num(r["Apv Total"]||r["Apv Program Total"]);
        const getPct = r => {
          const apv = getApv(r);
          const { actTotal } = getFinancialSummary(str(r["Grant Number"]));
          return (apv > 0 && actTotal > 0) ? actTotal / apv : null;
        };
        av = getPct(a); bv = getPct(b);
        // Nulls sort last in both directions
        if (av === null && bv === null) return 0;
        if (av === null) return 1;
        if (bv === null) return -1;
        return giSortDir === "asc" ? av - bv : bv - av;
      }
      return 0;
    });
  }

  // Render tbody — row template copied verbatim from the renderGI Grant History table map
  document.getElementById("gi-tbody").innerHTML = rows.map(r => { /* copy renderGI row template exactly — do not duplicate logic, just move it here */ }).join("");
}
```

The `<tbody>` gets `id="gi-tbody"` so `renderGITable` can target it directly.

### New function: `giSort(col)`

The `<thead>` is rendered once by `renderGI` and not re-rendered by `renderGITable`. Therefore `giSort` must update the `<th>` sort indicator classes directly on the DOM, using `data-gi-col` attributes as the selector target.

```js
function giSort(col) {
  if (giSortCol === col) giSortDir = giSortDir === "asc" ? "desc" : "asc";
  else { giSortCol = col; giSortDir = "asc"; }
  document.querySelectorAll("th[data-gi-col]").forEach(th => {
    th.classList.remove("sort-asc", "sort-desc");
    if (th.dataset.giCol === col) th.classList.add(giSortDir === "asc" ? "sort-asc" : "sort-desc");
  });
  renderGITable();
}
```

### Sortable `<th>` elements

The four sortable headers get `data-gi-col` for direct DOM targeting by `giSort`, and initial sort class rendered at template time:

```html
<th data-gi-col="type"   onclick="giSort('type')"   class="${giSortCol==='type'   ? (giSortDir==='asc'?'sort-asc':'sort-desc') : ''}">Type</th>
<th data-gi-col="fy"     onclick="giSort('fy')"     class="${giSortCol==='fy'     ? (giSortDir==='asc'?'sort-asc':'sort-desc') : ''}">FY</th>
<th data-gi-col="status" onclick="giSort('status')" class="${giSortCol==='status' ? (giSortDir==='asc'?'sort-asc':'sort-desc') : ''}">Status</th>
<th data-gi-col="budget" onclick="giSort('budget')" class="${giSortCol==='budget' ? (giSortDir==='asc'?'sort-asc':'sort-desc') : ''}">Budget Used</th>
```

Non-sortable headers (Grant Number, Project Title, Signal, Links) get `class="no-sort"` so the default `th:hover` accent color does not mislead.

The existing CSS `th.sort-asc::after` and `th.sort-desc::after` rules already render the ↑ ↓ indicators — no CSS changes needed.

---

## Files Changed

| File | Change |
|---|---|
| `index.html` | Add `giSelectedFYs`, `giSortCol`, `giSortDir` module-level vars |
| `index.html` | Reset all three in `showGI()` |
| `index.html` | Replace FY filter logic in `renderGI` |
| `index.html` | Replace FY button `onclick` / active-state template strings |
| `index.html` | Add `renderGITable()` function |
| `index.html` | Add `giSort(col)` function |
| `index.html` | Add `id="gi-tbody"` to Grant History `<tbody>` |
| `index.html` | Add `onclick` + sort classes to Type/FY/Status/Budget Used `<th>` elements |
| `index.html` | Add `class="no-sort"` to Grant Number, Project Title, Signal, Links `<th>` elements |

---

## Verify When Done

- [ ] Opening an org shows "All Years" highlighted; no individual year highlighted
- [ ] Clicking FY26 highlights FY26; "All Years" un-highlights; score recalculates for FY26 only
- [ ] Clicking FY25 while FY26 is active highlights both; score recalculates for FY25+FY26
- [ ] Clicking FY26 again (while FY25+FY26 active) deselects FY26; FY25 stays; score recalculates
- [ ] Clicking "All Years" clears all FY selections; score reverts to all-years aggregate
- [ ] Opening a different org resets to "All Years" with no active FY selections
- [ ] Clicking "Type" header sorts table by grant type A→Z; ↑ indicator appears
- [ ] Clicking "Type" again reverses to Z→A; ↓ indicator appears
- [ ] Clicking "FY" sorts numerically (oldest→newest or newest→oldest)
- [ ] Clicking "Budget Used" sorts by percentage; null (—) rows go to bottom in both directions
- [ ] Clicking "Status" sorts alphabetically
- [ ] Switching active FY selection while a sort is active — table re-renders sorted correctly
- [ ] Score panels and metrics row do NOT re-render when sort column is clicked (no flash)
- [ ] No console errors
