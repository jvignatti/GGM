# AI Workflow Rules — GGM Chrome Extension

## Working Style

This project is worked on-demand — sessions happen when the user has time, not on
a fixed schedule. Between sessions, the codebase is live and in use.

**Claude's role between units:** Suggest from the open list (progress-tracker.md
open bugs + open questions) but do not assume priority. The user decides what to
work on. Surface relevant findings, flag regressions, propose the next logical
step — but wait for direction before starting.

**Do not push a roadmap.** The 18-unit build plan is a reference, not a sprint.

## Overall Approach

Build one unit at a time. Every unit has a spec file in `context/specs/` that defines
exactly what to build and how to verify it. Do not begin implementation until the spec
for the current unit exists and has been reviewed.

## Scoping Rules

- Implement exactly what the current spec says. Do not add features, refactors, or
  "while I'm here" improvements beyond the spec.
- Do not modify files outside the unit's defined scope unless the spec explicitly names
  them. If a change requires touching another unit's boundary, stop and note the
  dependency rather than expanding scope.
- One unit = one focused session. If a session runs long, finish the current unit
  cleanly rather than starting the next one mid-stream.

## Before Writing Any Code

1. Read `CLAUDE.md` — the project snapshot.
2. Read `context/architecture.md` — especially the Invariants section. Every invariant
   must be respected in the implementation. If the spec conflicts with an invariant,
   flag it before proceeding.
3. Read `context/progress-tracker.md` — know what is done and what is next.
4. Read the spec file for the current unit.

## Implementing a Unit

- Update `context/progress-tracker.md` to mark the unit as `in_progress` before
  writing the first line of implementation code.
- Write the simplest code that satisfies the spec. No speculative abstractions.
- For portal scraping code: always add a comment citing the specific element selector
  or parent ID being used, and why (e.g., `// auxNav parentId=5000008 = invoices`).
  These are portal implementation details that will not be obvious from the code.
- Financial calculations: always use `getFinancialSummary(grantNumber)` from
  `shared/financial.js` — never inline a `.find()` call on the financial store.

## Verification Before Completing a Unit

Do not mark a unit complete until all of these hold:
- [ ] The spec's "Verify when done" checklist is fully green.
- [ ] No `console.error` or unhandled rejections in the browser DevTools console.
- [ ] The extension loads without manifest errors in `chrome://extensions`.
- [ ] Dark mode renders correctly (toggle and verify).
- [ ] `context/progress-tracker.md` updated with the unit marked complete.

## Protected Files

Do not modify these files without explicit instruction:
- `context/architecture.md` — architectural changes require discussion first
- `old_gears_data.json` — read-only legacy data, never written by any script
- `manifest.json` — permission changes require explicit approval

## Handling Ambiguity

- If the spec is ambiguous about a portal selector or data field name, check
  `context/architecture.md` — the portal structure is fully documented there.
- If the financial data behavior is unclear, apply INV-1: sum all matching rows.
- If the QTR format is unclear, apply the normalization rule in architecture.md.
- Do not make up portal structure — the architecture doc has the ground truth from
  live scraping sessions. If something isn't documented there, flag it.

## When to Stop and Ask

Stop and flag before proceeding if:
- The implementation would violate any invariant in `context/architecture.md`.
- A portal page structure doesn't match what's documented (portal may have changed).
- The spec requires a permission not currently in `manifest.json`.
- A unit would require changes to more than 3 files outside its defined boundary.

## Updating Docs

- After any session that adds a new portal selector, discovered data shape, or
  confirmed vendor behavior: update `context/architecture.md` with the finding.
- After any bug fix that should become a rule: add it to the Invariants section.
- Keep `context/progress-tracker.md` current. It is the single source of truth for
  what is done and what is next.
