# UI Context — GGM Dashboard

## Theme

Government/public service financial dashboard. Data-dense, professional, readable.
Two modes: light (default) and dark. Dark mode toggles via `[data-theme="dark"]` on
`<html>` and persists in `chrome.storage.local` as `shso_theme`.

## Color Tokens

### Light Mode (`:root`)

| Token | Value | Usage |
|---|---|---|
| `--primary` | `#0F172A` | Header background, primary text |
| `--primary-fg` | `#FFFFFF` | Text on primary |
| `--secondary` | `#334155` | Secondary elements |
| `--accent` | `#0369A1` | Links, badges, interactive elements |
| `--accent-fg` | `#FFFFFF` | Text on accent |
| `--accent-light` | `#E0F2FE` | Accent hover backgrounds |
| `--accent-hover` | `#0284C7` | Accent hover state |
| `--bg` | `#F8FAFC` | Page background |
| `--surface` | `#FFFFFF` | Card/panel background |
| `--surface2` | `#F1F5F9` | Table header, secondary surface |
| `--surface3` | `#E8ECF1` | Progress bar background |
| `--border` | `#E2E8F0` | Default border |
| `--border-strong` | `#CBD5E1` | Emphasized border |
| `--text` | `#020617` | Body text |
| `--text-secondary` | `#475569` | Secondary body text |
| `--text-muted` | `#64748B` | Labels, captions |
| `--text-faint` | `#94A3B8` | Placeholder, disabled |
| `--green` | `#16A34A` | Success, approved |
| `--green-bg` | `#DCFCE7` | Success background |
| `--green-border` | `#86EFAC` | Success border |
| `--amber` | `#D97706` | Warning |
| `--amber-bg` | `#FEF3C7` | Warning background |
| `--amber-border` | `#FCD34D` | Warning border |
| `--orange` | `#EA580C` | High priority |
| `--orange-bg` | `#FFEDD5` | High priority background |
| `--red` | `#DC2626` | Critical, error, over-budget |
| `--red-bg` | `#FEE2E2` | Critical background |
| `--red-border` | `#FCA5A5` | Critical border |

### Dark Mode (`[data-theme="dark"]`)

| Token | Value |
|---|---|
| `--primary` | `#1E293B` |
| `--accent` | `#38BDF8` |
| `--accent-fg` | `#0F172A` |
| `--bg` | `#0F172A` |
| `--surface` | `#1E293B` |
| `--surface2` | `#263348` |
| `--text` | `#F1F5F9` |
| `--green` | `#4ADE80` |
| `--green-bg` | `#14532D` |
| `--amber` | `#FCD34D` |
| `--amber-bg` | `#78350F` |
| `--red` | `#F87171` |
| `--red-bg` | `#7F1D1D` |

## Typography

| Font | Role | Variable |
|---|---|---|
| Lexend | Headings, labels, badges | `font-family: 'Lexend', sans-serif` |
| Source Sans 3 | Body text, inputs | `font-family: 'Source Sans 3', sans-serif` |
| JetBrains Mono | Grant numbers, amounts, timestamps, codes | `font-family: 'JetBrains Mono', monospace` |

All three fonts must be bundled as woff2 in `assets/fonts/` — no Google Fonts CDN.

## Spacing & Radius

| Token | Value | Usage |
|---|---|---|
| `--radius` | `6px` | Default border radius |
| `--radius-lg` | `10px` | Cards, panels |
| `--header-height` | `56px` | Sticky header |
| `--table-row` | `36px` | Table row height |

## Shadows

| Token | Value |
|---|---|
| `--shadow-sm` | `0 1px 2px rgba(15,23,42,0.06)` |
| `--shadow` | `0 1px 3px rgba(15,23,42,0.1), 0 1px 2px rgba(15,23,42,0.06)` |
| `--shadow-md` | `0 4px 6px rgba(15,23,42,0.08), 0 2px 4px rgba(15,23,42,0.04)` |
| `--shadow-lg` | `0 10px 15px rgba(15,23,42,0.08), 0 4px 6px rgba(15,23,42,0.04)` |

## Signal Badges

Signal level 0–5. CSS class `signal signal-{level}`:

| Level | Meaning | Background | Color |
|---|---|---|---|
| 0 | Clean | `#DCFCE7` | `#15803D` |
| 1 | Watch | `#D1FAE5` | `#065F46` |
| 2 | Caution | `#FEF3C7` | `#92400E` |
| 3 | Warning | `#FFEDD5` | `#9A3412` |
| 4 | High | `#FEE2E2` | `#991B1B` |
| 5 | Critical | `#1C0505` | `#F87171` |
| na | No data | `var(--surface2)` | `var(--text-faint)` |

## Grant Type Badges

CSS class `badge badge-{type}`:

| Type | Background | Color | Border |
|---|---|---|---|
| DRE | `#DBEAFE` | `#1D4ED8` | `#BFDBFE` |
| EE | `#DCFCE7` | `#15803D` | `#86EFAC` |
| EDUC | `#FEF3C7` | `#B45309` | `#FCD34D` |
| TRCC | `#F3E8FF` | `#7C3AED` | `#DDD6FE` |

## Status Badges

| Badge | Background | Color | Border |
|---|---|---|---|
| legacy | `#F1F5F9` | `#64748B` | `#CBD5E1` |
| amendment | `#FEF9C3` | `#854D0E` | `#FDE047` |
| supplemental | `#EDE9FE` | `#6D28D9` | `#C4B5FD` |

## Layout Patterns

- **Header**: sticky, `--header-height` tall, `--primary` background, logo + stats + controls
- **Breadcrumb**: `36px`, `--surface` background, shows on detail views, has Prev/Next nav
- **Toolbar**: `--surface`, filters row, search input, per-page selector
- **Table**: `--surface` card with `--radius-lg`, sticky thead, hover row highlight
- **Detail view**: `padding: 20px 28px`, flex-column, gap 18px between sections
- **Panel**: `--surface` card with `--radius-lg`, panel-head section with title + meta
- **Financial grid**: 3-column CSS grid, `fin-cell` with label/amount/badge/progress-bar
- **Side panel**: narrow (360px), `--surface` background, always-visible signal + score

## Component Conventions

- All interactive elements (`<button>`, `<select>`, `<input>`) use `--border` and
  transition to `--accent` on focus/hover.
- Table rows are clickable (`cursor: pointer`) — click opens detail view.
- Tooltips via `data-tip` attribute + `::after` pseudo-element on `.lbtn:hover`.
- PDF export forces `background: #ffffff; color: #000000` on the detail container,
  hides the export button during capture, then restores.

## Side Panel Specific

- Width: 360px, fixed
- Shows: grant number (mono), signal badge, org name, Grantee Score (large number)
- Last-scraped timestamp in muted text at bottom
- "Open in Dashboard" button → opens/focuses the full extension dashboard tab
