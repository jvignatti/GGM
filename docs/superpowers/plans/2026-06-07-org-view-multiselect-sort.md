# Org View — Multi-select FY Filter + Column Sort — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add multi-select FY filter buttons and clickable column sort to the Grantee Intelligence org view in `index.html`.

**Architecture:** All changes are confined to one file (`index.html`). Three module-level state variables drive both features. The existing `renderGI()` FY filter is replaced with a `giSelectedFYs` array toggle. The Grant History table body is extracted into a new `renderGITable()` function that applies sort before rendering; a new `giSort()` function toggles sort state and calls it. The `<thead>` row gains `data-gi-col` attributes so `giSort` can update indicators directly without re-rendering the full panel.

**Tech Stack:** Vanilla JS, no build step. All edits are in `index.html`. Verify by opening the file in Chrome.

---

## File Map

| File | Lines touched | What changes |
|---|---|---|
| `index.html` | 886 | Add 3 state vars after `orgNavList` |
| `index.html` | 1920–1934 | `showGI()` — reset state before calling `renderGI` |
| `index.html` | 1939 | `renderGI` — replace single-string filter with `giSelectedFYs` array filter |
| `index.html` | 2111–2118 | `renderGI` template — replace FY button `onclick` + active-state logic |
| `index.html` | 2188–2197 | `renderGI` template — replace `<thead>` with sortable headers |
| `index.html` | 2198–2227 | `renderGI` template — replace `<tbody>` with static `id="gi-tbody"` tag |
| `index.html` | 2235 (after renderGI) | Add `renderGITable()` function |
| `index.html` | after renderGITable | Add `giSort()` function |

---

## Task 1: Add module-level state variables

**Files:**
- Modify: `index.html:886`

- [ ] **Step 1: Add the three state variables after `orgNavList` on line 886**

Current line 886:
```js
let orgNavList = [];
```

Replace with:
```js
let orgNavList = [];
let giSelectedFYs = [];
let giSortCol     = null;
let giSortDir     = "asc";
```

- [ ] **Step 2: Open `index.html` in Chrome and verify no console errors**

Open the file, open DevTools Console (F12), load a `grants_data.json`. Expect: no errors. The dashboard should behave identically to before.

---

## Task 2: Reset state in `showGI()`

**Files:**
- Modify: `index.html:1933`

- [ ] **Step 1: Add state resets inside `showGI()` before the `renderGI` call**

Current line 1933:
```js
  renderGI(orgName, orgGrants, fyList, "");
```

Replace with:
```js
  giSelectedFYs = [];
  giSortCol     = null;
  giSortDir     = "asc";
  renderGI(orgName, orgGrants, fyList, "");
```

- [ ] **Step 2: Verify in browser**

Click "↗ Organization View" on any grant. Org view opens. No console errors. Behavior unchanged from before.

---

## Task 3: Replace FY filter logic in `renderGI`

**Files:**
- Modify: `index.html:1939`

- [ ] **Step 1: Replace the single-string filter with the `giSelectedFYs` array filter**

Current line 1939:
```js
  const filtered = fyFilter ? orgGrants.filter(r => str(r["Sub Code"]||r["Year"]) === fyFilter) : orgGrants;
```

Replace with:
```js
  const filtered = giSelectedFYs.length
    ? orgGrants.filter(r => giSelectedFYs.includes(str(r["Sub Code"]||r["Year"])))
    : orgGrants;
```

- [ ] **Step 2: Verify in browser**

Open any org view. Table still shows all grants (all years). No console errors. Score unchanged.

---

## Task 4: Replace FY filter buttons in the `renderGI` template

**Files:**
- Modify: `index.html:2111–2118`

This replaces the FY button block inside the `renderGI` template string (the `<!-- FY FILTER -->` section).

- [ ] **Step 1: Replace the "All Years" button (line 2111–2114)**

Current:
```js
      <button onclick="renderGI('${esc(orgName)}',orgGrantsCache,'${fyList.join(",")}','')" 
        style="padding:5px 12px;border-radius:20px;border:1px solid ${!fyFilter?'var(--accent)':'var(--border)'};background:${!fyFilter?'var(--accent)':'white'};color:${!fyFilter?'white':'var(--text-secondary)'};font-size:12px;cursor:pointer;font-family:'Lexend',sans-serif;font-weight:600">
        All Years
      </button>
```

Replace with:
```js
      <button onclick="giSelectedFYs=[];renderGI('${esc(orgName)}',orgGrantsCache,'${fyList.join(",")}','')"
        style="padding:5px 12px;border-radius:20px;border:1px solid ${giSelectedFYs.length===0?'var(--accent)':'var(--border)'};background:${giSelectedFYs.length===0?'var(--accent)':'white'};color:${giSelectedFYs.length===0?'white':'var(--text-secondary)'};font-size:12px;cursor:pointer;font-family:'Lexend',sans-serif;font-weight:600">
        All Years
      </button>
```

- [ ] **Step 2: Replace the individual FY buttons (lines 2115–2118)**

Current:
```js
      ${fyList.map(fy=>`<button onclick="renderGI('${esc(orgName)}',orgGrantsCache,'${fyList.join(",")}','${fy}')"
        style="padding:5px 12px;border-radius:20px;border:1px solid ${fyFilter===fy?'var(--accent)':'var(--border)'};background:${fyFilter===fy?'var(--accent)':'white'};color:${fyFilter===fy?'white':'var(--text-secondary)'};font-size:12px;cursor:pointer;font-family:'Lexend',sans-serif;font-weight:600">
        FY${fy}
      </button>`).join("")}
```

Replace with:
```js
      ${fyList.map(fy=>`<button onclick="(function(y){const i=giSelectedFYs.indexOf(y);if(i===-1)giSelectedFYs.push(y);else giSelectedFYs.splice(i,1);renderGI('${esc(orgName)}',orgGrantsCache,'${fyList.join(",")}','');}('${fy}'))"
        style="padding:5px 12px;border-radius:20px;border:1px solid ${giSelectedFYs.includes(fy)?'var(--accent)':'var(--border)'};background:${giSelectedFYs.includes(fy)?'var(--accent)':'white'};color:${giSelectedFYs.includes(fy)?'white':'var(--text-secondary)'};font-size:12px;cursor:pointer;font-family:'Lexend',sans-serif;font-weight:600">
        FY${fy}
      </button>`).join("")}
```

- [ ] **Step 3: Verify multi-select in browser**

Open an org that has grants in 2+ fiscal years (e.g. an org with both FY25 and FY26 grants).

Expected behavior:
- "All Years" highlighted on first open
- Click FY26 → FY26 highlighted, All Years un-highlights, table/score shows FY26 only
- Click FY25 → both FY25 and FY26 highlighted, table/score shows both years
- Click FY26 again → only FY25 highlighted, table/score shows FY25 only
- Click "All Years" → all FY buttons un-highlight, table/score shows all years
- Open a different org → resets to All Years (Task 2 reset fires)

No console errors.

---

## Task 5: Add `renderGITable()` function

This function owns the Grant History table body render. It re-derives `filtered` from `orgGrantsCache` + `giSelectedFYs`, applies sort, and writes to `#gi-tbody`.

**Files:**
- Modify: `index.html` — add after line 2235 (end of `renderGI`)

- [ ] **Step 1: Add `renderGITable()` after the closing `}` of `renderGI` (after line 2235)**

```js
function renderGITable() {
  const rows = (() => {
    const base = giSelectedFYs.length
      ? window.orgGrantsCache.filter(r => giSelectedFYs.includes(str(r["Sub Code"]||r["Year"])))
      : window.orgGrantsCache;
    if (!giSortCol) return [...base];
    return [...base].sort((a, b) => {
      if (giSortCol === "type") {
        const av = str(a["Grant Type"]).toLowerCase();
        const bv = str(b["Grant Type"]).toLowerCase();
        return giSortDir === "asc" ? av.localeCompare(bv) : bv.localeCompare(av);
      }
      if (giSortCol === "fy") {
        const av = parseInt(a["Sub Code"]||a["Year"]||0);
        const bv = parseInt(b["Sub Code"]||b["Year"]||0);
        return giSortDir === "asc" ? av - bv : bv - av;
      }
      if (giSortCol === "status") {
        const av = str(a["Status"]).toLowerCase();
        const bv = str(b["Status"]).toLowerCase();
        return giSortDir === "asc" ? av.localeCompare(bv) : bv.localeCompare(av);
      }
      if (giSortCol === "budget") {
        const getPct = r => {
          const apv = num(r["Apv Total"]||r["Apv Program Total"]);
          const { actTotal } = getFinancialSummary(str(r["Grant Number"]));
          return (apv > 0 && actTotal > 0) ? actTotal / apv : null;
        };
        const av = getPct(a), bv = getPct(b);
        if (av === null && bv === null) return 0;
        if (av === null) return 1;
        if (bv === null) return -1;
        return giSortDir === "asc" ? av - bv : bv - av;
      }
      return 0;
    });
  })();

  document.getElementById("gi-tbody").innerHTML = rows.map(r => {
    const gn  = str(r["Grant Number"]);
    const fy  = parseInt(r["Sub Code"]||r["Year"]||0);
    const sig = calcSignal(gn, fy);
    const apv = num(r["Apv Total"]||r["Apv Program Total"]);
    const { actTotal: actual } = getFinancialSummary(gn);
    const pctUsed = (apv > 0 && actual > 0) ? actual/apv : null;
    return `<tr onclick="showDetailFromOrg('${esc(gn)}')" style="cursor:pointer">
      <td class="left"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--accent);font-weight:500">${esc(gn)}</span></td>
      <td><span class="badge badge-${esc(str(r["Grant Type"]))}">${esc(str(r["Grant Type"])||"—")}</span></td>
      <td><span class="badge badge-year">FY${fy}</span></td>
      <td class="left" style="color:var(--text-secondary);max-width:200px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(str(r["Project Title"])||"—")}</td>
      <td style="font-size:11px;color:var(--text-muted)">${esc(str(r["Status"])||"—")}</td>
      <td style="text-align:center">
        ${pctUsed !== null
          ? `<span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:${pctUsed>=0.92?'var(--green)':pctUsed>=0.5?'var(--amber)':'var(--red)'};font-weight:600">${(pctUsed*100).toFixed(1)}%</span>`
          : `<span class="status-null">—</span>`}
      </td>
      <td><span class="signal signal-${sig.level}" style="font-size:10px">${sig.label}</span></td>
      <td>
        <div class="link-row">
          ${lbtn(r["Landing Page URL"],    "🏠","Landing Page")}
          ${lbtn(r["Grant Agreement URL"], "📄","Agreement")}
        </div>
      </td>
    </tr>`;
  }).join("");
}
```

- [ ] **Step 2: Verify function exists with no syntax errors**

Open the file in Chrome. Open DevTools Console. Type `renderGITable` and press Enter.
Expected: `ƒ renderGITable()` (function reference, not an error).

---

## Task 6: Add `giSort()` function

**Files:**
- Modify: `index.html` — add immediately after `renderGITable()`

- [ ] **Step 1: Add `giSort()` after `renderGITable()`**

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

- [ ] **Step 2: Verify function exists with no syntax errors**

In DevTools Console, type `giSort` and press Enter.
Expected: `ƒ giSort(col)` (not an error).

---

## Task 7: Wire up the Grant History table — `<thead>` and `<tbody>`

> **Prerequisite:** Tasks 5 and 6 must be complete — `renderGITable()` and `giSort()` must exist before this task wires them into the DOM.

This task replaces the existing `<thead>` and `<tbody>` block inside the `renderGI` template string (lines 2188–2227).

**Files:**
- Modify: `index.html:2188–2227`

- [ ] **Step 1: Replace the `<thead>` block (lines 2188–2197)**

Current:
```html
        <thead><tr>
          <th class="left">Grant Number</th>
          <th>Type</th>
          <th>FY</th>
          <th class="left">Project Title</th>
          <th>Status</th>
          <th>Budget Used</th>
          <th>Signal</th>
          <th>Links</th>
        </tr></thead>
```

Replace with:
```html
        <thead><tr>
          <th class="left no-sort">Grant Number</th>
          <th data-gi-col="type"   onclick="giSort('type')"   class="${giSortCol==='type'   ?(giSortDir==='asc'?'sort-asc':'sort-desc'):''}">Type</th>
          <th data-gi-col="fy"     onclick="giSort('fy')"     class="${giSortCol==='fy'     ?(giSortDir==='asc'?'sort-asc':'sort-desc'):''}">FY</th>
          <th class="left no-sort">Project Title</th>
          <th data-gi-col="status" onclick="giSort('status')" class="${giSortCol==='status' ?(giSortDir==='asc'?'sort-asc':'sort-desc'):''}">Status</th>
          <th data-gi-col="budget" onclick="giSort('budget')" class="${giSortCol==='budget' ?(giSortDir==='asc'?'sort-asc':'sort-desc'):''}">Budget Used</th>
          <th class="no-sort">Signal</th>
          <th class="no-sort">Links</th>
        </tr></thead>
```

- [ ] **Step 2: Replace the `<tbody>` block (lines 2198–2227)**

Current (lines 2198–2227):
```js
        <tbody>
          ${filtered.map(r => {
            const gn  = str(r["Grant Number"]);
            const fy  = parseInt(r["Sub Code"]||r["Year"]||0);
            const sig = calcSignal(gn, fy);
            const apv = num(r["Apv Total"]||r["Apv Program Total"]);
            const { actTotal: actual } = getFinancialSummary(gn);
            let pctUsed = null;
            if (apv > 0 && actual > 0) { pctUsed = actual/apv; }
            return `<tr onclick="showDetailFromOrg('${esc(gn)}')" style="cursor:pointer">
              <td class="left"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:var(--accent);font-weight:500">${esc(gn)}</span></td>
              <td><span class="badge badge-${esc(str(r["Grant Type"]))}">${esc(str(r["Grant Type"])||"—")}</span></td>
              <td><span class="badge badge-year">FY${fy}</span></td>
              <td class="left" style="color:var(--text-secondary);max-width:200px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(str(r["Project Title"])||"—")}</td>
              <td style="font-size:11px;color:var(--text-muted)">${esc(str(r["Status"])||"—")}</td>
              <td style="text-align:center">
                ${pctUsed !== null
                  ? `<span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:${pctUsed>=0.92?'var(--green)':pctUsed>=0.5?'var(--amber)':'var(--red)'};font-weight:600">${(pctUsed*100).toFixed(1)}%</span>`
                  : `<span class="status-null">—</span>`}
              </td>
              <td><span class="signal signal-${sig.level}" style="font-size:10px">${sig.label}</span></td>
              <td>
                <div class="link-row">
                  ${lbtn(r["Landing Page URL"],    "🏠","Landing Page")}
                  ${lbtn(r["Grant Agreement URL"], "📄","Agreement")}
                </div>
              </td>
            </tr>`;
          }).join("")}
        </tbody>
```

Replace with (static tag — `renderGITable()` fills it after render):
```html
        <tbody id="gi-tbody"></tbody>
```

- [ ] **Step 3: Call `renderGITable()` at the end of `renderGI`, after setting `window.orgGrantsCache`**

Current lines 2233–2235 (end of renderGI):
```js
  // Cache org grants for FY toggle buttons
  window.orgGrantsCache = orgGrants;
}
```

Replace with:
```js
  // Cache org grants for FY toggle buttons and renderGITable
  window.orgGrantsCache = orgGrants;
  renderGITable();
}
```

- [ ] **Step 4: Verify full org view in browser**

Open any org view. Expected:
- Grant History table renders with all grants
- Type, FY, Status, Budget Used headers show pointer cursor and accent hover color
- Grant Number, Project Title, Signal, Links headers show muted cursor (no-sort)
- No console errors

---

## Task 8: Verify all acceptance criteria and commit

- [ ] **Step 1: Run through the full verify checklist**

Open an org with grants in 2+ FYs:

| Check | Expected |
|---|---|
| Open org | "All Years" highlighted, no FY highlighted |
| Click FY26 | FY26 highlighted, All Years un-highlights, score recalculates for FY26 only |
| Click FY25 | Both FY25+FY26 highlighted, score recalculates for both years |
| Click FY26 again | Only FY25 highlighted, score recalculates for FY25 only |
| Click All Years | All FY buttons un-highlight, score reverts to all-years |
| Open different org | Resets to All Years |
| Click Type header | Table sorts A→Z, ↑ indicator on Type |
| Click Type again | Table sorts Z→A, ↓ indicator on Type |
| Click FY header | Table sorts numerically |
| Click Status header | Table sorts alphabetically |
| Click Budget Used | Sorts by %, nulls (—) at bottom in both directions |
| Sort active, change FY | Table re-renders with new FY filter AND current sort applied |
| Click sort header | Score panels and metrics row do NOT flash/re-render |
| Console | No errors |

- [ ] **Step 2: Commit**

```
git add index.html
git commit -m "feat: multi-select FY filter and column sort in org view"
```
