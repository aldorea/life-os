---
name: sync
description: Runs all configured sync connectors and reports results. Use when user says "sync", "sincroniza", "sincronizar todo", "sync all".
---

# sync

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Unified sync orchestrator. Runs all configured connectors sequentially in fixed order, collects results, and displays a compact status report. Each connector is independent -- if one fails, the rest continue. Non-interactive -- never prompts the user.

## Process

### Step 1 -- Read connector configuration

Read two config sources to determine which connectors are configured:

| Connector | Config Source | Check |
|-----------|-------------|-------|
| Calendar | `${CLAUDE_PLUGIN_ROOT}/config.yaml` | `config.calendar.tool` is NOT `"none"` and NOT empty |
| Jira | `VAULT/{config.structure.connectors_config}` (connectors.yaml) | `jira.projects` array has at least one entry |
| Slack | `VAULT/{config.structure.connectors_config}` (connectors.yaml) | `slack.channels` array has at least one entry |
| Reminders | `VAULT/{config.structure.connectors_config}` (connectors.yaml) | `reminders.lists` array has at least one entry |
| Granola | MCP tool availability | `query_granola_meetings` tool is available (NOT a YAML config check) |
| Training | `${CLAUDE_PLUGIN_ROOT}/config.yaml` | `config.training.import_source` exists and is not empty |

If `connectors.yaml` does not exist at `VAULT/{config.structure.connectors_config}`, log a warning and mark Jira, Slack, and Reminders as skipped.

Initialize a results list to collect `{connector, status, message}` for each of the 6 connectors.

### Step 2 -- Execute connectors (sequential, fixed order)

Execute each connector in the exact order below. Each connector is wrapped in independent error handling -- if one fails, log the error as the result message and continue to the next.

#### 2.1 Calendar

Syncs calendar events via system tool (e.g. icalBuddy) and writes a cache file to the vault.

- **Config check:** `config.calendar.tool` is NOT `"none"` and NOT empty
- **If not configured:** result = `{status: "skipped", message: "no configurado"}`
- **If configured:** Execute the sync logic defined in `skills/sync-calendar/SKILL.md`
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

#### 2.2 Jira

Syncs Jira tickets from configured projects via MCP into individual vault notes and creates inbox items for new assignments.

- **Config check:** `connectors.yaml` has `jira.projects` with at least one entry
- **If not configured:** result = `{status: "skipped", message: "no configurado"}`
- **If configured:** Execute the sync logic defined in `skills/sync-jira/SKILL.md`
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

#### 2.3 Slack

Reads Slack channels via MCP and extracts actionable items (decisions, action items, questions) into the vault inbox.

- **Config check:** `connectors.yaml` has `slack.channels` with at least one entry
- **If not configured:** result = `{status: "skipped", message: "no configurado"}`
- **If configured:** Execute the sync logic defined in `skills/sync-slack/SKILL.md`
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

#### 2.4 Reminders

Reads Apple Reminders via osascript and adds unprocessed items to the vault inbox.

- **Config check:** `connectors.yaml` has `reminders.lists` with at least one entry
- **If not configured:** result = `{status: "skipped", message: "no configurado"}`
- **If configured:** Execute the sync logic defined in `skills/sync-reminders/SKILL.md`
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

#### 2.5 Granola

Syncs Granola meeting notes into the vault, enriches People notes, and extracts action items to the Backlog.

- **Config check:** Check if MCP tool `query_granola_meetings` is available. This is NOT a YAML config check -- Granola relies on MCP availability, not connectors.yaml.
- **If not available:** result = `{status: "skipped", message: "MCP no disponible"}`
- **If available:** Execute the sync logic defined in `skills/sync-granola/SKILL.md`
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

#### 2.6 Training

Imports training data from a fitness app CSV export into the vault.

- **Config check:** `config.training.import_source` exists and is not empty
- **If not configured:** result = `{status: "skipped", message: "no configurado"}`
- **If configured:** Execute the sync logic defined in `skills/sync-training/SKILL.md`
- **IMPORTANT:** If no CSV file is found automatically in the configured search paths, result = `{status: "skipped", message: "no hay CSV nuevo"}`. Do NOT prompt the user for a file path -- /sync is non-interactive.
- **Collect result:** `{status: "success"|"error"|"skipped", message: "..."}`

### Step 3 -- Format and display report

After all connectors have executed, display the report:

```
Sync completado:

{icon} Calendar   -- {message}
{icon} Jira       -- {message}
{icon} Slack      -- {message}
{icon} Reminders  -- {message}
{icon} Granola    -- {message}
{icon} Training   -- {message}

{error_summary_line}
```

**Icon rules:**
- `✓` for status = `"success"`
- `✗` for status = `"error"`
- `─` for status = `"skipped"`

**Error summary line:** Only show if at least one connector has status `"error"`. Format:

```
{count} error(es). Revisa {comma-separated names of failed connectors}.
```

If zero errors, do NOT show the error summary line.

## Graceful Degradation

| Problem | Behavior |
|---------|----------|
| config.yaml missing | Stop with message: copy config.example.yaml to config.yaml |
| connectors.yaml missing | Mark Jira, Slack, Reminders as skipped; continue with Calendar, Granola, Training |
| Calendar tool not installed | Calendar result = error with message from sync-calendar degradation |
| Jira MCP unavailable | Jira result = error with MCP error message |
| Slack MCP unavailable | Slack result = error with MCP error message |
| Reminders permission denied | Reminders result = error with permission message |
| Granola MCP unavailable | Granola result = skipped (MCP no disponible) |
| Training CSV not found | Training result = skipped (no hay CSV nuevo) |
| ALL connectors fail or skip | Still display the full report showing all failures/skips |

## Important Rules

- Each connector is independent -- failure in one NEVER blocks the next
- Non-interactive -- never prompt the user for input during /sync
- Spanish language for all user-facing output
- Git safety is inherited from each connector's SKILL.md -- do NOT add a separate git layer in /sync (each sync-* skill already handles its own git commits)
- If ALL connectors fail or skip, still display the report showing all failures/skips
- Do NOT duplicate full connector logic here -- reference the `skills/sync-*/SKILL.md` files as the source of truth for each connector's implementation
