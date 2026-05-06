---
name: capture
description: Use when capturing knowledge, interesting links, ideas, or learnings into the knowledge base. Use when user says "capture", "captura", "guarda esto", "esto es interesante", "save this", or shares a link/idea they want to remember.
---

# capture

## Overview

Captures knowledge inputs (articles, ideas, experiences, tools, learnings) into thematic notes in the knowledge base. Classifies, deduplicates, and connects with wikilinks.

## Input Types

The user can pass:
- A URL: `/capture https://blog.com/article`
- A text idea: `/capture "idea: usar granola para alimentar CRM automáticamente"`
- A free-form observation: `/capture me ha gustado cómo Felipe usa obsidian como CRM`
- A tool/resource: `/capture herramienta: Whisper Flow para escribir por voz`

## Vault I/O

Use `obsidian` CLI for vault operations:
- Read knowledge note: `obsidian read path="08 Resources/knowledge/{topic}.md"`
- Append entry: `obsidian append path="08 Resources/knowledge/{topic}.md" content="### [{date}] {title}\n**Tipo:** {type}\n**Fuente:** {source}\n\n{content}"`
- Update property: `obsidian property:set name=last_updated value="{date}" path="..."`
- Update maturity: `obsidian property:set name=maturity value="{seed|growing|ready}" path="..."`
- Update entries count: `obsidian property:set name=entries value=N path="..."`
- Create new topic: `obsidian create path="08 Resources/knowledge/{topic}.md" content="..."`

## Process

### 1. Understand the input

- If URL → fetch with WebFetch, extract key ideas (NOT copy entire article)
- If text → parse the insight directly
- If ambiguous → ask user to clarify

### 2. Identify the topic

- Check existing notes in `08 Resources/knowledge/` for matching topic
- If match found → will append to existing note
- If no match → propose a new topic name and ask user to confirm

### 3. Check for duplicates

- Read the target note's existing entries
- If the insight is already captured (same source or same idea) → tell user and skip
- If related but different angle → proceed, note the connection

### 4. Create the entry

Append to the note under `## Entradas`, most recent first:

```markdown
### [YYYY-MM-DD] Title or source description
**Tipo:** articulo | experiencia | idea | herramienta | lectura
**Fuente:** [URL or context description]

3-5 lines: what was learned, why it matters, key takeaway.
Connect with wikilinks: [[personas]], [[proyectos]], [[tickets]], [[other knowledge notes]].
```

### 5. Update frontmatter

Use `obsidian property:set` for each property:
- `obsidian property:set name=last_updated value="{today}" path="..."` → today
- `obsidian property:set name=entries value=N path="..."` → increment by 1
- `obsidian property:set name=maturity value="{seed|growing|ready}" path="..."` → recalculate: seed (<3), growing (3-6), ready (7+)

### 6. Check for content potential

If the entry suggests publishable content:
- Add idea to the `## Ideas para contenido` section of the same note
- Also add to `08 Resources/knowledge/content-ideas.md` with link to the knowledge note

### 7. Notify maturity changes

If the note just reached `ready` (7+ entries):
> "El tema '[topic]' tiene ya [N] entradas y está listo para generar contenido. ¿Quieres que prepare un draft?"

## Creating New Topic Notes

When a new topic is needed, create with this format:

```markdown
---
topic: Nombre del tema
tags: knowledge, [relevant tags]
last_updated: YYYY-MM-DD
entries: 1
maturity: seed
---

## Entradas

### [YYYY-MM-DD] First entry title
**Tipo:** [type]
**Fuente:** [source]

[Content]

## Ideas para contenido

```

## Important Rules

- ALWAYS ask before saving — never auto-save
- Capture the LEARNING, not the task: "discovered pattern X solves Y" > "did task Z"
- Keep entries to 3-5 lines — concise, not exhaustive
- Always use wikilinks to connect to People, Projects, Jira tickets
- Before creating a wikilink, verify the target note exists
- If source is a URL, include it for future reference
- Spanish language for all content

## Vault I/O

Use `obsidian vault="Obsidian Vault"` CLI for all vault reads and writes — never use the Read tool directly on vault files.
