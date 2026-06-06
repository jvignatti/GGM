## Application Building Context

Read the following files in order before implementing
or making any architectural decision:

1. `context/project-overview.md` — product definition, goals, features, and scope
2. `context/architecture.md` — system structure, boundaries, storage model, and invariants
3. `context/ui-context.md` — theme, colors, typography, and component conventions
4. `context/code-standards.md` — implementation rules and conventions
5. `context/ai-workflow-rules.md` — development workflow, scoping rules, and delivery approach
6. `context/progress-tracker.md` — current phase, completed work, open questions, and next steps

Update `context/progress-tracker.md` after each meaningful implementation change.

If implementation changes the architecture, scope, or standards documented in the context
files, update the relevant file before continuing.

---

## Project Snapshot (read this first)

**What it is:** A local-only Manifest V3 Chrome extension that replaces a Python scraper +
standalone HTML dashboard. It rides the user's existing logged-in session on
grants.vermont.gov (Vermont SHSO grants portal), scrapes all grant data in-browser, stores
it locally in IndexedDB, and renders the GGM dashboard inside the extension.

**What it is not:** A server. A cloud app. A tool that stores credentials. A tool that sends
any data anywhere.

**Current state:** The old pipeline (master_scrape.py + index.html + grants_data.json) is
still running and working. The extension is a full replacement, not an addition.

**The financial bug is documented** in `context/architecture.md` invariants — financial data
requires summing all matching rows, not returning the first match.
