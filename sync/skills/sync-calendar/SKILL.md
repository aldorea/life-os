---
name: sync-calendar
---

# sync-calendar

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that reads calendar events via `{config.calendar.command}` and writes a markdown cache file to the Obsidian vault. Other skills (like `today` or `week`) read from this cache.

## Prerequisites

- `{config.calendar.command}` must be installed (e.g., `brew install ical-buddy` for icalBuddy)
- If not installed, warn the user and skip gracefully

## Process

### 1. Read events for today and next N days

Using `{config.calendar.command}` with scope of `{config.calendar.scope_days}` days:

```bash
# Today's events with details
{config.calendar.command} -f -ea -b "" -nc -nrd -df "%Y-%m-%d" -tf "%H:%M" -iep "title,datetime,location,attendees,notes" -ps "| — |" -po "datetime,title,attendees,location" eventsToday

# Future events (next N days)
{config.calendar.command} -f -ea -b "" -nc -nrd -df "%Y-%m-%d" -tf "%H:%M" -iep "title,datetime,location,attendees,notes" -ps "| — |" -po "datetime,title,attendees,location" eventsToday+{config.calendar.scope_days}
```

### 2. Parse and enrich with wikilinks

For each event:
- Extract: date, time, title, attendees, location
- Check if attendees have notes in `VAULT/{config.structure.people}/` — if yes, add `[[Person Name]]`
- Check if event title matches a project in `VAULT/{config.structure.projects}/` — if yes, add `[[Project]]`

### 3. Write cache file

Write to: `VAULT/{config.structure.calendar_cache}`

```markdown
---
last_sync: YYYY-MM-DD HH:MM
source: {config.calendar.command}
---

# Calendar Cache

## Hoy — YYYY-MM-DD

| Hora | Evento | Asistentes | Lugar |
|------|--------|------------|-------|
| 09:00 | Daily standup | [[Person]], [[Person]] | Teams |

## Próximos {config.calendar.scope_days} días

### YYYY-MM-DD (DayName)

| Hora | Evento | Asistentes | Lugar |
|------|--------|------------|-------|
| ... | ... | ... | ... |
```

### 4. Graceful degradation

- If calendar tool is not installed: write cache with message "Tool not installed"
- If no events found: write cache with "Sin eventos programados"
- If command fails: preserve previous cache, log error in cache header
- Never abort — always write something to the cache file
