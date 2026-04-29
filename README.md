# life-os

Personal productivity plugin for Claude Code. Manages your Obsidian vault: daily and weekly notes, goal tracking, meeting prep, inbox processing, and more.

## Installation

Add the marketplace, then install the plugins you need:

```bash
/plugin marketplace add github:aldorea/life-os
/plugin install rituals@life-os
/plugin install sync@life-os
/plugin install capture@life-os
/plugin install planning@life-os
/plugin install training@life-os   # optional
/plugin install content@life-os    # optional
```

## Configuration

```bash
mkdir -p ~/.config/life-os
cp config.example.yaml ~/.config/life-os/config.yaml
# Edit ~/.config/life-os/config.yaml with your vault path and preferences
```

## Skills

| Skill | Description |
|-------|-------------|
| `/life-os:today` | Generate or update daily note |
| `/life-os:week` | Generate or update weekly note |
| `/life-os:status` | Dashboard overview of goals, tasks, contacts |
| `/life-os:prep` | Pre-meeting briefing |
| `/life-os:shop` | Weekly shopping list from menus |
| `/life-os:train` | Query training history and PRs |
| `/life-os:sync-training` | Import training data from CSV |
| `/life-os:sync-calendar` | Sync Apple Calendar to vault |
| `/life-os:sync-slack` | Extract actions from Slack channels |
| `/life-os:process-inbox` | Process inbox items to backlog |

## License

MIT
