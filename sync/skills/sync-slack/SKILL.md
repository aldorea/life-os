---
name: sync-slack
---

# sync-slack

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that reads Slack channels via MCP and extracts actionable items (decisions, action items, questions) into the Obsidian Inbox.

## Prerequisites

- Slack MCP server must be available
- If not available, warn the user and skip gracefully

## Process

### 1. Read configured channels

Read `VAULT/{config.structure.connectors_config}` for the list of channels to scan. If no config exists, ask the user which channels to monitor.

### 2. Scan channels for actionable items

Use Slack MCP tools to read each configured channel. Look for messages from the last 24 hours (or since last sync).

Extract only:
- **Decisions** — Messages that conclude a discussion or set direction
- **Action items** — Tasks explicitly assigned or requested
- **Questions pending response** — Direct questions without reply
- **Important announcements** — Team-wide updates

Ignore:
- Casual conversation, greetings, reactions
- Threads already resolved
- Messages from bots (unless critical alerts)

### 3. Enrich with wikilinks

For each extracted item:
- Match sender name against `VAULT/{config.structure.people}/` → add `[[Person Name]]`
- Match project/channel context → add project wikilink if applicable

### 4. Write to Inbox

Append new items to `VAULT/{config.structure.inbox}` with format:

```markdown
- Slack #channel — [[Persona]]: description (YYYY-MM-DD)
```

Do NOT duplicate items already in the Inbox.

### 5. Graceful degradation

- If Slack MCP unavailable: log warning, skip entirely
- If a channel is not found: skip that channel, continue with others
- If no actionable items found: do nothing (don't write empty results)

## Output

Appended items in `VAULT/{config.structure.inbox}`. No separate cache file.
