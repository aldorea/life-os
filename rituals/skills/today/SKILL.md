---
name: today
description: Use when the user wants to generate or update their daily note, start their day, or says "today", "hoy", "que tengo hoy", "genera daily", "daily note", "mi dia".
---

# today

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates or updates the daily note in Obsidian by reading existing vault data. Includes goal-aligned focus suggestion from goals.yaml and an active goals dashboard. Does not call external APIs — reads only markdown files that connectors have already written.

## Vault I/O

Use `obsidian` CLI for vault operations:
- Read files: `obsidian read path="..."`
- Create daily note (new): `obsidian create path="{daily_note_path}" content="..."` (full template as content with `\n` for newlines)
- Update daily note (existing): read with `obsidian read path="..."`, then rewrite updated sections with `obsidian create path="..." content="..." overwrite`
- Daily note path: `obsidian daily:path` (returns today's configured path)

## Process

### 1. Run sync-calendar first

Before generating, ensure calendar data is fresh:
- Execute the `sync-calendar` skill
- If it fails, continue anyway — the daily note will just lack calendar data

### 2. Read vault data

Read these files from the vault using `obsidian read path="{file}"` for each source (skip gracefully if any is missing):

| Source | File | What to extract |
|--------|------|-----------------|
| Calendar | `VAULT/{config.structure.calendar_cache}` | Today's events |
| Backlog | `VAULT/{config.structure.backlog}` | Tasks tagged `#next` or current week `#YYYY-W##` |
| Jira | `VAULT/{config.structure.jira_notes}/*.md` | Tickets with status != "Done" assigned to `{config.daily.assignee_name}` |
| People | `VAULT/{config.structure.people}/*.md` | Names for wikilink matching |
| Goals | `VAULT/{config.structure.goals}` | Active goals (YAML format — goals.yaml) |
| Constraints | `VAULT/{config.structure.constraints}` | Work hours, hard stops |

### 3. Determine the focus of the day

Read goals from `VAULT/{config.structure.goals}` (YAML format).
Parse the `goals` array. Filter to goals with status = "in_progress".

Focus selection algorithm:
1. Score each goal: `weight * (1 - current/target) * deadline_urgency`
   - deadline_urgency = 1.0 if deadline > 30 days, 1.5 if 14-30 days, 2.0 if < 14 days
2. Cross with today's calendar: if a meeting relates to a goal, boost that goal's score by 1.5x
3. Cross with backlog: if tasks tagged `#next` relate to a goal, include as actionable focus
4. Pick the highest-scoring goal as "Foco del dia"
5. Show: goal name, current progress (X/Y unit, Z%), and suggested action from `#next` tasks

Format in daily note:
> **Foco del dia:** [Goal name] — [current]/[target] [unit] ([%]) — [suggested next action]

### 4. Identify top 3 active goals

From the filtered in_progress goals, sort by weight descending. Take the top 3.
For each, compute: progress percentage = `(current / target) * 100`, and format the deadline as `DD MMM`.

### 5. Git safety — pre-mutation snapshot

Before writing/updating the daily note:
- `git -C VAULT add {daily_note_path}` (if file exists)
- `git -C VAULT commit -m "backup: before today mutation"` (if anything to commit, continue if nothing to commit)

### 6. Generate daily note

Check if today's daily note already exists at `VAULT/{config.structure.daily_notes}/{date}.md` (using `{config.daily.date_format}`):
- **If exists**: Only update the "Foco del dia", "Objetivos activos", and "Agenda" sections. Do NOT overwrite Log, Cierre, or any user-written content.
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

> **Foco del dia:** [Goal name] — [current]/[target] [unit] ([%]) — [suggested next action]

## Objetivos activos

| Objetivo | Progreso | Deadline |
|----------|----------|----------|
| [Goal 1 name] | X/Y unit (Z%) | DD MMM |
| [Goal 2 name] | X/Y unit (Z%) | DD MMM |
| [Goal 3 name] | X/Y unit (Z%) | DD MMM |

## Agenda

| Hora | Evento | Con | Lugar |
|------|--------|-----|-------|
| 09:00 | Daily standup | [[Pedro Lopez]], [[Maria Garcia]] | Teams |
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


## Cierre del dia

**Que salio bien?**


**Que haria diferente?**


**Energia:** rojo / amarillo / verde
```

### 7. Git safety — post-mutation commit

After writing:
- `git -C VAULT add {daily_note_path}`
- `git -C VAULT commit -m "feat(today): generate daily note {date}"`

### 8. Wikilinks

Every person, project, and Jira ticket mentioned should be a wikilink:
- People: `[[Nombre Apellido]]` — match against `VAULT/{config.structure.people}/` filenames
- Projects: `[[ProjectName]]` — match against `VAULT/{config.structure.projects}/`
- Jira: `[[TICKET-123]]` — link to `VAULT/{config.structure.jira_notes}/TICKET-123.md`

### 9. Present summary

After generating, show the user a brief summary:

```
Daily note generada: {config.structure.daily_notes}/{date}.md

Foco: [goal-aligned focus with progress]
Objetivos: [top 3 goals with progress %]
Reuniones: X reuniones, Xh de foco disponible
Tareas: X trabajo + X personal
[any alerts: dragged tasks, overdue items, hard stop conflicts, goal deadlines < 14 days]
```

## Graceful Degradation

| Missing data | Behavior |
|-------------|----------|
| No calendar cache | Skip Agenda section, note "Calendario no sincronizado" |
| No Backlog | Skip task sections, note "Backlog vacio o no encontrado" |
| No Jira folder | Skip Jira tasks |
| No People folder | Skip wikilinks for people |
| No goals config | Skip focus suggestion and Objetivos activos section, leave blank for user |
| No constraints config | Use default work hours from `{config.daily.default_work_hours}` |
| Goal has no current/target | Show "Sin datos" in progress column |
| goals.yaml has no in_progress goals | Show "Sin objetivos activos" in focus, skip Objetivos table |

## Important Rules

- NEVER delete or overwrite user content (Log, Cierre sections)
- NEVER duplicate tasks — the Tasks plugin queries handle filtering
- Keep the `tasks` code blocks exactly as shown — they are plugin queries, not static lists
- The Agenda section IS static (generated from calendar-cache), not a query
- The Objetivos activos table IS static (generated from goals.yaml), not a query
- All dates in Spanish format for display, ISO for frontmatter
- Day names in English for frontmatter `day` field, Spanish for the header
- Focus algorithm uses goals.yaml fields: weight, current, target, deadline, status
- When updating an existing note, preserve ALL content outside the auto-generated sections (Foco, Objetivos activos, Agenda)
