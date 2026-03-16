---
name: status
description: Use when the user wants a general dashboard overview of their life, goals progress, or says "status", "estado", "dashboard", "cómo voy", "resumen general", "overview".
---

# status

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Read-only dashboard that shows a quick overview of goals, tasks, calendar, training, and contacts. Does not modify any files.

## Process

### 1. Read all vault data

| Source | File |
|--------|------|
| Goals | `VAULT/{config.structure.goals}` |
| Backlog | `VAULT/{config.structure.backlog}` |
| Jira | `VAULT/{config.structure.jira_notes}/*.md` |
| Calendar | `VAULT/{config.structure.calendar_cache}` |
| Training | `VAULT/{config.structure.training_log}` |
| People | `VAULT/{config.structure.people}/*.md` |
| Constraints | `VAULT/{config.structure.constraints}` |

### 2. Generate dashboard

Present in conversation:

```
## Status — YYYY-MM-DD

### Objetivos Q#
| Objetivo | Peso | Estado | Progreso | Deadline |
|----------|------|--------|----------|----------|
| [goal] | X% | [status] | [progress] | [date] |

### Backlog
- Pendientes: X tareas (Y urgentes, Z importantes)
- Arrastradas >2 semanas: X
- Tickets Jira activos: X

### Hoy
- Reuniones: X (Xh)
- Foco disponible: ~Xh
- Próxima reunión: [name] a las HH:MM

### Salud
- Sesiones esta semana: X/{config.training.weekly_goal} objetivo
- Última sesión: YYYY-MM-DD
- (si no hay training-log: "Sin datos de entrenamiento")

### Contactos
- Total: X personas
- Sin interacción >{config.contacts.staleness_days} días: X
  - [[Persona]] — última vez: YYYY-MM-DD

### Alertas
- ⚠️ [goal approaching deadline with no progress]
- ⚠️ [tasks dragged >2 weeks]
- ⚠️ [stale contacts]
```

### 3. Graceful degradation

Each section is independent. If a source is missing, show "Sin datos" for that section and continue with the rest.

## Important Rules

- Read-only — NEVER modify any file
- Keep output scannable — numbers, not paragraphs
- Alerts only for actionable items
- Show staleness for contacts based on `last_interaction` in People frontmatter
