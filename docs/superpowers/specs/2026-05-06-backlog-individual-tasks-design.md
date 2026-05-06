# Diseño: Backlog como notas individuales

**Fecha:** 2026-05-06  
**Estado:** Aprobado  

## Contexto

El backlog actual (`01 Backlog.md`) es un único archivo plano con ~60 tareas como checkboxes. No permite añadir contexto, comentarios ni subtareas por tarea. Tampoco es consultable como Obsidian Base.

## Objetivo

Migrar a notas individuales en `01 Tasks/` para:
- Añadir contexto, notas y subtareas por tarea
- Consultar el backlog mediante Obsidian Bases (tabla, kanban)
- Mantener captura rápida sin fricción extra

## Estructura de archivos

```
01 Tasks/
  DEVPT-478.md              ← tareas con referencia a ticket Jira
  IDP-confirmar-JWT.md      ← tareas libres (nombre-kebab-case)
  KS-workbench-guion.md
  ...
  Backlog.base              ← Base con vistas
  someday.md                ← archivo plano para tareas sin prioridad/semana
```

## Frontmatter de cada tarea

```yaml
---
status: todo | in-progress | waiting | someday | done
priority: urgente | importante | normal
project: miportal | orbitant | previene | lifeos | personal
week: 2026-W19        # semana objetivo (opcional)
due: 2026-05-13       # deadline duro (opcional)
context: [work, salud]  # tags libres
---

# Título de la tarea

Contexto, notas, subtareas, links — todo aquí en markdown libre.
```

### Valores de status

| Valor | Equivalencia actual |
|-------|---------------------|
| `todo` | `- [ ]` sin tag especial |
| `in-progress` | `- [ ] #next` activo esta semana |
| `waiting` | `- [ ] #esperando` |
| `someday` | `- [ ] #someday` |
| `done` | `- [x]` |

### Valores de priority

| Valor | Equivalencia actual |
|-------|---------------------|
| `urgente` | `#urgente` |
| `importante` | `#importante` |
| `normal` | sin tag de prioridad |

## Base: Backlog.base

Tres vistas:

### Vista 1: Hacer ya (table)
Filtro: `status == "todo" || status == "in-progress"` + `priority == "urgente" || priority == "importante"`  
Columns: nombre, priority, project, week, due

### Vista 2: Esta semana (table)
Filtro: `week == <semana-actual>`  
Columns: nombre, status, priority, project, due

### Vista 3: Kanban (cards)
Sin filtro de status  
Agrupado por: `status`  
Columns: nombre, priority, project

## Migración

### Qué se migra como nota individual
- Tareas con `#next`
- Tareas con `#urgente` o `#importante`
- Tareas con semana asignada (`#2026-W##`)

### Qué va a someday.md
- Tareas sin semana ni prioridad alta (`#someday`, sin tags de prioridad)

### Destino de 01 Backlog.md
- Se elimina tras confirmar migración completa

## Captura

### Rápida (sin cambios)
`02 Inbox.md` sigue siendo el punto de entrada: `- [ ] texto libre`  
El skill `/process-inbox` convierte inbox items en notas `01 Tasks/` al procesarlos.

### Detallada (skill /dump o /capture)
Crea nota directamente en `01 Tasks/` con frontmatter mínimo:
- `status: todo`
- `priority: normal` (ajustable)
- `project` inferido del contexto

## Skills afectadas

| Skill | Cambio |
|-------|--------|
| `rituals/morning` | Leer `01 Tasks/` en lugar de `01 Backlog.md` |
| `planning/review` | Leer `01 Tasks/` + actualizar lógica de revisión de backlog |
| `process-inbox` | Al mover al backlog, crear nota en `01 Tasks/` en lugar de añadir checkbox |
| `capture/dump` | Crear nota en `01 Tasks/` con frontmatter mínimo |
| `status` | Leer `01 Tasks/` para contar tareas pendientes |

## Config afectada

`config.yaml` → `config.structure.backlog` pasa de `01 Backlog.md` a `01 Tasks/`

## Lo que NO cambia

- `02 Inbox.md` — captura rápida sin fricción
- `06 Jira/` — tickets Jira siguen en su propia carpeta
- Numeración de carpetas del vault
