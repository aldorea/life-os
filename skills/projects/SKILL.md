---
name: projects
description: Use when viewing projects, checking project health, or detecting stalled projects. Use when user says "projects", "proyectos", "project list", "stalled projects", "proyectos parados".
---

# projects

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Read-only view of all projects with health status and stalled detection. Reads project notes from the Projects folder and cross-references with the Backlog to check for next actions.

## Process

### 1. Read project files

Read all `.md` files in `VAULT/{config.structure.projects}/`. For each file:
- Read YAML frontmatter (expect fields: tags, status, last_updated)
- Extract project name from filename (without `.md` extension)
- Note the status field value (active, completed, archived, etc.)

If a project file has no frontmatter, treat it as active with last_updated unknown.

### 2. Cross-reference with Backlog

Read `VAULT/{config.structure.backlog}`. For each project:
- Search for unchecked tasks (`- [ ]`) containing the project's tag (e.g., #lifeos, #sherpa -- from `{config.tags.projects}`)
- Check if ANY unchecked task has the #next tag -- this is the project's "next action"
- Count total pending (unchecked) tasks for this project

### 3. Detect stalled projects

For each project with status = "active" (or #active tag):

a. **No next action:** If no unchecked task has #next tag for this project -> STALLED (reason: "sin next action")
b. **No progress:** If frontmatter `last_updated` is older than `{config.projects.stalled_days}` days (default: 14) -> STALLED (reason: "sin progreso en X dias")
c. Both conditions can be true simultaneously

### 4. Present

```
## Proyectos

| Proyecto | Estado | Next Action | Tareas pendientes | Ultima actualizacion | Alerta |
|----------|--------|-------------|-------------------|---------------------|--------|
| [[Life OS]] | active | "Implementar fase 1" | 5 | hace 3 dias | |
| [[Sherpa]] | active | NINGUNA | 2 | hace 18 dias | PARADO -- sin next action, sin progreso |
| [[Previene]] | active | "Revisar diseno" | 3 | hace 5 dias | |

---
Activos: X | Parados: Y | Completados: Z
```

If user says `/projects stalled` -- show only stalled projects.

### 5. Suggest actions for stalled projects

For each stalled project, show actionable suggestions:

```
Proyecto [[Sherpa]] esta parado:
- Sin next action: agrega una con /process-inbox o editando el Backlog
- Sin progreso en 18 dias: actualiza el estado del proyecto o archivalo
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| Projects folder empty | "No hay proyectos. Crea notas en {path}." |
| No frontmatter in project file | Treat as active, last_updated unknown |
| Backlog doesn't exist | Skip next action check, show "Backlog no encontrado" |
| stalled_days not in config | Default to 14 days |

## Important Rules

- Read-only -- NEVER modify any file
- Show ALL projects, not just stalled ones (unless user filters with `/projects stalled`)
- Stalled is a warning, not an error -- user decides what to do
- All paths resolved through `config.structure.projects` and `config.structure.backlog` -- never hardcoded
- Spanish language for all output
