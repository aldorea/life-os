---
name: sync-granola
---

# sync-granola

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that syncs Granola meeting data into the vault. Two outputs:
1. **Meeting notes** — individual meeting notes in `VAULT/{config.structure.meetings}/`
2. **People enrichment** — attendee People notes created/updated with `last_interaction`
3. **Backlog tasks** — action items extracted to `VAULT/{config.structure.backlog}`

## Prerequisites

- Granola MCP available (query_granola_meetings)
- `VAULT/{config.structure.granola_processed}` exists (dedup registry)

## Process

### 1. Fetch new meetings

Query Granola MCP for recent meetings. Cross-reference IDs against `VAULT/{config.structure.granola_processed}` to identify unprocessed ones. If no new meetings, report and stop.

### 2. Create meeting notes & enrich People notes

For each new meeting:

#### 2a. Create meeting note

Create a note at `VAULT/{config.structure.meetings}/YYYY-MM-DD-slug.md` where `slug` is a kebab-case version of the meeting title.

**Meeting note format:**

```markdown
---
date: YYYY-MM-DD
type: meeting
---
# Title — YYYY-MM-DD

**Asistentes:** [[Person 1]], [[Person 2]], ...

## Resumen
[3-5 lines from Granola notes]

## Decisiones
- [decisions or "—"]

## Action items
- [actions or "—"]
```

**Rules for meeting note generation:**
- Generate ONLY from Granola meeting notes — never invent information
- Focus on what was discussed, decided, and committed
- Keep summary to 3-5 lines max
- Action items here are INFORMATIONAL (not Backlog tasks) — they show what was committed in the meeting
- Use wikilinks for ALL person names: `[[Nombre Apellido]]`
- Do NOT include Alfonso himself as an attendee wikilink (he's the vault owner)

#### 2b. Enrich People notes

For each attendee of the meeting:

**If no note exists in `VAULT/{config.structure.people}/[name].md`** → create with this structure:

```yaml
---
name: [Full Name]
company: [inferred from meeting context, or empty]
role: [inferred from meeting context, or empty]
context: [work if work-related meeting, personal otherwise]
last_interaction: [meeting date YYYY-MM-DD]
---

## Perfil


## Personalidad


## Intereses


## Reuniones

```dataview
TABLE date as Fecha, file.link as Reunión
FROM "{config.structure.meetings}"
WHERE contains(file.outlinks, this.file.link)
SORT date DESC
```

## Notas personales

```

**If note already exists** → update `last_interaction` in frontmatter if this meeting date is more recent

### 3. Extract action items for Backlog

For each new meeting:

1. **Filter actions assigned to Alfonso** — discard tasks for other team members
2. **Cross-check with `VAULT/{config.structure.backlog}`** — skip duplicates
3. **Add new tasks to Backlog** in the appropriate subsection with tags:
   - `#work` or `#personal` based on meeting context
   - `#YYYY-W##` for the current week
   - Project tag if identifiable (`#miportal`, `#sherpa`, `#previene`, `#orbitant`)
4. **Evaluate priority** — apply `#urgente` and `#importante` criteria from `VAULT/{config.structure.claude_memory}/feedback_priority_criteria.md` with explicit reasoning
5. **Group related actions** — if multiple actions from different meetings are related, create a parent task with subtasks

### 4. Update processed registry

Append all processed meeting IDs to `VAULT/{config.structure.granola_processed}`.

### 5. Report

Output a summary to the user:

```
Granola sync completado:
- X reuniones procesadas
- Y notas de personas actualizadas (Z nuevas creadas)
- W tareas añadidas al Backlog
```

## Graceful Degradation

| Problem | Behavior |
|---------|----------|
| Granola MCP not available | Skip entirely, warn user |
| No new meetings | Report "No hay reuniones nuevas en Granola" and continue |
| Attendee name ambiguous | Create note with best guess, flag for user review |
| `granola-processed.md` missing | Create it, process all available meetings |
| People note has old format (embedded meeting blocks in ## Reuniones) | Replace with Dataview query, meetings already exist as separate notes |
| `VAULT/{config.structure.meetings}/` folder missing | Create it |

## Important Rules

- Never skip the People enrichment step — it's the core value of this connector
- Always use wikilinks when referencing people: `[[Nombre Apellido]]`
- If a person's name in Granola doesn't match any existing note, try fuzzy matching (first name + last name variations) before creating a new note
- When inferring company/role for new people, use the meeting context (title, other attendees, project) — leave empty if uncertain

