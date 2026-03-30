---
name: someday
description: Use when reviewing someday/maybe items, the "algun dia" list, or parking lot ideas. Use when user says "someday", "algun dia", "maybe", "someday maybe", "parking lot", "ideas aparcadas".
---

# someday

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Read-only view of someday/maybe items from the Backlog's "Algun dia" section. Designed for periodic review (weekly/quarterly). Does not modify files unless user explicitly enters review mode.

## Process

### 1. Read Backlog

Read `VAULT/{config.structure.backlog}`. Find the section matching the last entry in `{config.backlog_sections}` (the "Algun dia" section). Also find any tasks tagged #someday regardless of which section they appear in.

### 2. Collect someday items

Combine:
- All unchecked items (`- [ ]`) in the "Algun dia" section
- All unchecked items tagged #someday in any section
- Deduplicate (same item may appear in both)

### 3. Present

```
## Algun dia / Someday

X items en la lista:

- [ ] Aprender a tocar guitarra #personal #someday
- [ ] Escribir un libro sobre productividad #someday
- [ ] Curso de Machine Learning #professional #someday
...

---
Acciones:
- Para promover un item a activo: muevelo al Backlog con /process-inbox o edita directamente
- Para eliminar: marca como [x] completado o eliminalo del Backlog
- Proxima revision: en el weekly review (/weekly-review)
```

### 4. Review mode

If user says `/someday review`:

Present each item one at a time for interactive review:

```
1/X: "Aprender a tocar guitarra #personal #someday"
- Promover a activo? (s/n/saltar)
- Si si: pedir contexto tag (#home/#office/#calls/#computer) y mover a seccion apropiada del Backlog
- Si no: mantener en someday
- Si saltar: siguiente item
```

**IMPORTANT:** Review mode is a mutating flow. Before making any changes:
1. Git commit current state of `VAULT/{config.structure.backlog}` first (per D-14 git safety)
2. Show proposed changes and ask user confirmation before writing (per D-15 suggest & confirm)
3. Apply all confirmed changes at once after review completes

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| Backlog doesn't exist | "Backlog no encontrado en {path}." |
| No "Algun dia" section | "No hay seccion 'Algun dia' en el Backlog." |
| No #someday tasks and no items in section | "Lista someday vacia. Usa /dump o /process-inbox para agregar items." |

## Important Rules

- Default mode is read-only (just viewing)
- Review mode (`/someday review`) is mutating -- requires git commit before changes (D-14) and user confirmation per item (D-15)
- All paths resolved through `config.structure.backlog` and `config.backlog_sections` -- never hardcoded
- Spanish language for all output
