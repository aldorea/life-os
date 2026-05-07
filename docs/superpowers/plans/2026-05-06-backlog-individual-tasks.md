# Backlog: Migración a notas individuales — Plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrar el backlog de un único archivo de checkboxes a notas individuales en `01 Tasks/` consultables como Obsidian Base.

**Architecture:** Cada tarea vive como `.md` con frontmatter en `01 Tasks/`. Una Base (`Backlog.base`) expone 3 vistas. El inbox sigue siendo el punto de entrada rápido; el skill `/process-inbox` (dentro de `morning`) crea notas en lugar de checkboxes al procesar.

**Tech Stack:** Obsidian CLI, Obsidian Bases (.base YAML), skills markdown

---

### Task 1: Crear estructura de carpeta y someday.md

**Files:**
- Create: `VAULT/01 Tasks/someday.md`

- [ ] Crear la carpeta `01 Tasks` en el vault:
```bash
obsidian vault="Obsidian Vault" create path="01 Tasks/someday.md" content="---\ntags: someday\n---\n# Algún día\n\nTareas sin prioridad ni fecha — procesar en revisión semanal.\n\n"
```

- [ ] Verificar que el archivo existe:
```bash
obsidian vault="Obsidian Vault" read path="01 Tasks/someday.md"
```
Expected: frontmatter + header visible

- [ ] Commit:
```bash
git add -A && git commit -m "feat(tasks): create 01 Tasks folder with someday.md"
```

---

### Task 2: Actualizar config.yaml

**Files:**
- Modify: `~/.config/life-os/config.yaml`

- [ ] Cambiar `backlog: "01 Backlog.md"` por `backlog: "01 Tasks"`:
```yaml
structure:
  backlog: "01 Tasks"
```

- [ ] Verificar que el cambio es correcto:
```bash
grep "backlog:" ~/.config/life-os/config.yaml
```
Expected: `backlog: "01 Tasks"`

- [ ] Commit:
```bash
git add -A && git commit -m "chore(config): update backlog path to 01 Tasks"
```

---

### Task 3: Crear Backlog.base

**Files:**
- Create: `VAULT/01 Tasks/Backlog.base`

- [ ] Crear el archivo Base con 3 vistas:
```bash
obsidian vault="Obsidian Vault" create path="01 Tasks/Backlog.base" overwrite content="views:\n  - type: table\n    name: Hacer ya\n    filters:\n      and:\n        - 'status == \"todo\" || status == \"in-progress\"'\n        - 'priority == \"urgente\" || priority == \"importante\"'\n    order:\n      - file.name\n      - priority\n      - project\n      - week\n      - due\n\n  - type: table\n    name: Esta semana\n    filters:\n      and:\n        - 'week != \"\"'\n    order:\n      - file.name\n      - status\n      - priority\n      - project\n      - due\n\n  - type: cards\n    name: Kanban\n    groupBy:\n      property: status\n      direction: ASC\n    order:\n      - file.name\n      - priority\n      - project\n"
```

- [ ] Abrir Obsidian y verificar que `01 Tasks/Backlog.base` se abre correctamente con las 3 vistas (Hacer ya, Esta semana, Kanban). Estará vacío hasta la migración.

- [ ] Commit:
```bash
git add -A && git commit -m "feat(tasks): add Backlog.base with 3 views"
```

---

### Task 4: Migrar tareas activas

**Files:**
- Read: `VAULT/01 Backlog.md`
- Create: `VAULT/01 Tasks/*.md` (una por tarea activa)

Criterio de migración:
- ✅ Migrar: tareas con `#next`, `#urgente`, `#importante`, o `#2026-W##`
- 📄 A `someday.md`: tareas con `#someday` o sin prioridad/semana

**Mapeo de tags → frontmatter:**

| Tag inline | Frontmatter |
|-----------|-------------|
| `#urgente` | `priority: urgente` |
| `#importante` | `priority: importante` |
| sin tag | `priority: normal` |
| `#next` / `#2026-W##` | `status: in-progress` |
| `#esperando` | `status: waiting` |
| sin tag | `status: todo` |
| `#miportal` | `project: miportal` |
| `#orbitant` | `project: orbitant` |
| `#previene` | `project: previene` |
| `#lifeos` | `project: lifeos` |
| `#personal` | `project: personal` |
| `#work`, `#salud`, etc. | `context: [work]` |
| `📅 YYYY-MM-DD` | `due: YYYY-MM-DD` |
| `#2026-W##` | `week: 2026-W##` |

**Nombre del archivo:** kebab-case del título de la tarea, máx 50 chars. Si referencia un ticket Jira (`[[DEVPT-XXX]]`), usar `DEVPT-XXX.md`.

- [ ] Leer el backlog completo:
```bash
obsidian vault="Obsidian Vault" read path="01 Backlog.md"
```

- [ ] Para cada tarea activa, crear su nota. Ejemplo para "IDP: confirmar con David Yusta TTL del JWT":
```bash
obsidian vault="Obsidian Vault" create path="01 Tasks/IDP-confirmar-JWT-TTL.md" content="---\nstatus: in-progress\npriority: urgente\nproject: miportal\nweek: 2026-W19\ndue: \ncontext: [work]\n---\n\n# IDP: confirmar con David Yusta TTL del JWT interno, claims schema y provisioning de UserAppAccess\n\n[[David Yusta]] — confirmar antes de la reunión IDP Final Steps Review.\n" overwrite
```

- [ ] Repetir para cada tarea activa (ver lista completa en `01 Backlog.md`)

- [ ] Para tareas someday, añadir a `someday.md`:
```bash
obsidian vault="Obsidian Vault" append path="01 Tasks/someday.md" content="\n- [ ] [texto de la tarea someday]"
```

- [ ] Verificar conteo de notas creadas:
```bash
obsidian vault="Obsidian Vault" files folder="01 Tasks" total
```

- [ ] Commit:
```bash
git add -A && git commit -m "feat(tasks): migrate active backlog tasks to individual notes"
```

---

### Task 5: Actualizar skill rituals/morning

**Files:**
- Modify: `rituals/skills/morning/SKILL.md`

El paso "Process inbox" debe crear notas en `01 Tasks/` en lugar de añadir checkboxes al backlog.

- [ ] Actualizar el paso 3 (Process inbox), sección "On confirmation":

Cambiar:
```
6. On confirmation: move tasks to appropriate `{config.backlog_sections}` in Backlog, clear from Inbox
```

Por:
```
6. On confirmation: for each inbox item create a note in `01 Tasks/`:
   - File name: kebab-case del título, máx 50 chars
   - `obsidian vault="Obsidian Vault" create path="01 Tasks/{name}.md" content="---\nstatus: todo\npriority: {priority}\nproject: {project}\nweek: {week}\ndue: {due}\ncontext: [{context}]\n---\n\n# {title}\n" overwrite`
   - Clear from Inbox: `obsidian vault="Obsidian Vault" read path="{config.structure.inbox}"` + remove processed lines + `obsidian vault="Obsidian Vault" create path="{config.structure.inbox}" content="..." overwrite`
```

- [ ] Actualizar el paso 4 (Generate daily note), sección "Read all vault data":

Cambiar `{config.structure.backlog}` por `01 Tasks/`:
```
- Tareas activas: `obsidian vault="Obsidian Vault" files folder="01 Tasks"` + leer cada nota relevante
```

- [ ] Commit:
```bash
git add rituals/skills/morning/SKILL.md && git commit -m "feat(morning): update inbox processing to create task notes"
```

---

### Task 6: Actualizar skill planning/review

**Files:**
- Modify: `planning/skills/review/SKILL.md`

- [ ] En Step 2 (Process Inbox), actualizar "Apply":
```
6. Apply: create note in `01 Tasks/` per inbox item (same pattern as morning), clear Inbox
```

- [ ] En Step 3 (Review Backlog), cambiar todas las referencias a `01 Backlog.md`:
```
Run `obsidian vault="Obsidian Vault" files folder="01 Tasks"` to list all task notes.
For stale tasks: find notes where `week` property < current week and `status != "done"`.
For overload: count notes where `week == current-week`.
For goal alignment: check notes where `project` matches in-progress goals.
Apply changes with `obsidian vault="Obsidian Vault" property:set name=status value=someday path="01 Tasks/{file}"` etc.
```

- [ ] En Step 4 (Plan next week), cambiar referencia al backlog:
```
Suggest tagging tasks: `obsidian vault="Obsidian Vault" property:set name=week value="2026-W##" path="01 Tasks/{file}"`
```

- [ ] Commit:
```bash
git add planning/skills/review/SKILL.md && git commit -m "feat(review): update backlog references to 01 Tasks"
```

---

### Task 7: Actualizar skills close y status

**Files:**
- Modify: `rituals/skills/close/SKILL.md`
- Modify: (skill status — en skills/ del sistema, no del repo)

- [ ] En `close/SKILL.md`, Step 5 (Handle pending tasks):

Cambiar:
```
`obsidian vault="Obsidian Vault" read path="{config.structure.backlog}"`. Find unchecked tasks...
```

Por:
```
`obsidian vault="Obsidian Vault" files folder="01 Tasks"` — list task notes.
For pending tasks: find notes with `status: in-progress` or `status: todo` and `week == current-week`.
To reschedule: `obsidian vault="Obsidian Vault" property:set name=week value="2026-W##" path="01 Tasks/{file}"`
To move to someday: `obsidian vault="Obsidian Vault" property:set name=status value="someday" path="01 Tasks/{file}"`
```

- [ ] Commit:
```bash
git add rituals/skills/close/SKILL.md && git commit -m "feat(close): update pending task handling for 01 Tasks"
```

---

### Task 8: Eliminar 01 Backlog.md

- [ ] Confirmar que todas las tareas activas están migradas comparando conteo:
```bash
obsidian vault="Obsidian Vault" files folder="01 Tasks" total
```

- [ ] Confirmar que `someday.md` tiene las tareas sin prioridad

- [ ] Eliminar el archivo original:
```bash
obsidian vault="Obsidian Vault" delete path="01 Backlog.md"
```

- [ ] Commit final:
```bash
git add -A && git commit -m "chore(tasks): remove 01 Backlog.md after migration complete"
```

---

## Verificación final

- [ ] Abrir `01 Tasks/Backlog.base` en Obsidian — las 3 vistas muestran las tareas migradas
- [ ] Vista "Hacer ya" filtra correctamente por prioridad urgente/importante
- [ ] Vista "Esta semana" muestra tareas con `week: 2026-W19`
- [ ] Vista "Kanban" agrupa por status
- [ ] `/rituals:morning` procesa inbox y crea nota en `01 Tasks/` (no checkbox)
- [ ] `config.yaml` apunta a `01 Tasks`
