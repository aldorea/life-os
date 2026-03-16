---
name: week
description: Use when the user wants to generate or update their weekly note, do weekly planning, or says "week", "semana", "weekly", "planifica semana", "revisión semanal", "qué tengo esta semana".
---

# week

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates or updates the weekly note in Obsidian. Includes retrospective of the previous week and planning for the current one.

## Process

### 1. Determine current week

Calculate current ISO week: `{config.weekly.format}`, start date (Monday), end date (Sunday).

### 2. Run sync-calendar

Execute `sync-calendar` skill to ensure calendar data is fresh.

### 3. Read vault data

| Source | File | What to extract |
|--------|------|-----------------|
| Previous week dailies | `VAULT/{config.structure.daily_notes}/*.md` | Completed tasks, energy levels, reflections |
| Backlog | `VAULT/{config.structure.backlog}` | Pending tasks with `#next` or current week tag |
| Jira | `VAULT/{config.structure.jira_notes}/*.md` | Active tickets |
| Calendar cache | `VAULT/{config.structure.calendar_cache}` | This week's meetings |
| Goals | `VAULT/{config.structure.goals}` | Active goals for alignment |
| Constraints | `VAULT/{config.structure.constraints}` | Work hours for capacity calculation |

### 4. Generate retrospective (if previous week dailies exist)

Summarize from previous week's daily notes:
- Tasks completed (count work vs personal)
- Energy trend (from Cierre del día entries)
- Key wins and blockers mentioned in logs
- Tasks dragged from previous week (pending > `{config.weekly.dragged_task_alert}` days = alert)

### 5. Suggest weekly priorities

Using goals config:
- List top 3 work priorities aligned with highest-weight in_progress goals
- List top 2 personal priorities
- Flag any goal with deadline approaching (< `{config.weekly.goal_deadline_warning}` days) and no progress

### 6. Calculate capacity

From constraints + calendar cache:
- Total work hours this week
- Hours in meetings
- Available focus time
- Days with gym (if tracked)

### 7. Generate or update weekly note

File: `VAULT/{config.structure.weekly_notes}/{week}.md`

If exists: update Prioridades and add retrospective, preserve user content.
If not exists: create with template.

```markdown
---
tags: weekly
week: YYYY-W##
date_start: YYYY-MM-DD
date_end: YYYY-MM-DD
---

# Semana ## · YYYY
`DD MMM` → `DD MMM`

---

## Retrospectiva semana anterior

**Completadas:** X tareas (Y trabajo, Z personal)
**Energía media:** 🟢/🟡/🔴
**Qué fue bien:** [extracted from dailies]
**Qué mejorar:** [extracted from dailies]
**Arrastradas:** [tasks pending > threshold days]

---

## Prioridades de la semana

### Trabajo
1. [Suggested from goals — highest weight in_progress]
2. [Next priority]
3. [Next priority]

### Personal
1. [Suggested]
2. [Suggested]

---

## Capacidad

| Métrica | Valor |
|---------|-------|
| Horas laborables | Xh |
| Reuniones programadas | Xh (Y reuniones) |
| Foco disponible | ~Xh |
| Días gimnasio planificados | X |

---

## Tareas de esta semana
```tasks
not done
(path includes {config.structure.backlog}) OR (path includes {config.structure.jira_notes})
(description includes #YYYY-W##) OR (description includes #next)
```

---

## Completadas esta semana
```tasks
done
done after YYYY-MM-DD
done before YYYY-MM-DD
```

---

## Reflexión
**¿Qué fue bien esta semana?**

**¿Qué mejorar?**

**¿Algo que deba eliminar o delegar?**
```

### 8. Present summary

```
Weekly note generada: {config.structure.weekly_notes}/{week}.md

📊 Retrospectiva: X completadas, energía media 🟡
📋 Prioridades sugeridas: [top 3]
📅 Capacidad: Xh foco disponible, Y reuniones
⚠️ [alerts: dragged tasks, approaching deadlines, goals at risk]
```

## Important Rules

- NEVER overwrite user-written reflections
- Priorities are SUGGESTIONS — present them for user approval
- Keep `tasks` code blocks as plugin queries, not static lists
- Retrospective only if previous week dailies exist — skip gracefully otherwise
