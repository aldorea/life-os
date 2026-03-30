---
name: weekly-review
description: Use when doing the weekly review, GTD review, or weekly planning session. Use when user says "weekly review", "revision semanal", "review", "revisemos la semana", "GTD review".
---

# weekly-review

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Interactive GTD weekly review facilitator. Walks through each review step, presenting data and waiting for user input at each stage. This is a conversation, not a report.

For generating/updating the weekly note without the interactive review, use `/week` instead.

## Process

### 1. Git safety -- pre-review backup

Before any vault writes during the review:

```
git -C VAULT add -A
git -C VAULT commit -m "backup: before weekly review"
```

If commit fails (nothing to commit), continue anyway.

### 2. Start review

```
Iniciamos la revision semanal. Voy a guiarte paso a paso.
Puedes escribir "saltar" en cualquier paso para ir al siguiente.
```

### 3. Paso 1: Retrospectiva

Read previous week's daily notes from `VAULT/{config.structure.daily_notes}/`.
Determine previous ISO week and read all daily notes whose `week` frontmatter matches.

Present:

```
## Paso 1: Retrospectiva

Completadas esta semana: X tareas (Y trabajo, Z personal)
Energia media: [emoji trend from Cierre sections]

Dias con energia alta: X
Dias con energia baja: Y

Logros destacados:
- [extracted from daily note logs and completions]
```

Ask: **"Como fue la semana? Algo que agregar a la retrospectiva?"**

Wait for user response. Capture their reflection.

### 4. Paso 2: Inbox Zero

Read `VAULT/{config.structure.inbox}`.

```
## Paso 2: Inbox Zero

Items en el inbox: X
```

- If inbox has items: **"Hay X items sin procesar. Procesamos ahora? (s/n)"**
  - If yes: run process-inbox logic inline (classify each item, suggest tags and section, confirm, apply to Backlog, clear from Inbox)
  - If no: "OK, queda pendiente. Recuerda procesar antes de empezar la proxima semana."
- If inbox is empty: "Inbox limpio! Bien hecho."

### 5. Paso 3: Salud del Backlog

Read `VAULT/{config.structure.backlog}`.

```
## Paso 3: Salud del Backlog

Total tareas pendientes: X
- #next: Y
- #esperando: Z
- Sin tag de accion: W

Arrastradas >{config.weekly.dragged_task_alert} dias: A tareas
[list dragged tasks with age in days]
```

If dragged tasks exist: **"Estas tareas llevan mas de {config.weekly.dragged_task_alert} dias. Para cada una: mantener / reprogramar / archivar / someday?"**

Present each dragged task for decision. Collect all decisions, then present a summary table of changes and ask: **"Aplicar estos cambios? (s/n)"**

Apply changes only after confirmation.

### 6. Paso 4: Proyectos Parados

Read all project notes from `VAULT/{config.structure.projects}/`.
Cross-reference each active project with `VAULT/{config.structure.backlog}` to detect stalled projects:
- A project is **stalled** if it has NO task with `#next` tag in the Backlog, OR its `last_updated` frontmatter is older than `{config.projects.stalled_days}` days.

```
## Paso 4: Proyectos Parados

Proyectos activos: X
Parados: Y

| Proyecto | Razon |
|----------|-------|
| [[Name]] | Sin next action |
| [[Name]] | Sin actividad hace Z dias |
```

For each stalled project: **"Proyecto [[Name]] esta parado: [reason]. Quieres agregar un next action? (escribelo o 'saltar')"**

Wait for user response. If user provides a next action, add it to Backlog with the project's tag + `#next`. Confirm before writing.

### 7. Paso 5: Algun Dia / Someday

Read someday items from `VAULT/{config.structure.backlog}` -- items tagged with `#someday` or in the "Algun dia" section.

```
## Paso 5: Algun Dia / Someday

Items en someday: X
[numbered list of items]
```

Ask: **"Hay algo que quieras promover a activo esta semana? (numeros separados por coma, o 'ninguno')"**

If user selects items: ask for context tag and week tag, move from someday section to active section in Backlog. Confirm before writing.

### 8. Paso 6: Progreso de Objetivos (GOAL-05)

Read `VAULT/{config.structure.goals}`.

If goals.yaml does not exist or has no goals, skip this step: "Sin objetivos definidos. Usa /goal add para crear el primero."

```
## Paso 6: Progreso de Objetivos

| Objetivo | Progreso | Cambio semanal | Deadline |
|----------|----------|----------------|----------|
| [name] | X/Y (Z%) | +delta desde ultimo snapshot | DD MMM |
```

The "Cambio semanal" column shows the difference between current value and the last `history[]` entry value. If no history exists, show "Sin historial previo".

For each `in_progress` goal: **"Progreso actual de '[name]': {current}. Actualizar? (nuevo valor o Enter para mantener)"**

Wait for user response. If user provides a new value, update `current` in goals.yaml.

After all goals reviewed, **auto-append a history[] entry** for each in_progress goal:

```yaml
- date: YYYY-MM-DD  # today's date
  value: {current value after any updates}
  note: "Weekly review snapshot"
```

This automatic weekly snapshot fulfills GOAL-05 (track goal progress over time).

Present changes and ask: **"Guardar progreso de objetivos? (s/n)"**

Apply only after confirmation. History entries are **append-only** -- never modify or remove past entries.

### 9. Paso 7: Plan de la Proxima Semana

Read calendar for next week (from `VAULT/{config.structure.calendar_cache}` if fresh, or run sync-calendar).
Read goals from `VAULT/{config.structure.goals}`.

```
## Paso 7: Plan de la Proxima Semana

Reuniones proxima semana: X (Yh)
Foco disponible: ~Zh

Prioridades sugeridas basadas en objetivos:
1. [highest weight in_progress goal with most gap to target]
2. [next]
3. [next]
```

Ask: **"Estas de acuerdo con las prioridades? Ajusta o confirma."**

Wait for user input. Capture confirmed priorities.

### 10. Generate weekly note

Run /week generation logic to create or update the weekly note at `VAULT/{config.structure.weekly_notes}/{week}.md` with all data collected during this review:
- Retrospective from paso 1
- Backlog health summary from paso 3
- Confirmed priorities from paso 7
- Capacity from paso 7

Follow the same template and rules as the `/week` skill. If the weekly note already exists, update only auto-generated sections -- never overwrite user-written content.

### 11. Git safety -- post-review commit

After all changes are written:

```
git -C VAULT add -A
git -C VAULT commit -m "ritual(weekly-review): week YYYY-W##"
```

### 12. Summary

```
Revision semanal completada:
- Retrospectiva: capturada
- Inbox: [limpio / X items procesados]
- Backlog: X tareas revisadas, Y reprogramadas
- Proyectos: Z parados atendidos
- Someday: W items promovidos
- Objetivos: progreso actualizado, snapshots guardados
- Plan proxima semana: prioridades confirmadas

Weekly note: VAULT/{config.structure.weekly_notes}/{week}.md
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| No daily notes for previous week | Skip retrospective, note "Sin dailies de la semana anterior" |
| Empty inbox | Skip inbox step, note "Inbox limpio" |
| No backlog file | Skip backlog health step, note "Sin backlog encontrado" |
| No projects folder | Skip stalled projects step, note "Sin carpeta de proyectos" |
| No goals.yaml | Skip goal progress step, note "Sin objetivos definidos" |
| No calendar data | Skip capacity calculation in planning step |
| User says "saltar" at any step | Move to next step immediately |

## Important Rules

- This is INTERACTIVE -- wait for user input at EVERY step
- NEVER auto-apply changes -- always present a summary and confirm before writing (per D-15)
- Git safety before any writes (per D-14)
- Each step reads FRESH data from the vault (not cached from the start of the review)
- User can skip any step by typing "saltar"
- Spanish language for all user-facing output
- Goal history snapshots are append-only -- never modify or remove past entries
- Wikilinks for all project and people references: `[[Name]]`
- All vault paths resolved through `config.structure.*` -- zero hardcoded paths
