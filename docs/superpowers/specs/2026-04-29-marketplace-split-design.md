# Design: life-os marketplace multi-plugin split

**Date:** 2026-04-29
**Status:** Approved

## Problem

The current `life-os` plugin is a monolith with 16+ skills spanning GTD rituals, sync connectors, capture workflows, planning tools, training tracking, and content generation. Maintenance is harder as unrelated domains grow together.

## Goal

Reorganize `life-os` into a marketplace of 6 focused plugins within the same repo, keeping a single `git clone` and a single install command per plugin.

## Plugin Structure

```
life-os/ (repo)
├── .claude-plugin/
│   └── marketplace.json        ← lists all 6 plugins
├── rituals/                    → plugin "rituals"
│   ├── .claude-plugin/plugin.json
│   ├── hooks/hooks.json        ← SessionStart hook
│   └── skills/
│       ├── morning/
│       ├── today/
│       ├── close/
│       └── week/
├── sync/                       → plugin "sync"
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── sync/
│       ├── sync-calendar/
│       ├── sync-granola/
│       ├── sync-jira/
│       ├── sync-slack/
│       └── sync-telegram/
├── capture/                    → plugin "capture"
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── dump/
│       ├── capture/
│       ├── process-inbox/
│       └── clarify/
├── planning/                   → plugin "planning"
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── prep/
│       ├── status/
│       ├── review-backlog/
│       └── review/
├── training/                   → plugin "training"
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── train/
│       └── sync-training/
├── content/                    → plugin "content"
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── content/
│       └── shop/
├── config.example.yaml         ← repo root, documentation only
├── connectors.example.yaml
└── README.md
```

## Config Sharing

**Problem:** `${CLAUDE_PLUGIN_ROOT}` resolves to each plugin's own cache directory. Multiple plugins cannot share a single `config.yaml` via this variable.

**Solution:** Move config to a fixed user-level path:

```
~/.config/life-os/config.yaml     ← main config (was ${CLAUDE_PLUGIN_ROOT}/config.yaml)
~/.config/life-os/secrets.yaml    ← already exists
```

All skills replace `${CLAUDE_PLUGIN_ROOT}/config.yaml` with `~/.config/life-os/config.yaml`.

The `config.example.yaml` stays at the repo root as a reference template. Setup instructions update accordingly.

## Cross-Plugin Dependencies

Some plugins call skills from other plugins:

| Caller | Depends on |
|--------|-----------|
| `rituals:morning` | `sync:sync`, `capture:process-inbox`, `rituals:today` |
| `rituals:close` | `sync:sync-telegram` |
| `sync:sync` | `sync:sync-calendar`, `sync:sync-granola`, `sync:sync-jira`, `sync:sync-slack`, `sync:sync-telegram`, `training:sync-training` |

These work transparently as long as the required plugins are installed — Claude invokes skills by name regardless of which plugin they belong to. Dependencies are documented in each plugin's README.

## Resulting Skill Invocation

| Plugin | Skills |
|--------|--------|
| `rituals` | `rituals:morning`, `rituals:today`, `rituals:close`, `rituals:week` |
| `sync` | `sync:sync`, `sync:sync-calendar`, `sync:sync-granola`, `sync:sync-jira`, `sync:sync-slack`, `sync:sync-telegram` |
| `capture` | `capture:dump`, `capture:capture`, `capture:process-inbox`, `capture:clarify` |
| `planning` | `planning:prep`, `planning:status`, `planning:review-backlog`, `planning:review` |
| `training` | `training:train`, `training:sync-training` |
| `content` | `content:content`, `content:shop` |

## Marketplace manifest

The root `marketplace.json` lists all 6 plugins with local sources:

```json
{
  "name": "life-os",
  "owner": { "name": "aldorea", "url": "https://github.com/aldorea" },
  "plugins": [
    { "name": "rituals",  "source": "./rituals",  "version": "0.1.0", "description": "Daily and weekly rituals: morning, today, close, week." },
    { "name": "sync",     "source": "./sync",     "version": "0.1.0", "description": "Sync connectors: calendar, Granola, Jira, Slack, Telegram." },
    { "name": "capture",  "source": "./capture",  "version": "0.1.0", "description": "Capture and inbox processing: dump, capture, process-inbox, clarify." },
    { "name": "planning", "source": "./planning", "version": "0.1.0", "description": "Planning tools: prep, status, review-backlog, review." },
    { "name": "training", "source": "./training", "version": "0.1.0", "description": "Training tracking and import: train, sync-training." },
    { "name": "content",  "source": "./content",  "version": "0.1.0", "description": "Content generation and shopping list: content, shop." }
  ]
}
```

## Migration Steps (high level)

1. Create 6 plugin directories with `plugin.json` manifests
2. Move skills to their respective plugin directories
3. Move `hooks/hooks.json` to `rituals/hooks/hooks.json`
4. Update `marketplace.json` at repo root
5. Update all skill `Step 0` config path: `${CLAUDE_PLUGIN_ROOT}/config.yaml` → `~/.config/life-os/config.yaml`
6. Update `config.example.yaml` setup instructions
7. Update root `README.md` with new install commands per plugin
8. Remove old top-level `plugin.json` (repo becomes pure marketplace)
9. Test each plugin independently

## Out of Scope

- No changes to skill logic or behavior
- No changes to vault structure
- No changes to `connectors.yaml` or `secrets.yaml` paths
