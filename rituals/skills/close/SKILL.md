---
name: close
description: Use when ending the day, doing end-of-day reflection, or closing out the workday. Use when user says "close", "cierre", "end of day", "fin del dia", "cierra el dia", "reflection", "reflexion".
---

# close

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

End-of-day ritual that guides the user through reflection, energy check, and task completion logging. Updates the daily note's "Cierre del dia" section and handles pending task decisions.

## Vault I/O

Use `obsidian` CLI for vault operations:
- Read daily note: `obsidian daily:read`
- List completed tasks: `obsidian tasks done`
- List pending #next tasks: `obsidian tasks todo path="{config.structure.backlog}"`
- Append cierre section: `obsidian daily:append content="## Cierre del dia\n\n**Que salio bien?**\n{response}\n\n**Que haria diferente?**\n{response}\n\n**Energia:** {value}"`
- Toggle task done: `obsidian task toggle path="{config.structure.backlog}" line=N done`
- Read backlog: `obsidian read path="{config.structure.backlog}"`

## Process

### 0.4. Load critical facts

Read `08 Resources/CRITICAL_FACTS.md` as base context (role, timezone, manager, location, preferences). If missing, continue without warning — `morning` already warns once per day.

### 0.5. Sync Telegram captures

Run the `sync-telegram` skill to fetch any unprocessed messages from the Telegram bot before starting the close ritual. This ensures training logs and inbox items captured during the day are in the vault before reflection.

If sync-telegram fails or is unreachable, continue with the close ritual — don't block on it.

### 1. Read today's daily note

Use `obsidian daily:read` to read today's daily note.

If it doesn't exist: "No hay daily note para hoy. Quieres generarla primero? (/today)". Wait for response — if yes, run the today skill logic first; if no, proceed with reflection only (skip steps 2-3, go to step 4).

### 2. Read completed tasks

Use `obsidian tasks done` to list tasks completed today.
Also check today's daily note "Completadas hoy" section (rendered by Tasks plugin query).

Collect all tasks completed today into a list.

### 3. Review the day

Present a summary of the day to the user:

```
## Cierre del dia — {date}

Foco del dia: [from daily note's Foco del dia line]
Reuniones: X (counted from daily note Agenda table)

Tareas completadas hoy: Y
- [x] Tarea uno
- [x] Tarea dos

Tareas pendientes #next no completadas: Z
- [ ] Tarea pendiente uno
- [ ] Tarea pendiente dos
```

### 4. Interactive reflection

Ask the user these questions one at a time. Wait for each response before asking the next.

**"Que salio bien hoy?"**
Wait for user response. Capture their text.

**"Que harias diferente?"**
Wait for user response. Capture their text.

**"Energia del dia?"**
Show options: "1) Baja (roja) 2) Media (amarilla) 3) Alta (verde)"
Wait for user selection. Map: 1 = rojo, 2 = amarillo, 3 = verde.

### 5. Handle pending tasks

`obsidian vault="Obsidian Vault" files folder="01 Tasks"` — list task notes. Find notes with `status: in-progress` or `status: todo` and `week == current-week` via reading each note's frontmatter.

For each pending task, present options:

```
Tarea: "[task text]"
- (s) Mantener para mañana (sin cambio)
- (r) Reprogramar → obsidian property:set name=week value="YYYY-W##" path="01 Tasks/{file}"
- (m) Mover a algún día → obsidian property:set name=status value="someday" path="01 Tasks/{file}"
- (x) Descartar → obsidian delete path="01 Tasks/{file}"
```

Batch all decisions — present all pending tasks at once, collect all responses, then show the full plan:

```
## Cambios propuestos

| # | Tarea | Accion |
|---|-------|--------|
| 1 | Task text | Mantener #next |
| 2 | Task text | Reprogramar -> #YYYY-W## |
| 3 | Task text | Mover a Algun dia |

Aplicar estos cambios? (s/n)
```

Wait for confirmation before applying any changes (per D-15).

### 6. Git safety — pre-mutation snapshot

Before any writes:
- `git -C VAULT add {daily_note_path} {backlog_path}`
- `git -C VAULT commit -m "backup: before close ritual"` (continue if nothing to commit)

Where:
- `{daily_note_path}` = `{config.structure.daily_notes}/{date}.md`
- `{backlog_path}` = `{config.structure.backlog}`

### 7. Update daily note

Use `obsidian daily:append` to write to today's daily note's "Cierre del dia" section:

```markdown
## Cierre del dia

**Que salio bien?**
[user's response from step 4]

**Que haria diferente?**
[user's response from step 4]

**Energia:** [emoji based on selection: rojo / amarillo / verde]
```

Do NOT overwrite any other section of the daily note. Only replace the content within the "Cierre del dia" section (from `## Cierre del dia` until the next `##` heading or end of file).

### 8. Apply task changes

Based on Step 5 decisions (only after user confirmation):

- **Tasks kept as #next (s):** No change needed — they carry forward automatically
- **Tasks reprogrammed (r):** Update the week tag from current `#YYYY-W##` to the target week tag. Ask user which week if not specified.
- **Tasks moved to Algun dia (m):** Remove `#next` and week tags, add `#someday` tag. Move to the "Algun dia" section in Backlog (last section per `{config.backlog_sections}`).
- **Tasks marked irrelevant (x):** Mark as `- [x]` with note appended: "(Descartada)"

### 8b. Bridge daily → wiki log

Append a one-line pointer to the wiki activity log so the day shows up in `/wiki:digest` alongside ingests, queries, and synthesizes.

Build a one-line summary from what just happened:
- Foco del día (1 noun phrase)
- Tareas completadas: count
- Energía: rojo/amarillo/verde
- Decisiones notables (de "Que harias diferente?" si hay algo accionable)

Then:

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | daily | [[YYYY-MM-DD]]\nFoco: <foco>\nCompletadas: N | Energia: <color>\nNote: <one-sentence highlight>\n"
```

Use the wikilink format `[[YYYY-MM-DD]]` so the log entry navigates back to the daily note in Obsidian. The date inside the wikilink must match `{config.structure.daily_notes}` filename pattern (typically `DD-MM-YYYY` or `YYYY-MM-DD` depending on user setup — read the actual filename to be sure).

Skip this step if step 4 (interactive reflection) was skipped (no daily note generated).

### 8c. Drift detection — goals ausentes

Read `{config.structure.goals}` via `obsidian read path="{config.structure.goals}"`:
- On error or missing file → skip this step silently and proceed to Step 9 (R9).

Parse the file as YAML. Extract entries under `detection_goals:` where `status == "active"`:
- If no active detection goals exist → proceed to Step 9 with no output (R8).

**Scan each active goal:**

For each goal, walk backward from yesterday collecting existing daily notes:
- Format each date as `DD-MM-YYYY` (per `{config.daily.date_format}`)
- Call `obsidian read path="{config.structure.daily_notes}/{date}.md"` per day
- Skip days where the file does not exist (not counted as absences)
- Stop when `alert_after_days` existing notes have been found, OR 30 calendar days have elapsed
  - Note: `alert_after_days` is the target number of *existing notes* to scan, not calendar days. A goal with `alert_after_days: 3` whose user has 1 note in the last 30 days will check 1 note, not 3.
- Check case-insensitive keyword presence: for each found note, check whether any keyword appears (lowercase keyword `.includes()` check against lowercase note content)
- A goal is **flagged** when no keyword matched across all found notes AND at least 1 note was found

**If any goals are flagged:**

Show a warning block (in Spanish):

```
⚠️ Goals sin mención reciente:
- [goal name]: N notas revisadas, sin mención de [keywords]
```

For each flagged goal, offer a pause toggle:
```
Pausar "[goal name]"? (s/n)
```

Wait for user response for each. For each goal the user chooses to pause:
- Read `{config.structure.goals}` again, parse YAML
- Locate the goal by `name`, set `status: paused`, set `paused_since: {today as YYYY-MM-DD}`
- Active goals omit `paused_since` entirely; it is only present when `status == "paused"`. When resuming manually, set `status: active` and remove the `paused_since` field.
- Serialize with a YAML library and write back via `fs.writeFileSync` (not `obsidian CLI` — goals.yaml is not an Obsidian markdown file)
- If the write fails: surface the error to the user ("No se pudo guardar la pausa para [goal name]") but continue the ritual — do not abort
- Set `pause_written = true`

### 9. Git commit result

After all changes applied:
- `git -C {config.vault_path} add {daily_note_path} {backlog_path}`
- If `pause_written` is true from Step 8c: also add `git -C {config.vault_path} add {config.structure.goals}`
- `git -C {config.vault_path} commit -m "ritual(close): end-of-day reflection {date}"`

Note: paths in `git add` are vault-relative (the `-C` flag already sets CWD to the vault).

### 10. Good night summary

```
Dia cerrado.
Completadas: X tareas
Energia: [rojo/amarillo/verde]
Pendientes: Y tareas para manana
[if any moved to Algun dia: "Z tareas movidas a Algun dia"]
[if any discarded: "W tareas descartadas"]
```

**Paused goals (R12):** Before "Descansa bien!", always attempt to list paused detection goals:
- Read `{config.structure.goals}` via `obsidian read`; parse YAML
- If `detection_goals` contains any entries with `status == "paused"`, append:
  ```
  Goals pausados:
  - [goal name] (pausado desde [paused_since])
  ```
- If the file is missing or unreadable, skip this block silently (graceful degradation — R9 applies here too)

```
Descansa bien!
```

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| No daily note | Offer to create one (/today), or skip to reflection only (step 4) |
| No completed tasks | Skip completed tasks listing in step 3, still ask reflection questions |
| No pending #next tasks | Skip step 5 task review entirely |
| Backlog doesn't exist | Skip task management (steps 2, 3 task listing, 5, 8), do reflection only |
| Daily note has no Agenda section | Skip meeting count in step 3 review |
| Daily note has no Foco del dia | Show "Sin foco definido" in step 3 review |
| `goals.yaml` missing or unreadable | Skip Step 8c silently; skip paused-goals block in Step 10 silently |
| `goals.yaml` write failure (pause) | Surface error to user, continue ritual without applying the pause |

## Important Rules

- NEVER overwrite existing content in daily note sections other than "Cierre del dia"
- ALWAYS wait for user input at each reflection question (steps 4 and 5)
- ALWAYS present-then-confirm before modifying Backlog tasks (per D-15)
- Apply git safety before any writes (per D-14) — backup commit first, result commit after
- Spanish language for all user-facing output
- Energy mapping: baja = rojo, media = amarillo, alta = verde (matching daily note template)
- If user provides free-form energy response instead of number, map to closest option
- Batch task decisions — present all pending tasks, collect all responses, confirm once, then apply all
