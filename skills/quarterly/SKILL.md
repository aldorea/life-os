---
name: quarterly
description: Use when doing quarterly reflection, goal review, or planning the next quarter. Use when user says "quarterly", "trimestral", "quarterly review", "revision trimestral", "goal review", "revision de objetivos".
---

# quarterly

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Interactive quarterly reflection workflow. Reviews each goal's progress over the quarter, facilitates start/stop/continue analysis, and helps set next quarter goals. Generates a quarterly reflection note in the vault.

## Process

### 1. Determine quarter

Calculate current quarter and year:
- Q1 = Jan-Mar, Q2 = Apr-Jun, Q3 = Jul-Sep, Q4 = Oct-Dec

If user specifies a quarter (e.g., "Q1 2026"), use that instead of the current one.

Set `QUARTER` = "YYYY-Q#" (e.g., "2026-Q1").

### 2. Git safety -- pre-reflection backup

Before any vault writes:

```
git -C VAULT add VAULT/{config.structure.goals}
git -C VAULT commit -m "backup: before quarterly reflection"
```

If commit fails (nothing to commit), continue anyway.

### 3. Read goal data

Read `VAULT/{config.structure.goals}`.

If goals.yaml does not exist or has no goals:
- "No hay objetivos definidos. Usa /goal add para crear el primero."
- Stop here.

Filter goals relevant to this quarter:
- Goals with `horizon: quarterly` whose deadline falls within this quarter
- Goals with `horizon: annual` (always shown for progress check)
- Goals with `status: in_progress` or `status: not_started`

### 4. Goal-by-goal review

For each relevant goal, present:

```
## Objetivo: [name]

Dimension: [dimension]
Metrica: [metric]
Progreso: [current]/[target] [unit] ([percentage]%)
Deadline: [deadline]
Status: [status]

Historial este trimestre:
| Fecha | Valor | Nota |
|-------|-------|------|
[history entries whose date falls within this quarter]

Tendencia: [improving / declining / flat]
```

Trend is calculated from history entries:
- **Improving**: last 3+ entries show increasing values
- **Declining**: last 3+ entries show decreasing values
- **Flat**: values are stable or fewer than 3 entries

Ask: **"Como evaluas este objetivo? (1-5) y comentario"**

Wait for user response. Capture rating (1-5) and comment for each goal.

### 5. Start / Stop / Continue analysis

After reviewing all goals, present an initial analysis:

```
## Start / Stop / Continue

Basado en la revision:

**Continue (objetivos que van bien):**
- [goals with good progress and rating >= 4]

**Stop (candidatos a pausar o abandonar):**
- [goals with poor progress or rating <= 2]

**Start (nuevas ideas):**
- [to be filled by user]
```

Ask: **"Que quieres EMPEZAR el proximo trimestre?"**

Wait for response. Capture new ideas.

Ask: **"Algun objetivo actual que quieras PARAR o pausar?"**

Wait for response. If user wants to stop a goal, present: "Cambiar status de '[name]' a 'abandoned'. Confirmar? (s/n)"

Apply only after confirmation -- set `status: abandoned` in goals.yaml.

Ask: **"Algo mas que quieras CONTINUAR con ajustes? (nuevo target, deadline, peso)"**

Wait for response. If user wants to adjust a goal, present proposed changes and confirm before applying updates to target/deadline/weight in goals.yaml.

### 6. Set next quarter goals

Ask: **"Quieres definir nuevos objetivos para Q[next]? (s/n)"**

If yes, for each new goal walk through:
- **name**: nombre del objetivo
- **dimension**: professional / personal / health
- **horizon**: quarterly / annual
- **metric**: que se mide
- **target**: valor objetivo
- **unit**: unidad (optional)
- **weight**: peso (0.0-1.0)
- **deadline**: fecha limite

Present the new goal summary and ask: **"Agregar este objetivo? (s/n)"**

Add to goals.yaml with `status: not_started`, `current: 0`, and empty `history: []`.

After all new goals are added, validate weights:

```
Los pesos de objetivos quarterly suman X.
Los pesos de objetivos annual suman Y.
Deben sumar 1.0 cada grupo. [Ajustar si es necesario.]
```

If weights do not sum to 1.0, warn the user but do not block -- user adjusts at their discretion.

### 7. Generate quarterly note

Create `VAULT/{config.structure.weekly_notes}/YYYY-Q#-reflection.md`:

```markdown
---
tags: quarterly
quarter: YYYY-Q#
date: YYYY-MM-DD
---

# Reflexion Trimestral Q# YYYY

## Resumen de Objetivos

| Objetivo | Resultado | Rating | Comentario |
|----------|-----------|--------|------------|
| [name] | X/Y (Z%) | [1-5] | [user comment] |

## Progreso Global Ponderado: X%

[Weighted average: sum of (goal_percentage * weight) for all reviewed goals]

## Start / Stop / Continue

### Empezar
- [user inputs from step 5]

### Parar
- [user inputs from step 5]

### Continuar
- [user inputs from step 5]

## Objetivos Q[next]

| Objetivo | Dimension | Metrica | Target | Peso | Deadline |
|----------|-----------|---------|--------|------|----------|
| [new goals from step 6] | ... | ... | ... | ... | ... |

## Reflexion Personal

[Ask user: "Alguna reflexion personal sobre este trimestre?" -- capture free-form text]
```

### 8. Git safety -- post-reflection commit

After all changes are written:

```
git -C VAULT add -A
git -C VAULT commit -m "ritual(quarterly): Q# YYYY reflection"
```

### 9. Summary

```
Reflexion trimestral completada:
- Objetivos revisados: X
- Progreso global ponderado: Y%
- Parados/abandonados: Z
- Nuevos objetivos Q[next]: W
- Nota generada: VAULT/{config.structure.weekly_notes}/YYYY-Q#-reflection.md
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| No goals.yaml | "No hay objetivos definidos. Usa /goal add para crear el primero." Stop. |
| No history entries for a goal | Skip trend analysis for that goal, show "Sin historial" |
| User skips start/stop/continue | Generate note without that section |
| User skips new goals | Generate note without "Objetivos Q[next]" section |
| No calendar data for capacity | Skip capacity references |

## Important Rules

- This is INTERACTIVE -- wait for user input at each goal review and each analysis step
- NEVER auto-modify goals -- always present changes and confirm before writing (per D-15)
- Git safety before any writes (per D-14)
- Quarterly note stored alongside weekly notes using naming convention `YYYY-Q#-reflection.md`
- Spanish language for all user-facing output
- Weight validation after any goal changes -- warn but do not block
- Wikilinks for all project and people references: `[[Name]]`
- All vault paths resolved through `config.structure.*` -- zero hardcoded paths
- Goal status changes are explicit: only set `abandoned` when user confirms
- History entries are append-only -- never modify or remove past entries
