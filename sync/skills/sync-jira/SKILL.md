---
name: sync-jira
description: Use when syncing Jira tickets to the vault. Use when user says "sync jira", "sincroniza jira", "tickets", or invoked by /sync.
---

# sync-jira

## Step 0 -- Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that syncs Jira tickets from multiple configured projects via MCP servers into individual vault notes in `VAULT/{config.structure.jira_notes}/`. Each Jira project maps to a configured MCP server (per D-01). Creates inbox items only for new assignments or action-requiring status changes (per D-05). Re-sync overwrites all frontmatter and body -- Jira is source of truth (per D-04).

## Vault I/O

Use `obsidian` CLI for vault writes:
- Write/overwrite ticket note: `obsidian create path="{config.structure.jira_notes}/{ticket.key}.md" content="..." overwrite`
- Update single property: `obsidian property:set name=last_sync value="{timestamp}" path="{config.structure.jira_notes}/{ticket.key}.md"`
- Append inbox item: `obsidian append path="{config.structure.inbox}" content="- Jira [[{key}]] -- {message} ({date})"`
- Check inbox for duplicate: `obsidian read path="{config.structure.inbox}"` then grep for `[[{ticket.key}]]`

## Prerequisites

- At least one Jira MCP server must be available (e.g. `mcp__jira-afianza__jira_search`)
- `VAULT/{config.structure.connectors_config}` must contain a `jira:` section
- If no MCP server responds, warn and skip gracefully

## Process

### 1. Read Jira project config

`obsidian vault="Obsidian Vault" read path="{config.structure.connectors_config}"` and parse the `jira.projects` array.

If no `jira:` section exists or `jira.projects` is empty:
- Warn: "No hay proyectos Jira configurados en connectors.yaml. Configura al menos un proyecto para sincronizar."
- Stop Jira sync (do not abort other connectors if running in morning chain).

### 2. For each configured project, fetch tickets via MCP

For each project entry in `jira.projects`:

1. **Build MCP tool name:** `mcp__{project.mcp_server}__jira_search`
2. **Build JQL query:**
   ```
   project = {project.project_key} AND assignee = '{assignee}' AND status in ('{status1}', '{status2}', ...)
   ```
   - `assignee`: use `project.assignee_filter` if non-empty, otherwise fall back to `{config.daily.assignee_name}` from config.yaml
   - If BOTH are empty: omit the `assignee` clause entirely (sync all tickets regardless of assignee) and warn: "No hay filtro de assignee configurado para {project.project_key}. Sincronizando todos los tickets."
   - Statuses come from `project.sync_statuses` array
3. **Call** `mcp__{project.mcp_server}__jira_search` with the JQL
4. **If MCP tool fails or returns error:** log warning "Jira sync failed for {project.project_key}: {error}. Saltando este proyecto." and continue to next project
5. **For each ticket in results**, call `mcp__{project.mcp_server}__jira_get_issue` with `issue_key` to get full details (relationships, subtasks, epic, sprint)
6. **If a single ticket fetch fails:** skip that ticket, log warning, continue with remaining tickets

### 3. Write/update ticket notes in 06 Jira/

Ensure `VAULT/{config.structure.jira_notes}/` directory exists. If not, create it.

Write ticket notes using `obsidian create path="{config.structure.jira_notes}/{ticket.key}.md" content="..." overwrite` for each fetched ticket.

**Frontmatter (per D-03):**

```yaml
---
key: {ticket.key}
summary: "{ticket.summary}"
status: {ticket.status}
assignee: {ticket.assignee}
priority: {ticket.priority}
project: {project.project_key}
sprint: {ticket.sprint_name or ""}
due_date: {ticket.due_date or ""}
created: {ticket.created_date}
epic: {ticket.parent_epic_key or ""}
type: {ticket.type}
last_sync: {ISO 8601 timestamp of current sync}
tags: jira
---
```

**Body:**

```markdown
# {ticket.key}: {ticket.summary}

**Status:** {status} | **Priority:** {priority} | **Sprint:** {sprint or "--"}

## Relationships

- **Epic:** [[{epic_key}]] (or "--" if none)
- **Linked issues:** [[{linked_key1}]], [[{linked_key2}]] (or "--" if none)
- **Subtasks:**
  - [[{subtask_key}]] - {subtask_summary} (or "--" if none)
```

**Per D-04:** Re-sync overwrites ALL frontmatter and regenerates the body completely. Jira is source of truth. No user-editable sections are preserved.

### 4. Determine inbox-worthy changes (per D-05)

Compare current ticket data against existing note frontmatter (if note existed before this sync):

- **New ticket** (no prior note in `VAULT/{config.structure.jira_notes}/`): create inbox item IF assignee matches user (`project.assignee_filter` or `config.daily.assignee_name`)
- **Status changed to action-requiring status**: create inbox item. Action-requiring transitions are:
  - TO "To Do" (newly assigned or reassigned)
  - TO "In Review" (needs review action)
  - Any transition where the new status differs from the previously stored `status` frontmatter AND the new status is in `project.sync_statuses`
- **All other updates** (priority change, sprint change, summary edit): update note silently, NO inbox item

### 5. Write inbox items

For each inbox-worthy ticket:

1. **Read** `VAULT/{config.structure.inbox}` using `obsidian read path="{config.structure.inbox}"`
2. **Dedup check (D-11):** search for `[[{ticket.key}]]` in inbox content. If already present, skip (do not duplicate).
3. If NOT already in inbox, append using `obsidian append` the appropriate format:

**New assignment:**
```markdown
- Jira [[{ticket.key}]] -- Asignado: {summary} ({YYYY-MM-DD})
```

**Status change:**
```markdown
- Jira [[{ticket.key}]] -- Status changed to "{new_status}": {summary} ({YYYY-MM-DD})
```

If inbox file does not exist, create it with a `# Inbox` header before appending.

### 6. Enrich with wikilinks

For ticket notes and inbox items:
- Match assignee/reporter names against filenames in `VAULT/{config.structure.people}/` -- if a matching file exists, use `[[Person Name]]` wikilink in the note body
- Match project context against `VAULT/{config.structure.projects}/` -- add project wikilink if a matching project note exists

### 7. Graceful degradation

| Problem | Behavior |
|---------|----------|
| Jira MCP server unavailable | Skip that project, warn, continue with other projects |
| JQL returns error | Log error with JQL text, skip project, continue |
| JQL returns empty results | Log "No se encontraron tickets para {project_key}", continue |
| Single ticket fetch fails | Skip that ticket, log warning, continue with others |
| `{config.structure.jira_notes}/` folder missing | Create it |
| `connectors.yaml` missing jira section | Warn user to configure, skip Jira sync entirely |
| No `assignee_filter` and no `config.daily.assignee_name` | Sync all tickets regardless of assignee, warn user |
| Inbox file missing | Create it with `# Inbox` header, then append |

## Output

- **Updated ticket notes** in `VAULT/{config.structure.jira_notes}/` -- one `.md` file per ticket
- **Optional inbox items** in `VAULT/{config.structure.inbox}` -- only for new assignments and action-requiring status changes

## Important Rules

- Never abort on partial failure -- process as many projects/tickets as possible
- Always update `last_sync` in frontmatter on every sync (enables staleness detection)
- Wikilinks in the Relationships section enable Obsidian graph connectivity
- The skill reads Jira data via MCP tools, never via HTTP API directly
- Spanish language for all user-facing output (warnings, summaries, log messages)
- Re-sync is idempotent: running the skill twice produces the same vault state (overwrite + dedup)
