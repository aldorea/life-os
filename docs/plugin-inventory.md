# Plugin Inventory — Life OS

> Catalogue of every plugin and skill in the marketplace.
> User-invocable = has `description:` in frontmatter. Internal = orchestrated by another skill.

## `rituals` — Daily and weekly rituals

| Skill | Slash | Type | Purpose |
|-------|-------|------|---------|
| `morning` | `/rituals:morning` | User | Orchestrator: sync + inbox cleanup + daily note. Reads `CRITICAL_FACTS.md`, then delegates to `/sync`. |
| `today` | `/rituals:today` | User | Generate or update the daily note. |
| `close` | `/rituals:close` | User | End-of-day reflection, completed-tasks log, wiki log append. |

**Hook:** `SessionStart` reminds Claude to run `/rituals:morning` at session start unless already done today.

## `sync` — Data ingestion connectors

| Skill | Slash | Type | Purpose |
|-------|-------|------|---------|
| `sync` | `/sync` | User | Aggregator. Reads `connectors.yaml`, runs each enabled connector in sequence. Independent failure — if one fails, others continue. |
| `sync-calendar` | — | Internal | Apple Calendar + Gmail + Outlook via `icalbuddy`. Writes to `.cache/calendar-cache.md`. |
| `sync-granola` | — | Internal | Granola meeting notes → `07 Meetings/`. Source-agnostic abstraction. |
| `sync-jira` | — | Internal | Two Jira MCP instances (`jira-afianza`, `jira-previene`). Per-instance assignee filter and status filter. |
| `sync-slack` | — | Internal | Slack channels per `connectors.yaml`. Extracts actions/decisions. |
| `sync-telegram` | — | Internal | HTTP API to Fly.io-hosted Telegram bot. Reads token from `secrets.yaml`. |
| `sync-training` | — | DEPRECATED | Replaced by Telegram-based training capture (2026-05-07). Kept as archive. |

## `wiki` — Personal LLM Knowledge Base

Schema authoritative in vault `08 Resources/wiki/WIKI.md`. Every wiki write must append a line to `wiki/log.md`.

| Skill | Slash | Type | Purpose |
|-------|-------|------|---------|
| `ingest` | `/wiki:ingest <source>` | User | Process URL / YouTube / PDF / inline text. High/medium confidence → `pages/`. Low confidence → `.drafts/`. |
| `query` | `/wiki:query <question>` | User | Search pages and synthesize an **ephemeral** answer with citations. Suggests `/wiki:synthesize` when result worth persisting. |
| `synthesize` | `/wiki:synthesize [topic]` | User | Promote a query answer (or consolidate multiple pages) into a persistent `type: synthesis` page. |
| `approve` | `/wiki:approve [name]` | User | Review pending drafts. List, show, approve / edit / discard. |
| `lint` | `/wiki:lint` | User | Health check: orphans, dead wikilinks, missing cross-refs, stale pages, contradictions, stale drafts. |
| `digest` | `/wiki:digest [day\|week\|month]` | User | Activity summary from `log.md` + pending drafts snapshot. Defaults to `week`. |
| `capture` | `/wiki:capture` | User | Capture knowledge/links/observations. Routes through the same pipeline as URL inbox messages — delegates to `/wiki:ingest`. |

## `training` — Training tracking

| Skill | Slash | Type | Purpose |
|-------|-------|------|---------|
| `train` | `/training:train` | User | Query training history, PRs, progress. |
| `sync-training` | `/training:sync-training` | User | Import training data from CSV (Heavy/Strong format). |

## `content` — Content generation and nutrition

| Skill | Slash | Type | Purpose |
|-------|-------|------|---------|
| `content` | `/content:content` | User | Generate LinkedIn posts, blog articles, or talks from the wiki. Also covers nutrition (shopping list from menus). |

## Cross-plugin Patterns

- **All user-invocable skills load `~/.config/life-os/config.yaml` first** and stop with a friendly error if missing.
- **All vault writes go through the `obsidian` CLI** with `vault="Obsidian Vault"`.
- **Date format for daily notes**: `DD-MM-YYYY` (configurable in `config.daily.date_format`).
- **All skills speak Spanish to the user, English in code/docs** (per global preferences).

## Discovery Hints for LLMs

If you don't recognize a slash command:

1. Check `.claude-plugin/marketplace.json` for plugins.
2. For each plugin, list `skills/` to find skill folders.
3. Read `SKILL.md` frontmatter — if `description:` exists, it's user-callable.

If unsure where a behavior lives:

- "Capture / save link / save idea" → `wiki:capture` or `wiki:ingest`.
- "Daily note / how was today" → `rituals:today` or `rituals:close`.
- "Sync X" → look in `sync/skills/sync-X/`, called via `/sync`.
- "Generate content / draft post" → `content`.
- "What did I lift / training PR" → `training:train`.
