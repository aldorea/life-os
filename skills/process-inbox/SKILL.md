---
name: process-inbox
description: Use when processing the Obsidian Inbox, clarifying captured items, and moving them to the Backlog with proper tags. Use when user says "process inbox", "procesa inbox", "limpia inbox", "revisa inbox", "qué hay en el inbox".
---

# process-inbox

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Processes all items in the Inbox, clarifies them into actionable tasks, deduplicates against the Backlog, and moves them to the Backlog with proper tags. Leaves the Inbox clean.

## Process

### 1. Read Inbox and Backlog

| File | Purpose |
|------|---------|
| `VAULT/{config.structure.inbox}` | Items to process |
| `VAULT/{config.structure.backlog}` | Existing tasks (to detect duplicates) |
| `VAULT/{config.structure.goals}` | Active goals (to suggest tags) |

If Inbox is empty, inform the user and stop.

### 2. Classify each item

For each item in the Inbox:

1. **Reformulate** vague items into concrete, actionable tasks
   - Bad: "tema Jira" → Good: "Revisar configuración de Jira para proyecto X"
   - Bad: "hablar con Pedro" → Good: "Alinear con [[Pedro]] el scope del sprint"
2. **Detect duplicates** against existing Backlog tasks (fuzzy match on intent)
3. **Group related items** as subtasks under a parent task
4. **Suggest tags** for each task:
   - Context: from `{config.tags.contexts}` (e.g., #work, #personal)
   - Project: from `{config.tags.projects}`
   - Priority: from `{config.tags.priority}` (only if clearly warranted)
   - Actionability: from `{config.tags.actionability}` (e.g., #next, #esperando)
   - Week: `#YYYY-W##` for the current or next week

### 3. Present plan to user

Before modifying any file, show a table:

```
## Inbox → Backlog

| # | Item original | Tarea propuesta | Tags | Sección | Nota |
|---|--------------|-----------------|------|---------|------|
| 1 | hablar con Pedro | Alinear con [[Pedro]] el scope | #work #next | [section] | |
| 2 | tema Jira | — | — | — | Duplicado de "Configurar Jira..." |
```

Ask: "¿Aplico estos cambios? Puedes corregir cualquier línea."

### 4. Apply changes (only after confirmation)

1. **Add tasks** to `VAULT/{config.structure.backlog}` in the appropriate subsection from `{config.backlog_sections}`
2. **Remove processed items** from `VAULT/{config.structure.inbox}`
3. **Skip duplicates** — just remove from Inbox, don't add to Backlog
4. **Add wikilinks** for people, projects, and tickets

### 5. Summary

```
Inbox procesado:
- X tareas añadidas al Backlog
- X duplicados descartados
- X items reformulados
- Inbox limpio
```

## Backlog Sections

Place tasks in the right subsection using `{config.backlog_sections}`. Each section has a `name` and `filter` that determines which tags belong there.

If no subsection matches, create a new one following the pattern.

## Important Rules

- NEVER move items without user confirmation
- NEVER add duplicate tasks to the Backlog
- Always reformulate vague items — never copy raw text as-is
- Add wikilinks for any person, project, or ticket mentioned
- Keep the Inbox header and separator intact after clearing items
- Tasks in Backlog use `- [ ]` checkbox format
- "Algún día" items go to the last section (no week tag, no priority)
