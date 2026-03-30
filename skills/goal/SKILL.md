---
name: goal
description: Use when managing goals -- adding, updating, viewing, or tracking progress. Use when user says "goal", "objetivo", "meta", "add goal", "nuevo objetivo", "update goal", "goal progress", "progreso objetivo".
---

# goal

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

CRUD skill for goals stored in `VAULT/{config.structure.goals}` (goals.yaml). Supports add, update, list, progress, and remove operations. Goals are structured YAML entries with dimensions, horizons, weights, and progress history tracking.

## Goal Schema

Each goal in goals.yaml follows this structure:

```yaml
goals:
  - id: goal-unique-id       # * unique identifier (kebab-case)
    name: "Goal name"        # * human-readable goal description
    dimension: professional   # * professional | personal | health
    horizon: quarterly        # * quarterly | annual
    metric: "Metric name"    # * what is being measured
    target: 100              # * numeric target value
    current: 45              # * current numeric value
    unit: "%"                # ? unit of measurement (EUR, %, semanas, etc.)
    weight: 0.3              # * 0.0-1.0, weights within same horizon sum to 1.0
    deadline: 2026-06-30     # * target completion date (ISO format)
    status: in_progress      # * not_started | in_progress | completed | abandoned
    history:                 # progress snapshots (append-only)
      - date: 2026-03-22    # * snapshot date
        value: 40            # * value at that point
        note: "Description"  # * context for the change
```

## Process

### 1. Determine operation

Parse user input to identify the operation:
- `/goal` or `/goal list` -- list all goals grouped by dimension
- `/goal add` -- add a new goal
- `/goal update <id>` -- update a goal's fields
- `/goal progress <id> <value> [note]` -- record progress (append to history[])
- `/goal remove <id>` -- mark goal as abandoned (never delete)

### 2. Read goals.yaml

Read `VAULT/{config.structure.goals}`. Parse as YAML.

If the file doesn't exist, create it with:

```yaml
goals: []
```

Continue with the operation.

### 3. Execute operation

#### list

Group goals by dimension (professional, personal, health). For each goal show:

```
## Objetivos

### Profesional
| ID | Objetivo | Metrica | Progreso | Peso | Deadline | Estado |
|----|----------|---------|----------|------|----------|--------|
| revenue-q2 | Aumentar revenue... | Revenue mensual | 9500/12000 EUR (79%) | 35% | 2026-06-30 | in_progress |

### Personal
...

### Salud
...

Peso total por horizonte:
- Quarterly: X (debe ser 1.0)
- Annual: Y (debe ser 1.0)
```

Show progress as `current/target unit (percentage%)`.

If no goals exist, show: "No hay objetivos definidos. Usa `/goal add` para crear uno."

#### add

Ask user for each required field:

1. **id** -- suggest slug from name (kebab-case, unique)
2. **name** -- in Spanish
3. **dimension** -- professional | personal | health
4. **horizon** -- quarterly | annual
5. **metric** -- what is measured (e.g., "Revenue mensual")
6. **target** -- numeric target value
7. **current** -- numeric current value (default: 0)
8. **unit** -- optional unit of measurement (e.g., EUR, %, semanas, kg)
9. **weight** -- 0.0-1.0
10. **deadline** -- YYYY-MM-DD
11. **status** -- default: not_started

Initialize `history: []` (empty array).

**Before writing:** Present the complete goal object for user confirmation:

```
Nuevo objetivo:
  id: revenue-q2
  name: Aumentar revenue mensual
  dimension: professional
  horizon: quarterly
  metric: Revenue mensual
  target: 12000
  current: 0
  unit: EUR
  weight: 0.35
  deadline: 2026-06-30
  status: not_started

Confirmar? (s/n)
```

Only write after user confirms.

#### update <id>

1. Find goal by id. If not found, show error with available ids.
2. Show current values of the goal.
3. Ask which fields to change (any field except id and history).
4. Present updated goal for confirmation before writing.
5. Only write after user confirms.

#### progress <id> <value> [note]

1. Find goal by id. If not found, show error with available ids.
2. Update `current` to new value.
3. Append to `history[]`:

```yaml
- date: YYYY-MM-DD  # today's date
  value: <value>
  note: "<note or empty string>"
```

4. Show progress change:

```
revenue-q2: 9500 -> 10200 EUR (85% del target)
```

5. Confirm before writing.

#### remove <id>

1. Find goal by id. If not found, show error with available ids.
2. Set status to `abandoned`. Do NOT delete from file.
3. Show the goal being abandoned and confirm before writing.

### 4. Weight validation (per Pitfall 8)

After any add/update/remove operation, validate weights:

1. Sum weights of all non-abandoned goals with `horizon: quarterly`
2. Sum weights of all non-abandoned goals with `horizon: annual`
3. If either sum != 1.0, warn:

```
Los pesos de objetivos [horizon] suman X, deberian sumar 1.0. Ajustar? (s/n)
```

If user says yes, help them redistribute weights interactively.

### 5. Git safety (per D-14)

Before writing goals.yaml:

1. `git -C VAULT add {config.structure.goals}`
2. `git -C VAULT commit -m "backup: before goal mutation"`
3. Write changes to goals.yaml
4. `git -C VAULT add {config.structure.goals}`
5. `git -C VAULT commit -m "goal([operation]): [description]"`

If git is not available or VAULT is not a git repo, skip git steps and warn:

```
Nota: No se pudo hacer backup con git. Los cambios se escribieron directamente.
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| goals.yaml doesn't exist | Create with empty `goals: []` array and continue |
| Goal id not found | Show error message with list of available goal ids |
| git not available | Skip git backup/commit steps, warn user |
| config.structure.goals not set | Fall back to `config/goals.yaml` default path |
| Malformed YAML in goals.yaml | Show parse error, do NOT overwrite, ask user to fix manually |

## Important Rules

- NEVER delete goals -- only mark as abandoned with `status: abandoned`
- ALWAYS validate weights after mutations (add, update, remove)
- ALWAYS present changes for confirmation before writing (per D-15)
- Spanish language for all user-facing text
- `history[]` entries are append-only -- never modify or delete past entries
- `id` must be unique across all goals -- check before adding
- `current` should be between 0 and `target` -- warn if exceeded but allow it
- `history` entries should be in chronological order (newest last)
