# Data Safety & Privacy
## Data Governance, Security Policy, and Handling Guidelines

---

## Summary

| Item | Safe to Share Publicly | Notes |
|---|---|---|
| `dashboard.html` | ✅ Yes | Contains no data |
| `README.md` | ✅ Yes | No sensitive content |
| `docs/*.md` | ✅ Yes | Methodology only, no operational data |
| `grants_data.json` | ❌ Never | Contains sensitive operational data |
| Scripts (source code) | ❌ No | Internal automation logic |
| Portal credentials | ❌ Never | Stored in system keyring, never in code |

---

## What Data the JSON Contains

The `grants_data.json` file contains operational and financial data belonging to the State of Vermont. Specifically:

- Grant numbers, types, and award amounts
- Agency names and grant assignments
- Expenditure and budget figures
- Invoice and quarterly report records and statuses
- Supervisor names and grant assignments
- Timestamps of data collection

This data is operational in nature and is used solely for internal program management purposes.

---

## What Data the Dashboard Contains

The `dashboard.html` file is a completely empty interface. It contains:
- No grant data
- No names
- No financial figures
- No URLs or system references
- No credentials

It is a rendering engine only. Data enters it temporarily when a user loads the JSON file in their browser. When the browser tab is closed, the data is gone.

---

## Data Handling Rules

### The JSON File

1. **Do not upload `grants_data.json` to GitHub** or any public repository
2. **Do not attach `grants_data.json` to email** unless the recipient is an authorized team member and the email system is internal
3. **Do not store `grants_data.json` in shared cloud folders** (Google Drive, OneDrive public links, Dropbox) accessible outside the organization
4. **Do not share `grants_data.json` with vendors, contractors, or external parties** without explicit authorization
5. **Treat the JSON file as you would any sensitive HR or financial report** — same handling standards apply

### Authorized Internal Distribution

The JSON file may be shared:
- Directly between authorized SHSO staff members via internal systems
- Via secure internal file shares approved by IT
- As a local file on organization-managed devices

### Retention

JSON files are point-in-time snapshots. Older versions do not need to be retained indefinitely. Follow your organization's standard data retention policy for operational reports.

---

## How the Dashboard Protects Privacy

**No server transmission:** The dashboard operates entirely in the browser. When you drop the JSON file onto the upload area, it is read by JavaScript running on your local machine. The data is never sent anywhere.

**No storage:** The browser does not save the data between sessions. Every time you close and reopen the dashboard, you start with a blank interface.

**No tracking:** The dashboard contains no analytics, no cookies, no external script calls, and no telemetry of any kind.

**No authentication:** The safety model relies on keeping the JSON file internal. The dashboard itself has no login because it contains nothing to protect. The data file is what must be protected.

---

## GitHub Repository Security

This repository is designed to be public. A security review of its contents confirms:

| Content | Sensitive? | Status |
|---|---|---|
| `dashboard.html` | No | Safe — contains no data |
| `README.md` | No | Safe — no operational details |
| `METHODOLOGY.md` | No | Safe — describes process, no data |
| `MATH.md` | No | Safe — equations only |
| `DASHBOARD.md` | No | Safe — UI documentation |
| `SCRIPTS.md` | No | Safe — names and descriptions only |
| `HOW_TO_USE.md` | No | Safe — user guide |
| `IMPACT.md` | No | Safe — generic estimates |
| `DATA_SAFETY.md` | No | Safe — this document |
| `DISCLAIMER.md` | No | Safe — legal language |
| `.gitignore` | No | Safe — file exclusion rules |

**Nothing in this repository exposes:** grant numbers, agency names, financial figures, employee names, portal URLs, system credentials, or internal system architecture.

---

## Incident Response

If `grants_data.json` is accidentally published to a public location:

1. Remove the file immediately from the public location
2. Notify your supervisor and IT
3. If published to GitHub: delete the file, force-push to remove from history, and contact GitHub support to purge cached versions
4. Document the incident per your organization's data breach policy

---

## Questions

For questions about data handling policy, contact your IT department or designated data governance officer.
