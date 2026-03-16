---
name: today
description: Use when the user wants to generate or update their daily note, start their day, or says "today", "hoy", "qué tengo hoy", "genera daily", "daily note", "mi día".
---

# today

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates or updates the daily note in Obsidian by reading existing vault data. Does not call external APIs — reads only markdown files that connectors have already written.

## Process

### 1. Run sync-calendar first

Before generating, ensure calendar data is fresh:
- Execute the `sync-calendar` skill
- If it fails, continue anyway — the daily note will just lack calendar data

### 2. Read vault data

Read these files from the vault (skip gracefully if any is missing):

| Source | File | What to extract |
|--------|------|-----------------|
| Calendar | `VAULT/{config.structure.calendar_cache}` | Today's events |
| Backlog | `VAULT/{config.structure.backlog}` | Tasks tagged `#next` or current week `#YYYY-W##` |
| Jira | `VAULT/{config.structure.jira_notes}/*.md` | Tickets with status != "Done" assigned to `{config.daily.assignee_name}` |
| People | `VAULT/{config.structure.people}/*.md` | Names for wikilink matching |
| Goals | `VAULT/{config.structure.goals}` | Active goals for focus suggestion |
| Constraints | `VAULT/{config.structure.constraints}` | Work hours, hard stops |

### 3. Determine the focus of the day

Using goals config:
- Look at highest-weight goals with status `in_progress`
- Cross with today's calendar (meetings related to a goal)
- Cross with backlog tasks tagged for this week
- Suggest 1 "Foco del día" — the most impactful thing to advance today

### 4. Generate daily note

Check if today's daily note already exists at `VAULT/{config.structure.daily_notes}/{date}.md` (using `{config.daily.date_format}`):
- **If exists**: Only update the "Foco del día" and "Agenda" sections. Do NOT overwrite Log, Cierre, or any user-written content.
- **If not exists**: Create new note with full template.

#### Template format

```markdown
---
tags: daily
date: YYYY-MM-DD
week: YYYY-W##
day: DayName
---

# DayName, DD Month YYYY

> **Foco del día:** [Suggested focus based on goals + calendar + tasks]

## Agenda

| Hora | Evento | Con | Lugar |
|------|--------|-----|-------|
| 09:00 | Daily standup | [[Pedro López]], [[María García]] | Teams |
| ... | ... | ... | ... |

**Tiempo de foco disponible:** ~Xh (based on gaps between meetings within {config.daily.default_work_hours})

## Trabajo

```tasks
not done
path includes {config.structure.backlog}
tags include #work
(tags include #next) OR (tags include #YYYY-W##)
sort by priority
```

```dataview
TASK
FROM "{config.structure.jira_notes}"
WHERE !completed AND file.frontmatter.assignee = "{config.daily.assignee_name}" AND file.frontmatter.status = "To Do" AND meta(section).subpath = "Tareas"
SORT file.name ASC
```

## Personal

```tasks
not done
path includes {config.structure.backlog}
(tags include #personal) OR (tags include #previene)
(tags include #next) OR (tags include #YYYY-W##)
sort by priority
```

## Completadas hoy

```tasks
done on YYYY-MM-DD
```

## Log


## Cierre del día

**¿Qué salió bien?**


**¿Qué haría diferente?**


**Energía:** 🔴 🟡 🟢
```

### 5. Wikilinks

Every person, project, and Jira ticket mentioned should be a wikilink:
- People: `[[Nombre Apellido]]` — match against `VAULT/{config.structure.people}/` filenames
- Projects: `[[ProjectName]]` — match against `VAULT/{config.structure.projects}/`
- Jira: `[[TICKET-123]]` — link to `VAULT/{config.structure.jira_notes}/TICKET-123.md`

### 6. Present summary

After generating, show the user a brief summary:

```
Daily note generada: {config.structure.daily_notes}/{date}.md

📋 Foco: [suggested focus]
📅 Reuniones: X reuniones, Xh de foco disponible
✅ Tareas: X trabajo + X personal
⚠️ [any alerts: dragged tasks, overdue items, hard stop conflicts]
```

## Graceful Degradation

| Missing data | Behavior |
|-------------|----------|
| No calendar cache | Skip Agenda section, note "Calendario no sincronizado" |
| No Backlog | Skip task sections, note "Backlog vacío o no encontrado" |
| No Jira folder | Skip Jira tasks |
| No People folder | Skip wikilinks for people |
| No goals config | Skip focus suggestion, leave blank for user |
| No constraints config | Use default work hours from `{config.daily.default_work_hours}` |

## Important Rules

- NEVER delete or overwrite user content (Log, Cierre sections)
- NEVER duplicate tasks — the Tasks plugin queries handle filtering
- Keep the `tasks` code blocks exactly as shown — they are plugin queries, not static lists
- The Agenda section IS static (generated from calendar-cache), not a query
- All dates in Spanish format for display, ISO for frontmatter
- Day names in English for frontmatter `day` field, Spanish for the header
