---
name: morning
description: Use when starting the day, running the morning routine, or chaining all daily preparation. Use when user says "morning", "buenos dias", "manana", "start day", "empieza el dia", "rutina matutina".
---

# morning

## Step 0 -- Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Morning ritual orchestrator. Three phases: (1) sync all data sources via /sync, (2) process inbox interactively, (3) generate daily note. Each phase is independent -- if sync fails, inbox processing and daily note still run. The daily note generation is the core output and MUST succeed.

## Process

### 1. Announce start

```
Buenos dias! Iniciando rutina matutina...
```

### 2. Sync all data sources

Execute the `/sync` skill (defined in `skills/sync/SKILL.md`). This runs all configured connectors (calendar, jira, slack, reminders, granola, training) sequentially and produces a status report.

Capture the full /sync report output -- it will be included in the morning summary under the [Sync] section.

If /sync fails entirely (e.g., config.yaml missing connectors_config path): note "Sync no disponible" and continue to Step 3.

### 3. Process inbox

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

**This is the only interactive step** -- user confirms inbox processing before changes are applied.

**Result:** `X items procesados` | `vacio` | `skipped` (with reason)

### 4. Generate daily note

This is the core output -- it MUST succeed.

1. Read all vault data (calendar cache, backlog, jira notes, goals, constraints)
2. Determine focus of the day using goal alignment:
   - Read `VAULT/{config.structure.goals}` (YAML format)
   - Filter goals with status = "in_progress"
   - Score each: `weight * (1 - current/target) * deadline_urgency`
     - deadline_urgency = 1.0 if > 30 days, 1.5 if 14-30 days, 2.0 if < 14 days
   - Cross with today's calendar and backlog #next tasks
   - Pick highest-scoring goal as focus
3. Build top 3 active goals table (by weight)
4. Git safety -- pre-mutation:
   - `git -C VAULT add {daily_note_path}` (if exists)
   - `git -C VAULT commit -m "backup: before today mutation"` (continue if nothing to commit)
5. Generate or update daily note at `VAULT/{config.structure.daily_notes}/{date}.md`:
   - If exists: update only Foco, Objetivos activos, and Agenda sections
   - If new: create full template with all sections
6. Git safety -- post-mutation:
   - `git -C VAULT add {daily_note_path}`
   - `git -C VAULT commit -m "feat(today): generate daily note {date}"`
7. Add wikilinks for people, projects, Jira tickets

**Result:** `generada` | `actualizada` + path

### 5. Morning summary

Present results in two clear sections. The [Sync] section is the /sync report verbatim. The [Ritual] section covers inbox and daily note only. This prevents duplication of sync information.

```
Rutina matutina completada:

[Sync]
{/sync report output verbatim -- the "Sync completado:" block with check/X/dash lines}

[Ritual]
- Inbox: [X items procesados/vacio/skipped]
- Daily note: [generada/actualizada] {config.structure.daily_notes}/{date}.md

Foco del dia: [goal-aligned focus with progress]
Reuniones: X hoy
Foco disponible: ~Xh
[any alerts: goal deadlines < 14 days, dragged tasks, overdue items]
```

**CRITICAL -- what NOT to include in [Ritual]:** No per-connector sync lines (Calendar, Jira, Slack, Reminders, Granola). Those are already in the [Sync] section from /sync output. Including them again would be duplication.

## Graceful Degradation

| Phase | On failure | Impact |
|-------|-----------|--------|
| Sync (/sync) | Continue to inbox processing | Daily note lacks fresh sync data but still generates |
| Process inbox | Skip if error or empty | Inbox stays dirty, user can run `/process-inbox` manually |
| Generate daily note | MUST succeed | This is the core output -- if it fails, report error clearly |

## Important Rules

- Each phase is independent -- failure in sync NEVER blocks inbox or daily note
- Process inbox is the only interactive step (user confirms changes)
- Daily note generation MUST succeed -- it is the core output of the morning ritual
- Spanish language for all user-facing output
- If vault has no config.yaml, stop at Step 0 as usual
- If ALL syncs fail (via /sync), still generate the daily note -- it will degrade gracefully with available data
