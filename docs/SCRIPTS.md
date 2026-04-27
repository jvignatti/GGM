# Script Inventory
## File Names, Descriptions, and Pipeline Roles

---

> **Important:** This document lists script names and describes what each script does.
> No source code, internal URLs, credentials, system references, or operational details
> are included. Scripts are not published in this repository.

---

## Core Pipeline Scripts

### `master_scrape.py`

**Role:** Full pipeline runner — executes all steps in sequence.

**What it does:**
- Orchestrates the complete data pipeline from start to finish
- Calls the export automation, scraper, and JSON merge in order
- Logs progress and errors to the console
- Produces the final `grants_data.json` upon completion

**When to run:** When you need a full refresh of all data. Typical runtime: 10–25 minutes depending on portfolio size and portal response times.

**Dependencies:** Requires the grants portal to be accessible and valid credentials stored in the system keyring.

---

### `test_merge_json.py`

**Role:** Isolated JSON merge tester.

**What it does:**
- Reads the six CSV/Excel source files from the local output folder
- Merges them into `grants_data.json` without running the scraper or export steps
- Prints a summary of record counts per data source
- Performs a spot-check on the first record of each array

**When to run:** When the CSV files are already current and you only need to regenerate the JSON — for example, after manually correcting a CSV or after the scraper has already run. Much faster than `master_scrape.py`.

**Output:** `grants_data.json` in the same directory.

---

## Supporting Components

### Export Automation Module

**Role:** Automates the download of structured reports from the grants portal.

**What it does:**
- Navigates to each report section of the portal
- Triggers the export for each of the seven reports
- Saves the output files to the designated local folder
- Handles portal navigation timing and export confirmation

**Reports produced:**
- Grant Summary
- SHSO Financial Report
- Indirect Cost
- Supervisors Report
- (Grants Invoices and Grants QTR are populated by the scraper)

---

### Scraper Module

**Role:** Extracts invoice and quarterly report records for each grant.

**What it does:**
- Reads the grant list from a designated input file
- Navigates to each grant's detail page on the portal
- Extracts all invoice records and all QTR records
- Appends new records to the invoice and QTR CSV files
- Skips grants already fully scraped (deduplication)
- Logs timeouts and errors without stopping the full run

**Concurrency:** Processes multiple grants simultaneously to reduce total runtime. Concurrency level is configurable.

**Error handling:** Grants that time out are logged with their grant number and error message. They can be retried by re-running the scraper — already-scraped grants are skipped automatically.

---

### Merge Module

**Role:** Combines all data sources into a single JSON file.

**What it does:**
- Reads all six source files (CSVs and optionally Excel)
- Normalizes null/empty values
- Writes a structured JSON with six named arrays
- Records the generation timestamp
- Validates that key source files are present before writing

**Output structure:** See [`METHODOLOGY.md`](METHODOLOGY.md) for the full JSON schema.

---

## Configuration

All configurable parameters (timeouts, concurrency, folder paths, report identifiers) are defined in a single configuration block at the top of the master script. No configuration values are published in this repository.

---

## Dependencies

The pipeline depends on the following Python packages:

| Package | Purpose |
|---|---|
| `playwright` | Browser automation for portal navigation and export |
| `beautifulsoup4` | HTML parsing for scraper data extraction |
| `pandas` | CSV/Excel reading and data normalization |
| `keyring` | Secure credential storage (no passwords in code) |
| `pathlib` | Cross-platform file path handling |
| `asyncio` | Concurrent scraping execution |

All packages are installable via `pip`. No internal or proprietary packages are required.

---

## Runtime Summary

| Script | Typical Runtime | When to Use |
|---|---|---|
| `master_scrape.py` | 10–25 minutes | Full data refresh |
| `test_merge_json.py` | < 5 seconds | JSON rebuild only |
