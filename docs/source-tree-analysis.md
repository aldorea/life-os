# Source Tree Analysis — Life OS

> Annotated layout of the repository. Quick Scan — based on directory structure and file patterns, not exhaustive file reads.

## Top-level

```
life-os/
├── .claude-plugin/
│   └── marketplace.json            # Marketplace manifest — declares 5 plugins
├── .claude/                        # Local Claude Code config (skills cache, settings.local.json)
├── .worktrees/                     # Git worktrees (gitignored)
# NOTE: .compound-engineering/ and .planning/ were removed 2026-05-19
# Project history is now reconstructable only from git log.
├── _bmad/                          # BMad Method install (installer-managed; do NOT edit)
│   ├── _config/bmad-help.csv       # Skill catalog assembled at install
│   ├── bmm/                        # BMad Method module
│   ├── core/                       # BMad core module
│   ├── custom/                     # YOUR overrides go here (config.toml, config.user.toml)
│   ├── scripts/                    # Python helpers (resolve_customization.py, etc.)
│   ├── config.toml                 # Generated — do not edit
│   └── config.user.toml            # Personal install answers
├── _bmad-output/                   # BMad-generated artifacts (project-context.md, etc.)
├── content/                        # PLUGIN: content generation
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       └── content/SKILL.md        # /content:content
├── docs/                           # ← THIS folder (project_knowledge for BMad)
#   (ideation/ and superpowers/ subfolders were removed 2026-05-19)
├── rituals/                        # PLUGIN: rituals
│   ├── .claude-plugin/plugin.json
│   ├── hooks/hooks.json            # SessionStart hook → /rituals:morning
│   └── skills/
│       ├── morning/SKILL.md        # /rituals:morning  (user-facing)
│       ├── today/SKILL.md          # /rituals:today    (user-facing)
│       └── close/SKILL.md          # /rituals:close    (user-facing)
├── sync/                           # PLUGIN: data ingestion
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── sync/SKILL.md            # /sync:sync         (aggregator — user-facing)
│       ├── sync-calendar/SKILL.md   # internal (no description:)
│       ├── sync-granola/SKILL.md    # internal
│       ├── sync-jira/SKILL.md       # internal
│       ├── sync-slack/SKILL.md      # internal
│       ├── sync-telegram/SKILL.md   # internal
│       └── sync-training/SKILL.md   # DEPRECATED 2026-05-07 (see frontmatter)
├── training/                       # PLUGIN: training
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── train/SKILL.md          # /training:train       (user-facing)
│       └── sync-training/SKILL.md  # /training:sync-training
├── wiki/                           # PLUGIN: personal LLM-friendly wiki
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── ingest/SKILL.md         # /wiki:ingest    (user-facing)
│       ├── query/SKILL.md          # /wiki:query     (user-facing)
│       ├── synthesize/SKILL.md     # /wiki:synthesize
│       ├── approve/SKILL.md        # /wiki:approve
│       ├── lint/SKILL.md           # /wiki:lint
│       ├── digest/SKILL.md         # /wiki:digest
│       └── capture/SKILL.md        # /wiki:capture
├── CLAUDE.md                       # ★ AUTHORITATIVE behavior contract for Claude
├── README.md                       # User-facing install + skill catalogue
# NOTE: STRATEGY.md was removed 2026-05-19
├── LICENSE                         # MIT
├── config.example.yaml             # Template for ~/.config/life-os/config.yaml
├── connectors.example.yaml         # Template for {vault}/config/connectors.yaml
└── .gitignore
```

## Critical Directories

| Directory | Why it's critical | Read first when... |
|-----------|-------------------|--------------------|
| `rituals/skills/morning/` | Daily entry point; orchestrates sync + inbox + daily note | Touching any ritual logic |
| `sync/skills/sync/` | Single aggregator that drives all connectors | Adding a new connector |
| `sync/skills/sync-jira/` | Multi-instance Jira handling — non-trivial config logic | Adding another Jira org |
| `wiki/skills/ingest/` | Knowledge entry point; routes captures to wiki vs `.drafts/` | Modifying wiki contract |
| `_bmad/custom/` | Where to put BMad overrides (other `_bmad/` files are installer-managed) | Customizing BMad behavior |

## Entry Points

| Trigger | Entry point | What it does |
|---------|-------------|--------------|
| `SessionStart` (hook) | Echoes a reminder to run `/rituals:morning` | rituals/hooks/hooks.json |
| User types `/rituals:morning` | `rituals/skills/morning/SKILL.md` | Full daily preparation |
| User types `/sync` | `sync/skills/sync/SKILL.md` | All connectors |
| User types `/wiki:*` | Corresponding `wiki/skills/<name>/SKILL.md` | Wiki operations |
| User types `/bmad-*` | Skills under `.claude/skills/` | BMad workflows |

## File Count by Plugin

| Plugin | User-invocable skills | Internal skills | Hooks |
|--------|-----------------------|-----------------|-------|
| `rituals` | 3 (morning, today, close) | 0 | 1 (SessionStart) |
| `sync` | 1 (sync) | 6 (sync-calendar, sync-granola, sync-jira, sync-slack, sync-telegram, sync-training-DEPRECATED) | 0 |
| `wiki` | 7 (ingest, query, synthesize, approve, lint, digest, capture) | 0 | 0 |
| `training` | 2 (train, sync-training) | 0 | 0 |
| `content` | 1 (content) | 0 | 0 |

## Important Files Outside Plugins

| File | Role |
|------|------|
| `CLAUDE.md` | Behavior contract — read at every session start. Defines vault I/O, wiki contract, knowledge capture flow. |
| `_bmad/bmm/config.yaml` | BMad module config (user_name, languages, artifact paths). |
| `_bmad-output/project-context.md` | Just-generated AI agent rules. |
| `.gitignore` | Ignores `.worktrees/` and other transient paths. |

## What's NOT Here (Common Misconceptions)

- ❌ No `package.json`, `tsconfig.json`, `next.config.js` — there is no JS/TS app.
- ❌ No `Dockerfile`, no Kubernetes manifests — no deployment infra.
- ❌ No `requirements.txt` / `pyproject.toml` — Python is only used in `_bmad/scripts/`.
- ❌ No GitHub Actions workflow files — no CI pipeline.
- ❌ No `tests/` directory — validation is usage-based.
- ❌ No `migrations/` — vault has no schema.
