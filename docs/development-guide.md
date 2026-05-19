# Development Guide — Life OS

## Prerequisites

| Tool | Why | Notes |
|------|-----|-------|
| Claude Code CLI | Runtime for all skills | Required |
| Obsidian | Vault application | Required |
| `obsidian` CLI | Vault I/O (skills depend on it) | Install via the Obsidian CLI plugin/extension |
| `icalbuddy` | Calendar sync | macOS only; `brew install ical-buddy` |
| `gh` CLI | All GitHub operations | Per CLAUDE.md convention |
| `git` | Version control | Standard |
| Python 3 | BMad helpers under `_bmad/scripts/` | System Python is fine |
| Node | Not used by the marketplace itself | Some MCP servers may require it |

## Installation (end user)

```bash
# 1. Add marketplace
/plugin marketplace add github:aldorea/life-os

# 2. Install plugins you want
/plugin install rituals@life-os
/plugin install sync@life-os
/plugin install wiki@life-os
/plugin install training@life-os   # optional
/plugin install content@life-os    # optional

# 3. Configure runtime
mkdir -p ~/.config/life-os
cp config.example.yaml ~/.config/life-os/config.yaml
$EDITOR ~/.config/life-os/config.yaml

# 4. Secrets (600 perms!)
touch ~/.config/life-os/secrets.yaml
chmod 600 ~/.config/life-os/secrets.yaml

# 5. Per-vault connector config
cp connectors.example.yaml "{vault}/config/connectors.yaml"
$EDITOR "{vault}/config/connectors.yaml"
```

Then in Claude Code: type `/rituals:morning` to validate everything.

## Development Setup

### Clone

```bash
git clone https://github.com/aldorea/life-os.git
cd life-os
```

### Working on a skill

1. Create a feature branch: `git checkout -b feat/<name>`.
2. Edit the relevant `<plugin>/skills/<name>/SKILL.md`.
3. Open Claude Code in this repo; the local plugin instances pick up your changes immediately.
4. Test by invoking the slash command. **Validation is usage-based** — no test suite.
5. Per Phase 03 rule: **validate each skill change before building the next one**.

### Skill anatomy

```markdown
---
name: skill-name
description: Use when ... (omit description: to make it internal-only)
---

# skill-name

## Step 0 -- Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml`. Stop.

Set `VAULT` = `{config.vault_path}` for reference only.

**Vault I/O:** Use `obsidian vault="Obsidian Vault"` CLI — never raw fs.

## Overview

What the skill does, when it runs, what it produces.

## Process

### 1. ...
### 2. ...
```

### Conventions

| Rule | Source |
|------|--------|
| Code, variables, comments in English | Global CLAUDE.md |
| Skill docs match the user's preferred language (Spanish) | Local CLAUDE.md |
| Conventional Commits with scope: `feat(wiki): ...` | Global CLAUDE.md + git log |
| Feature branches: `feat/<name>`, `fix/<name>` | Never commit to `main` |
| Use `gh` for all GitHub ops | Global CLAUDE.md |
| Vault writes via `obsidian` CLI | Project CLAUDE.md |
| Append to `wiki/log.md` on every wiki write | Project CLAUDE.md |
| Skills without `description:` are internal | Phase 03 decision |
| Ask before persisting captures | Project CLAUDE.md |

## Configuration Reference

### `~/.config/life-os/config.yaml`

See `config.example.yaml` for the full schema. Key sections:

- `vault_path` — absolute path to your Obsidian vault.
- `structure.*` — vault folder layout (daily, weekly, backlog, etc.).
- `daily.date_format` — daily note file name format (default `DD-MM-YYYY`).
- `daily.assignee_name` — your name as it appears in Jira (for filtering).
- `calendar.tool` — `icalbuddy` | `gcal-cli` | `none`.
- `tags.*` — taxonomy (contexts, projects, priority, actionability, status).
- `backlog_sections` — backlog grouping with `#tag` filters.

### `~/.config/life-os/secrets.yaml`

```yaml
telegram:
  api_key: "<your-api-key>"
```

### `{vault}/config/connectors.yaml`

```yaml
slack:
  channels: [...]
jira:
  projects:
    - mcp_server: jira-afianza
      project_key: AFNZ
      ...
    - mcp_server: jira-previene
      project_key: PREV
      ...
reminders:
  lists: [...]
```

## MCP Servers

Configure in `~/.claude.json` (global) or `.claude/settings.json` (project). The skills assume these MCP names exist:

| MCP name | Used by |
|----------|---------|
| `jira-afianza` | `sync-jira` (Afianza projects) |
| `jira-previene` | `sync-jira` (Previene projects) |
| `slack` (or whatever your Slack MCP is named — adjust in skill) | `sync-slack` |
| `granola` | `sync-granola` |

## Build / Test / Deploy

- **Build**: none. Markdown skills don't compile.
- **Test**: invoke the skill manually; verify vault output.
- **Deploy**: push to `main` (after PR). End users pull via `/plugin install`.
- **Lint**: `/wiki:lint` for the wiki layer; no linter for skill markdown itself.

## Common Development Tasks

### Add a new connector

1. Create `sync/skills/sync-<name>/SKILL.md` **without `description:`** (internal-only).
2. Implement: load config, call MCP/CLI, write to vault.
3. Add the connector to `sync/skills/sync/SKILL.md` orchestration.
4. Add the connector to `connectors.example.yaml` with `enabled: false` default.
5. Document in `README.md`.

### Add a new ritual

1. Create `rituals/skills/<name>/SKILL.md` **with `description:`**.
2. Decide if it appears in `SessionStart` hook (probably not).
3. Update `README.md`.

### Modify the wiki contract

1. Read `08 Resources/wiki/WIKI.md` first.
2. Update the schema if needed.
3. Update affected skills (`ingest`, `synthesize`, `lint`).
4. Run `/wiki:lint` to verify existing pages still pass.

### Update Claude's behavior contract

Edit `CLAUDE.md`. Treat changes carefully — this file is loaded by every session.

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| "config.yaml not found" | `~/.config/life-os/config.yaml` missing. Copy from `config.example.yaml`. |
| Jira sync returns nothing | Wrong `assignee_filter`, wrong `mcp_server` name, or MCP server not running. |
| Telegram skill fails | `secrets.yaml` missing `telegram.api_key`, or file perms not 600. |
| Wiki ingest puts page in `.drafts/` | Source confidence < high. Use `/wiki:approve` to review. |
| `obsidian` CLI fails | Vault path wrong, Obsidian not running, or CLI plugin not installed. |
| SessionStart hook missing | Hook is in `rituals/hooks/hooks.json` — make sure `rituals` plugin is installed. |

## Contributing

1. Open an issue or branch with a clear scope.
2. Follow Conventional Commits.
3. Open a PR via `gh pr create`.
4. Self-validate: the user runs the skill before merge (usage gate).
