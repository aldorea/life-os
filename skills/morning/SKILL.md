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

### 3. Sync Jira

Check if Jira connector is configured in `VAULT/{config.structure.connectors_config}`:
- If no `jira` section or file doesn't exist: skip, note "Jira sync no configurado"
- If configured:
  1. Read `jira.projects` array from connectors config
  2. For each project:
     - Build MCP tool: `mcp__{project.mcp_server}__jira_search`
     - Build JQL: `project = {project.project_key} AND assignee = '{assignee}' AND status in ('{status1}', '{status2}', ...)`
       - `assignee`: use `project.assignee_filter` if non-empty, otherwise fall back to `{config.daily.assignee_name}` from config.yaml
       - If BOTH are empty: omit the `assignee` clause entirely (sync all tickets regardless of assignee) and warn: "No hay filtro de assignee configurado para {project.project_key}. Sincronizando todos los tickets."
       - Statuses come from `project.sync_statuses` array
     - Call MCP `mcp__{project.mcp_server}__jira_search` with the JQL. If fails: warn "Jira sync failed for {project.project_key}: {error}. Saltando este proyecto.", skip this project, continue
     - For each ticket: call `mcp__{project.mcp_server}__jira_get_issue` with `issue_key` for full details (relationships, subtasks, epic, sprint). If single ticket fetch fails: skip that ticket, log warning, continue
     - Write/update note at `VAULT/{config.structure.jira_notes}/{ticket.key}.md` with full frontmatter:
       ```yaml
       ---
       key: {ticket.key}
       summary: "{ticket.summary}"
       status: {ticket.status}
       assignee: {ticket.assignee}
       priority: {ticket.priority}
       project: {project.project_key}
       sprint: {ticket.sprint_name or ""}
       due_date: {ticket.due_date or ""}
       created: {ticket.created_date}
       epic: {ticket.parent_epic_key or ""}
       type: {ticket.type}
       last_sync: {ISO 8601 timestamp of current sync}
       tags: jira
       ---
       ```
       And body with Relationships section (epic, linked issues, subtasks as wikilinks)
     - Re-sync overwrites entire note (Jira is source of truth per D-04)
     - For NEW tickets or action-requiring status changes: check inbox for `[[ticket.key]]` dedup (D-11), append if not present:
       - New assignment: `- Jira [[{key}]] -- Asignado: {summary} ({YYYY-MM-DD})`
       - Status change: `- Jira [[{key}]] -- Status changed to "{status}": {summary} ({YYYY-MM-DD})`
  3. Enrich with wikilinks (match people in `VAULT/{config.structure.people}/`, projects in `VAULT/{config.structure.projects}/`)

**Result:** `X tickets sincronizados, Y proyectos` | `skipped — no configurado` | `skipped — MCP no disponible`

### 4. Sync Slack

Check if Slack connector is configured in `VAULT/{config.structure.connectors_config}`:
- If no `slack` section or file doesn't exist: skip, note "Slack sync no configurado"
- If configured:
  1. Read `slack.channels` array from connectors config
  2. For each channel, use Slack MCP to read last 24h messages
  3. Extract: decisions, action items, questions pending, important announcements
  4. Ignore: casual conversation, resolved threads, bot messages
  5. Enrich with wikilinks (match people in `VAULT/{config.structure.people}/`)
  6. Check inbox for existing wikilinks before appending (D-11 dedup)
  7. Append to `VAULT/{config.structure.inbox}`:
     `- Slack #{channel} — [[Persona]]: description (YYYY-MM-DD)`

**Result:** `X items de Y canales` | `skipped — no configurado` | `skipped — MCP no disponible`

### 5. Sync Reminders

Check if Reminders connector is configured in `VAULT/{config.structure.connectors_config}`:
- If no `reminders` section or file doesn't exist: skip, note "Reminders sync no configurado"
- If configured:
  1. Read `reminders.lists` and `reminders.completed` from connectors config
  2. Load processed registry from `VAULT/{config.structure.cache}/reminders-processed.md` (create if missing)
  3. For each configured list, run osascript to fetch reminders:
     ```bash
     osascript -e '
     tell application "Reminders"
         set theList to list "{list_name}"
         set output to ""
         repeat with r in (reminders of theList whose completed is false)
             set rName to name of r
             set rYear to year of (creation date of r)
             set rMonth to month of (creation date of r) as integer
             set rDay to day of (creation date of r)
             set rDateStr to (rYear as string) & "-" & text -2 thru -1 of ("0" & rMonth) & "-" & text -2 thru -1 of ("0" & rDay)
             set rBody to ""
             try
                 set rBody to body of r
             end try
             set rDue to ""
             try
                 set dYear to year of (due date of r)
                 set dMonth to month of (due date of r) as integer
                 set dDay to day of (due date of r)
                 set rDue to (dYear as string) & "-" & text -2 thru -1 of ("0" & dMonth) & "-" & text -2 thru -1 of ("0" & dDay)
             end try
             set output to output & rName & "|" & rDateStr & "|" & rBody & "|" & rDue & linefeed
         end repeat
         return output
     end tell'
     ```
     **Important:** ISO date construction avoids locale-dependent formatting. Do NOT use `creation date of r as string`.
  4. Filter out already-processed reminders (match `name|creation_date` against registry)
  5. For each new reminder, check inbox for existing text (D-11 dedup), then append:
     - Basic: `- Reminder ({list_name}) -- {name} ({YYYY-MM-DD})`
     - With due date: `- Reminder ({list_name}) -- {name} [vence: {due_date}] ({YYYY-MM-DD})`
     - With body: `- Reminder ({list_name}) -- {name}: {body_text} ({YYYY-MM-DD})`
     - With both: `- Reminder ({list_name}) -- {name}: {body_text} [vence: {due_date}] ({YYYY-MM-DD})`
  6. Update processed registry with newly synced entries (append-only)
  7. If osascript returns "not authorized": print "Permiso de Reminders necesario. Ve a Ajustes del Sistema > Privacidad y Seguridad > Automatizacion > Claude/Terminal y activa el acceso a Reminders." and skip
  8. If osascript not available (non-macOS): skip with "Solo disponible en macOS"

**Result:** `X recordatorios nuevos` | `skipped — no configurado` | `skipped — permiso denegado` | `skipped — no macOS`

### 6. Sync Granola

Check if Granola data is available:
- Look for unprocessed Granola meeting notes (source-agnostic pattern: check configured meeting source)
- If no data or connector not configured: skip, note "Granola sync no disponible"
- If available: process meeting notes, extract attendees, create/update meeting notes in `VAULT/{config.structure.meetings}/`, update people's `last_interaction` in `VAULT/{config.structure.people}/`

**Result:** `synced` | `skipped` (with reason)

### 7. Process inbox

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

### 8. Generate daily note

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

### 9. Morning summary

Present the results of all steps:

```
Rutina matutina completada:
- Calendario: [synced/skipped + reason]
- Jira: [X tickets de Y proyectos/skipped + reason]
- Slack: [X items de Y canales/skipped + reason]
- Reminders: [X recordatorios nuevos/skipped + reason]
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
| Sync Jira | Skip project or all, note in summary | No Jira updates in daily note |
| Sync Slack | Skip, note in summary | No Slack updates |
| Sync Reminders | Skip, note in summary | Reminders stay in app, not in inbox |
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
