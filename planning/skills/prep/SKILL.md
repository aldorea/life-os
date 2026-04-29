---
name: prep
description: Use when the user wants to prepare for a meeting, get briefing context, or says "prep", "prepara reunión", "briefing", "qué sé de esta reunión", "prepare meeting", "1-on-1 prep".
---

# prep

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates a pre-meeting briefing by aggregating context from people notes, calendar, backlog, and Jira. Helps arrive at meetings prepared with full context.

## Process

### 1. Identify the meeting

Options:
- User specifies: "prep para la reunión con Ana" or "prep PRESENTACION PREVIENE"
- No specification: pick the next upcoming meeting from `VAULT/{config.structure.calendar_cache}`

### 2. Read context

For each attendee in the meeting:

| Source | What to extract |
|--------|-----------------|
| `VAULT/{config.structure.people}/[name].md` — frontmatter | Role, company, `total_meetings`, `last_interaction` |
| `VAULT/{config.structure.meetings}/*.md` | Last 3 meeting notes that link to the attendee (via wikilinks) — title, summary, decisions |
| `VAULT/{config.structure.people}/[name].md` — `## Personalidad` | Communication style (if has content) |
| `VAULT/{config.structure.people}/[name].md` — `## Intereses` | Topics they care about (if has content) |
| `VAULT/{config.structure.backlog}` | Tasks that mention this person or their project |
| `VAULT/{config.structure.jira_notes}/*.md` | Tickets related to the person's project |
| `VAULT/{config.structure.calendar_cache}` | Recent/upcoming meetings with same people |

### 3. Generate briefing

Present in conversation (not saved as file unless user asks with `--save`):

```markdown
## Prep: [Meeting Title]
**Cuándo:** YYYY-MM-DD HH:MM — HH:MM
**Dónde:** [Location/Link]

### Asistentes

#### [[Persona 1]]
- **Rol:** Product Owner, Empresa X
- **Te has reunido X veces** (última: YYYY-MM-DD)
- **Personalidad:** [from ## Personalidad, if exists]
- **Intereses:** [from ## Intereses, if exists]
- **Últimas reuniones:**
  - [YYYY-MM-DD] Título → resumen 1 línea + decisiones clave
  - [YYYY-MM-DD] Título → resumen 1 línea + decisiones clave
  - [YYYY-MM-DD] Título → resumen 1 línea + decisiones clave
- **Pendientes en Backlog:** [tasks from Backlog mentioning this person]

#### [[Persona 2]]
...

### Contexto relevante
- [Tickets Jira activos relacionados]
- [Tareas del backlog que afectan a estos asistentes]
- [Decisiones recientes de reuniones anteriores]

### Temas sugeridos
1. [Tema basado en pendientes mutuos]
2. [Tema basado en tickets bloqueados o próximos a deadline]
3. [Tema basado en notas previas]

### Alineación con objetivos
- Este meeting contribuye a: [goal from {config.structure.goals}] (peso: X%)
- Relevancia: HIGH/MEDIUM/LOW
```

### 4. If `--save` flag

Write to `VAULT/{config.structure.daily_notes}/` or a prep file: `prep-YYYY-MM-DD-[meeting-slug].md`

### 5. Graceful degradation

| Missing | Behavior |
|---------|----------|
| No People notes for attendees | Show names without context, suggest creating notes |
| No calendar cache | Ask user for meeting details |
| No Jira tickets | Skip Jira context section |
| No backlog tasks related | Skip tasks section |
| No goals.yaml | Skip alignment section |
| No Reuniones section | Show "Sin historial de reuniones" |
| No Personalidad/Intereses | Omit those lines |
| total_meetings = 0 or missing | Show "Primera reunión con esta persona" |

## Important Rules

- Output is conversational by default (not a file) — faster for quick prep
- Only save to file if user explicitly asks
- If attendee has no People note, offer to create one after the meeting
- Suggest updating People notes after the meeting with new info

