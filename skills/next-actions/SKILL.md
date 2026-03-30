---
name: next-actions
description: Use when viewing next actions, filtering tasks by GTD context, or deciding what to do next. Use when user says "next actions", "proximas acciones", "que puedo hacer", "next", "actions", "tasks by context", "/next-actions #home".
---

# next-actions

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Read-only GTD view that shows next actions from the Backlog, optionally filtered by context tag. Does not modify any files.

Valid contexts are defined in `{config.tags.contexts}` (default: #home, #office, #calls, #computer).

## Process

### 1. Parse filter

Check if user provided a context filter:
- `/next-actions` -- no filter, show all grouped by context
- `/next-actions #home` -- filter by #home context
- `/next-actions #office` -- filter by #office
- Valid contexts: values from `{config.tags.contexts}`
- If invalid context provided, show available contexts and ask user to choose

### 2. Read Backlog

Read `VAULT/{config.structure.backlog}`. Parse all lines matching `- [ ] ` (unchecked tasks only). Skip checked tasks `- [x]`.

Exclude any task tagged #someday -- those belong in the /someday view.

### 3. Filter and group

**If context filter provided:**
Show only tasks containing that context tag. Also show tasks with #next tag that match the context. Sort by priority tags (#urgente first, then #importante, then others).

**If no filter:**
Group tasks by context tag:

```
## Next Actions

### #home (3)
- [ ] Tarea uno #personal #home #next
- [ ] Tarea dos #home
- [ ] Tarea tres #home #esperando

### #office (5)
- [ ] ...

### #calls (1)
- [ ] Llamar a [[Pedro]] #calls #next

### #computer (8)
- [ ] ...

### Sin contexto (X)
- [ ] Tasks without any context tag

---
Total: X next actions
#next: Y tareas marcadas como siguiente
#esperando: Z tareas en espera
```

**Ordering within each context group:**
1. Tasks tagged #next are shown **first** and highlighted
2. Tasks tagged #urgente shown next
3. Tasks tagged #importante shown next
4. Regular tasks (no priority/actionability tag)
5. Tasks tagged #esperando are shown **last** with "(esperando)" label

### 4. Show summary

```
Next Actions -- [#context or "Todos"]
Total: X tareas
Por contexto: #home (X), #office (X), #calls (X), #computer (X), sin contexto (X)
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| Backlog doesn't exist | "Backlog no encontrado en {path}. Crea tareas primero." |
| No tasks match filter | "No hay next actions para [context]. Prueba otro contexto." |
| No unchecked tasks | "Todas las tareas completadas. Buen trabajo!" |

## Important Rules

- Read-only -- NEVER modify any file
- Exclude #someday tasks (those belong in /someday view)
- Exclude checked tasks `- [x]`
- Preserve exact task text (don't reformulate)
- Spanish language for headers and messages
- All paths resolved through config -- never hardcoded
