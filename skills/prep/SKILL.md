---
name: prep
description: Use when the user wants to prepare for a meeting, get briefing context, or says "prep", "prepara reunión", "briefing", "qué sé de esta reunión", "prepare meeting", "1-on-1 prep".
---

# prep

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates a pre-meeting briefing by aggregating context from people notes, calendar, backlog, and Jira.

## Process

### 1. Identify the meeting

Options:
- User specifies: "prep para la reunión con Ana" or "prep PRESENTACION"
- No specification: pick the next upcoming meeting from `VAULT/{config.structure.calendar_cache}`

### 2. Read context

For each attendee in the meeting:

| Source | What to extract |
|--------|-----------------|
| `VAULT/{config.structure.people}/[name].md` | Role, company, last interaction, notes, pending items |
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
- **Rol:** [role from people note]
- **Última interacción:** YYYY-MM-DD — [summary]
- **Pendientes mutuos:**
  - [ ] Ella: [pending]
  - [ ] Tú: [pending]

### Contexto relevante
- [Active Jira tickets related]
- [Backlog tasks affecting attendees]
- [Recent decisions from previous meetings]

### Temas sugeridos
1. [Topic based on mutual pending items]
2. [Topic based on blocked/approaching tickets]
3. [Topic based on previous notes]

### Alineación con objetivos
- Este meeting contribuye a: [goal from goals config] (peso: X%)
- Relevancia: HIGH/MEDIUM/LOW
```

### 4. If `--save` flag

Write to `VAULT/{config.structure.daily_notes}/` or a prep file.

### 5. Graceful degradation

| Missing | Behavior |
|---------|----------|
| No People notes for attendees | Show names without context, suggest creating notes |
| No calendar cache | Ask user for meeting details |
| No Jira tickets | Skip Jira context section |
| No backlog tasks related | Skip tasks section |
| No goals config | Skip alignment section |

## Important Rules

- Output is conversational by default (not a file) — faster for quick prep
- Only save to file if user explicitly asks
- If attendee has no People note, offer to create one after the meeting
