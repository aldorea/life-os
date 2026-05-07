---
name: digest
description: Use when reviewing wiki activity over a time period. Triggered by `/wiki:digest [day|week|month]`. Reads log.md and produces a formatted summary of ingests, queries, lint passes, and pages created/updated. Defaults to 'week' if no argument given.
---

# wiki:digest

Activity summary for the personal wiki from `log.md`.

## Vault Paths
- Log: `08 Resources/wiki/log.md`
- Drafts: `08 Resources/wiki/.drafts/`

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
- **Drafts created**: sum of `Drafts created:` values across all ingest entries (excluding `none`)
- **Pages updated**: sum of `Pages updated:` values across all ingest entries (excluding `none`)
- **Queries**: total entries with `| query |`
- **Synthesize**: total entries with `| synthesize |`
- **Approves**: total entries with `| approve |` (broken down by `Action: published` vs `discarded`)
- **Lint passes**: total entries with `| lint |`
- **Daily closes**: total entries with `| daily |` — these are bridge entries appended by the `close` ritual. Each carries `Foco`, `Completadas`, `Energia`, `Note`. The digest uses them to build a per-day timeline of the period.
- **Inbox captures**: total entries with `| inbox |` — items captured to `02 Inbox.md` via dump or telegram (one log line per batch, not per item). Carries `Items: N`.
- **Process-inbox runs**: total entries with `| process-inbox |` — batches of clarification (raw → tasks). Carries `Items processed`, `Tasks created`.
- **Training captures**: total entries with `| training |` — telegram-routed gym sessions. Carries `Date`, `Lines`.

### Step 4b: Pending drafts (snapshot, not from log)

List current drafts in `.drafts/`:
```bash
ls "/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/08 Resources/wiki/.drafts/"
```

For each draft, read its frontmatter `updated`. Compute age in days.

### Step 5: Format report

```
📅 [Period label]
─────────────────────────────
Daily closes: N días cerrados
  - YYYY-MM-DD — Foco: <foco> — N tasks, <color> — "<note>"
  - ...
─────────────────────────────
Tasks (GTD)
  Inbox captures:    N (M items)
  Process-inbox:     N batches, X tasks created
─────────────────────────────
Knowledge (wiki)
  Ingests:           N  (X URLs, Y PDFs, Z YouTube, W text)
  Pages published:   [page1, ...]
  Pages updated:     [page1, ...]
  Synthesize:        N pages
  Queries:           N
  Approves:          N (X published, Y discarded)
  Lint:              N pass(es)
─────────────────────────────
Health
  Training captures: N sesiones
─────────────────────────────
Drafts pendientes: N
  - <name> (Nd) — <source>
  - ...
─────────────────────────────
```

Highlight any draft >14 days old as `⚠️`. If `Daily closes` count is less than expected (e.g., for `week` it should normally be ~5–7), flag the gap: `⚠️ N días sin cierre`.

If no activity in the period:
> "No wiki activity in the last [period]."

### Step 6: No log entry needed

`digest` is read-only — do not append to log.md.
