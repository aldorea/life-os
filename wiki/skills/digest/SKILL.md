---
name: digest
description: Use when reviewing wiki activity over a time period. Triggered by `/wiki:digest [day|week|month]`. Reads log.md and produces a formatted summary of ingests, queries, lint passes, and pages created/updated. Defaults to 'week' if no argument given.
---

# wiki:digest

Activity summary for the personal wiki from `log.md`.

## Vault Paths
- Log: `08 Resources/wiki/log.md`

## Process

### Step 1: Determine period

- `day` → today (current date)
- `week` → last 7 days
- `month` → last 30 days
- No argument → default to `week`

### Step 2: Read log

```
obsidian read path="08 Resources/wiki/log.md"
```

### Step 3: Filter entries for the period

Parse entries with format `## YYYY-MM-DDTHH:MM | <operation> | <detail>`. Select only entries within the date range.

### Step 4: Aggregate stats

From filtered entries, count:
- **Ingests**: total entries with `| ingest |`
  - Break down by source type: URL (starts with `http`), YouTube (`youtube.com`/`youtu.be`), PDF (ends with `.pdf`), text (everything else)
- **Pages created**: sum of `Pages created:` values across all ingest entries (excluding `none`)
- **Pages updated**: sum of `Pages updated:` values across all ingest entries (excluding `none`)
- **Queries**: total entries with `| query |`
- **Lint passes**: total entries with `| lint |`

### Step 5: Format report

```
📅 [Period label]
─────────────────────────────
Ingests: N  (X URLs, Y PDFs, Z YouTube, W text)
Pages new: [page1, page2, ...]
Pages updated: [page1, page2, ...]
Queries: N
Lint: N pass(es)
─────────────────────────────
```

If no activity in the period:
> "No wiki activity in the last [period]."

### Step 6: No log entry needed

`digest` is read-only — do not append to log.md.
