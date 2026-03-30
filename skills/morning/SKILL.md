---
name: morning
description: Use when starting the day, running the morning routine, or chaining all daily preparation. Use when user says "morning", "buenos dias", "manana", "start day", "empieza el dia", "rutina matutina".
---

# morning

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Morning ritual orchestrator. Runs all data sync + inbox processing + daily note generation in one command. Each step is independent — if one fails, the rest continue. The daily note generation is the core output and MUST succeed.

## Process

### 1. Announce start

```
Buenos dias! Iniciando rutina matutina...
```

### 2. Sync calendar

Read calendar data using the configured calendar tool (`{config.calendar.tool}`):

- Check if `{config.calendar.tool}` is set to "none" — if so, skip with note "Calendario deshabilitado en config"
- Run `{config.calendar.command}` to fetch today's events and the next `{config.calendar.scope_days}` days:
  ```bash
  {config.calendar.command} -f -ea -b "" -nc -nrd -df "%Y-%m-%d" -tf "%H:%M" -iep "title,datetime,location,attendees,notes" -ps "| — |" -po "datetime,title,attendees,location" eventsToday
  {config.calendar.command} -f -ea -b "" -nc -nrd -df "%Y-%m-%d" -tf "%H:%M" -iep "title,datetime,location,attendees,notes" -ps "| — |" -po "datetime,title,attendees,location" eventsToday+{config.calendar.scope_days}
  ```
- Parse output and enrich with wikilinks:
  - Check attendees against `VAULT/{config.structure.people}/` filenames — add `[[Person Name]]` if match found
  - Check event titles against `VAULT/{config.structure.projects}/` — add `[[Project]]` if match found
- Write cache to `VAULT/{config.structure.calendar_cache}`:
  ```markdown
  ---
  last_sync: YYYY-MM-DD HH:MM
  source: {config.calendar.command}
  ---

  # Calendar Cache

  ## Hoy — YYYY-MM-DD

  | Hora | Evento | Asistentes | Lugar |
  |------|--------|------------|-------|
  | HH:MM | Event title | [[Person]] | Location |

  ## Proximos {config.calendar.scope_days} dias

  ### YYYY-MM-DD (DayName)

  | Hora | Evento | Asistentes | Lugar |
  |------|--------|------------|-------|
  | HH:MM | Event title | [[Person]] | Location |
  ```
- If `{config.calendar.command}` is not installed or fails: skip, note "Calendario no sincronizado — herramienta no disponible"
- Never abort — always write something to the cache file (even if empty events)

**Result:** `synced` | `skipped` (with reason)

### 3. Sync Jira (Phase 2 — skip gracefully)

Check if Jira connector is configured in `VAULT/{config.structure.connectors_config}`:
- If not configured or file doesn't exist: skip, note "Jira sync no disponible (Phase 2)"
- If configured: read Jira data using the connector config and update notes in `VAULT/{config.structure.jira_notes}/`

**Result:** `synced` | `skipped — Phase 2`

### 4. Sync Slack (Phase 2 — skip gracefully)

Check if Slack connector is configured in `VAULT/{config.structure.connectors_config}`:
- If not configured or file doesn't exist: skip, note "Slack sync no disponible (Phase 2)"
- If configured: read Slack data and update relevant vault files

**Result:** `synced` | `skipped — Phase 2`

### 5. Sync Granola

Check if Granola data is available:
- Look for unprocessed Granola meeting notes (source-agnostic pattern: check configured meeting source)
- If no data or connector not configured: skip, note "Granola sync no disponible"
- If available: process meeting notes, extract attendees, create/update meeting notes in `VAULT/{config.structure.meetings}/`, update people's `last_interaction` in `VAULT/{config.structure.people}/`

**Result:** `synced` | `skipped` (with reason)

### 6. Process inbox

Read `VAULT/{config.structure.inbox}`:
- If file doesn't exist or is empty (no items below header): skip, note "Inbox vacio"
- If has items:
  1. Read `VAULT/{config.structure.backlog}` for duplicate detection
  2. Read `VAULT/{config.structure.goals}` for tag suggestions
  3. For each inbox item:
     - Reformulate vague items into concrete, actionable tasks
     - Detect duplicates against existing Backlog
     - Suggest tags from `{config.tags.contexts}`, `{config.tags.projects}`, `{config.tags.priority}`, `{config.tags.actionability}`
     - Add wikilinks for people, projects, tickets
  4. Present classification table to user:
     ```
     ## Inbox -> Backlog

     | # | Item original | Tarea propuesta | Tags | Seccion | Nota |
     |---|--------------|-----------------|------|---------|------|
     | 1 | raw item | Reformulated task | #tags | section | |
     ```
  5. Ask: "Aplico estos cambios? Puedes corregir cualquier linea."
  6. On confirmation: move tasks to appropriate `{config.backlog_sections}` in Backlog, clear from Inbox

**This is the only interactive step** — user confirms inbox processing before changes are applied.

**Result:** `X items procesados` | `vacio` | `skipped` (with reason)

### 7. Generate daily note

This is the core output — it MUST succeed.

Execute the full today skill logic:

1. Read all vault data (calendar cache, backlog, jira notes, goals, constraints)
2. Determine focus of the day using goal alignment:
   - Read `VAULT/{config.structure.goals}` (YAML format)
   - Filter goals with status = "in_progress"
   - Score each: `weight * (1 - current/target) * deadline_urgency`
     - deadline_urgency = 1.0 if > 30 days, 1.5 if 14-30 days, 2.0 if < 14 days
   - Cross with today's calendar and backlog #next tasks
   - Pick highest-scoring goal as focus
3. Build top 3 active goals table (by weight)
4. Git safety — pre-mutation:
   - `git -C VAULT add {daily_note_path}` (if exists)
   - `git -C VAULT commit -m "backup: before today mutation"` (continue if nothing to commit)
5. Generate or update daily note at `VAULT/{config.structure.daily_notes}/{date}.md`:
   - If exists: update only Foco, Objetivos activos, and Agenda sections
   - If new: create full template with all sections
6. Git safety — post-mutation:
   - `git -C VAULT add {daily_note_path}`
   - `git -C VAULT commit -m "feat(today): generate daily note {date}"`
7. Add wikilinks for people, projects, Jira tickets

**Result:** `generada` | `actualizada` + path

### 8. Morning summary

Present the results of all steps:

```
Rutina matutina completada:
- Calendario: [synced/skipped + reason]
- Jira: [synced/skipped — Phase 2]
- Slack: [synced/skipped — Phase 2]
- Granola: [synced/skipped + reason]
- Inbox: [X items procesados/vacio/skipped]
- Daily note: [generada/actualizada] {config.structure.daily_notes}/{date}.md

Foco del dia: [goal-aligned focus with progress]
Reuniones: X hoy
Foco disponible: ~Xh
[any alerts: goal deadlines < 14 days, dragged tasks, overdue items]
```

## Graceful Degradation

| Step | On failure | Impact |
|------|-----------|--------|
| Sync calendar | Skip, note in summary | Daily note lacks calendar data, Agenda section shows "Calendario no sincronizado" |
| Sync Jira | Skip (Phase 2) | No Jira task updates in daily note |
| Sync Slack | Skip (Phase 2) | No Slack updates |
| Sync Granola | Skip | No meeting note updates |
| Process inbox | Skip if error or empty | Inbox stays dirty, user can run `/process-inbox` manually |
| Generate daily note | MUST succeed | This is the core output — if it fails, report error clearly |

## Important Rules

- Each sync step is independent — failure in one NEVER blocks the next
- Process inbox is the only interactive step (user confirms changes)
- Daily note generation MUST succeed — it is the core output of the morning ritual
- All sync steps that write to vault use git safety (pre-mutation backup, post-mutation commit) per D-14
- Spanish language for all user-facing output
- If vault has no config.yaml, stop at Step 0 as usual
- Report the result of each step clearly in the morning summary
- If ALL syncs fail, still generate the daily note — it will degrade gracefully with available data
