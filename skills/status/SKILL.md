---
name: status
description: Use when the user wants a general dashboard overview of their life, goals progress, or says "status", "estado", "dashboard", "como voy", "resumen general", "overview".
---

# status

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Read-only dashboard that shows a quick overview of goals (grouped by dimension with progress trends), tasks, calendar, training, and contacts. Does not modify any files.

## Process

### 1. Read all vault data

| Source | File | Format |
|--------|------|--------|
| Goals | `VAULT/{config.structure.goals}` | YAML (goals.yaml) |
| Backlog | `VAULT/{config.structure.backlog}` | Markdown |
| Jira | `VAULT/{config.structure.jira_notes}/*.md` | Markdown |
| Calendar | `VAULT/{config.structure.calendar_cache}` | Markdown |
| Training | `VAULT/{config.structure.training_log}` | Markdown |
| People | `VAULT/{config.structure.people}/*.md` | Markdown |
| Constraints | `VAULT/{config.structure.constraints}` | YAML |

### 2. Generate dashboard

Present in conversation:

```
## Status -- YYYY-MM-DD

### Objetivos Q# -- Profesional
| Objetivo | Peso | Progreso | Tendencia | Deadline | Estado |
|----------|------|----------|-----------|----------|--------|
| [goal name] | 35% | 79% (9500/12000 EUR) | +12% vs last | 2026-06-30 | in_progress |

### Objetivos Q# -- Personal
| Objetivo | Peso | Progreso | Tendencia | Deadline | Estado |
|----------|------|----------|-----------|----------|--------|
| [goal name] | 25% | 60% (3/5 libros) | +20% vs last | 2026-06-30 | in_progress |

### Objetivos Q# -- Salud
| Objetivo | Peso | Progreso | Tendencia | Deadline | Estado |
|----------|------|----------|-----------|----------|--------|
| [goal name] | 40% | 50% (12/24 semanas) | Sin cambios | 2026-06-30 | in_progress |

Progreso global ponderado: X%

### Backlog
- Pendientes: X tareas (Y urgentes, Z importantes)
- Arrastradas >2 semanas: X
- Tickets Jira activos: X

### Hoy
- Reuniones: X (Xh)
- Foco disponible: ~Xh
- Proxima reunion: [name] a las HH:MM

### Salud
- Sesiones esta semana: X/{config.training.weekly_goal} objetivo
- Ultima sesion: YYYY-MM-DD
- (si no hay training-log: "Sin datos de entrenamiento")

### Contactos
- Total: X personas
- Sin interaccion >{config.contacts.staleness_days} dias: X
  - [[Persona]] -- ultima vez: YYYY-MM-DD

### Alertas
- [goal and task alerts listed here]
```

### 3. Goal dimension grouping

Goals are read from `VAULT/{config.structure.goals}` as YAML. Group goals by `dimension` field:

- **Profesional** -- goals with `dimension: professional`
- **Personal** -- goals with `dimension: personal`
- **Salud** -- goals with `dimension: health`

Within each dimension, show only non-abandoned goals. Display the current quarter (Q#) in the section title based on today's date.

### 4. Tendencia (trend) column

For each goal, compare `current` to the last entry in the goal's `history[]` array:

- **If history has entries:** Calculate delta = `current - history[-1].value`. Show as percentage change relative to target:
  - Positive: `+X% vs last`
  - Negative: `-X% vs last`
  - Zero: `Sin cambios`
- **If no history entries:** Show `Sin datos`

Formula for percentage: `((current - history[-1].value) / target) * 100`

### 5. Progreso global ponderado

Calculate weighted global progress across all non-abandoned goals:

For each non-abandoned goal:
```
goal_progress = (current / target) * weight
```

Sum all `goal_progress` values. Show as percentage:
```
Progreso global ponderado: X%
```

This gives a single number representing overall progress weighted by goal importance.

### 6. Goal-specific alerts

Add the following alerts to the Alertas section:

**Deadline approaching with low progress:**
- Condition: goal deadline < 14 days away AND `(current / target) < 0.5`
- Alert: `Objetivo '[name]' al X% con deadline en Y dias`
- Use `{config.weekly.goal_deadline_warning}` for the day threshold if configured (default 14)

**No progress registered:**
- Condition: goal has `status: in_progress` AND `history[]` is empty or has no entries AND goal was presumably created > 7 days ago (check if deadline is more than 7 days from today, indicating the goal is not brand new)
- Alert: `Objetivo '[name]' sin progreso registrado`

**Weight sum mismatch:**
- Condition: sum of weights for non-abandoned goals in a given horizon != 1.0
- Alert: `Pesos de objetivos [horizon] suman X, ajustar con /goal`

### 7. Graceful degradation

Each section is independent. If a source is missing, show "Sin datos" for that section and continue with the rest.

| Missing | Behavior |
|---------|----------|
| goals.yaml doesn't exist | Show "Sin objetivos definidos. Usa /goal add para crear uno." |
| goals.yaml has no goals | Same as above |
| Backlog.md missing | Show "Sin datos de backlog" |
| Calendar cache missing | Show "Sin datos de calendario" |
| Training log missing | Show "Sin datos de entrenamiento" |
| People folder empty | Show "Sin contactos registrados" |
| goal history[] empty | Show "Sin datos" in Tendencia column |

## Important Rules

- Read-only -- NEVER modify any file
- Keep output scannable -- numbers, not paragraphs
- Alerts only for actionable items
- Show staleness for contacts based on `last_interaction` in People frontmatter
- Goals grouped by dimension, not as a flat list
- Spanish language for all user-facing text
