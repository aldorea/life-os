---
project_name: life-os
user_name: Sito
date: 2026-05-19
sections_completed: ['technology_stack', 'critical_rules', 'conventions', 'workflow', 'pitfalls']
existing_patterns_found: 14
regenerated_after: '2026-05-19 cleanup (removed .planning/, STRATEGY.md, .compound-engineering/)'
---

# Project Context for AI Agents — Life OS

_Critical rules and patterns AI agents must follow when working in this repository. Focus on unobvious details that LLMs typically miss._

---

## What This Project Is (and Is Not)

**Life OS is a Claude Code plugin marketplace**, not a web/mobile application.

- The repo ships **markdown-based skills** that run inside the Claude Code CLI against an external Obsidian vault.
- There is **no `package.json`, no `tsconfig.json`, no Next.js, no React** at runtime. Any old discussion of a "web dashboard" was explicitly dropped — recoverable only from git history now.
- The **vault IS the database**. There is no SQL, no ORM, no migrations.

If a task seems to assume a JS/TS web app, stop and re-confirm with the user — the assumption is almost certainly wrong.

---

## Technology Stack & Versions

| Layer | Tech | Notes |
|-------|------|-------|
| Skill runtime | Claude Code CLI | Skills are markdown files in `<plugin>/skills/<name>/SKILL.md` |
| Data store | Obsidian vault (iCloud) | Markdown-first. No DB, no API, no schema migrations |
| Vault I/O | `obsidian` CLI | Routes through Obsidian's API — handles sync, plugins, wikilinks |
| Scripting | Bash + small Python helpers | E.g. `_bmad/scripts/resolve_customization.py` |
| Hooks | Shell scripts in `<plugin>/hooks/` | `SessionStart` (only `rituals/hooks/hooks.json` exists) |
| External data | MCP servers | Jira (×2 instances: `jira-afianza`, `jira-previene`), Slack, Granola, plus Telegram via HTTP, `icalbuddy` for calendar |
| Planning system | BMad Method v6.7.1 | **Sole planning layer.** Legacy GSD (`.planning/`) and Compound Engineering removed 2026-05-19 |

**Config locations:**

- Runtime config: `~/.config/life-os/config.yaml` (template in `config.example.yaml`)
- Secrets: `~/.config/life-os/secrets.yaml` (600 perms — Telegram, etc.)
- Connectors: `{vault}/config/connectors.yaml` (template in `connectors.example.yaml`)
- Goals: `{vault}/config/goals.yaml`
- Identity layer: `{vault}/08 Resources/CRITICAL_FACTS.md` (loaded by rituals)

---

## Repository Layout (current)

```
life-os/
├── .claude-plugin/marketplace.json   # Marketplace manifest — 5 plugins
├── .claude/                          # Local Claude Code settings + skills cache
├── .worktrees/                       # Git worktrees (gitignored)
├── rituals/  skills/{morning,today,close}      hooks/hooks.json
├── sync/     skills/{sync, sync-calendar, sync-granola, sync-jira, sync-slack, sync-telegram, sync-training[DEPRECATED]}
├── wiki/     skills/{ingest, query, synthesize, approve, lint, digest, capture}
├── training/ skills/{train, sync-training}
├── content/  skills/{content}
├── _bmad/                            # BMad install (read-only — installer regenerates)
├── _bmad-output/                     # BMad artifact output (this file)
├── docs/                             # Generated project documentation (index.md is entry point)
├── CLAUDE.md                         # ★ Authoritative project instructions for Claude
├── README.md                         # User-facing install + skill catalogue
├── LICENSE
├── config.example.yaml               # Runtime config template
└── connectors.example.yaml           # Connector config template
```

**Removed 2026-05-19** (do not look for them; reconstruct from git history if needed):
- `.planning/` (legacy GSD planning — phases 1-3 history)
- `STRATEGY.md` (product strategy)
- `.compound-engineering/` (CE artifacts)
- `docs/ideation/`, `docs/superpowers/` (subfolder content)

Each plugin is **independent** and installable separately via `/plugin install <name>@life-os`.

---

## Critical Implementation Rules

### 1. Vault I/O — always via the `obsidian` CLI

NEVER use `Read`/`Write` directly against the iCloud vault path. Always go through:

```bash
obsidian vault="Obsidian Vault" read path="03 Daily/04-05-2026.md"
obsidian vault="Obsidian Vault" create path="..." content="..." overwrite
obsidian vault="Obsidian Vault" append path="02 Inbox.md" content="..."
obsidian vault="Obsidian Vault" property:set name=status value=done path="..."
obsidian vault="Obsidian Vault" daily:read
obsidian vault="Obsidian Vault" daily:append content="..."
obsidian vault="Obsidian Vault" tasks todo path="01 Backlog.md"
obsidian vault="Obsidian Vault" task toggle path="..." line=N
```

Paths are vault-relative. Quote spaces. Use `\n` for newlines in `content`.

**Exception:** Large multi-line templates (full daily note, full Jira ticket) may use `Write` for initial creation, but reads, appends, and property updates always go through the CLI.

### 2. Skill frontmatter — `description:` controls invocability

Removing `description:` from a skill's frontmatter makes it **non-user-invocable** (internal-only, callable by other skills). This is the deliberate pattern for the sub-skills under `/sync` and `/morning`. Decision dating from the Phase 03 era (visible in git history).

### 3. Wiki contract — every wiki write logs

The wiki (`08 Resources/wiki/`) is the synthesized knowledge layer. Rules:

- **One concept per page.** Split when in doubt.
- **Frontmatter mandatory:** `type` (concept / entity / comparison / synthesis), `status` (`published` default; `draft` lives in `.drafts/`), `confidence` (low/medium/high), `updated`, `sources`, `tags`.
- **Cross-link aggressively** with `[[wikilink]]` syntax.
- **Page language matches source language; index entries always English.**
- **Bump `updated` on every edit.**
- **Append a one-line entry to `08 Resources/wiki/log.md` for any wiki write** (ingest / lint fix / page edit / synth).
- **Drafts (`.drafts/`) are not citable** and not in the index.
- Schema is authoritative in `08 Resources/wiki/WIKI.md` (in vault) — read it before editing the wiki.

### 4. Meeting-note ingestion is source-agnostic

Don't hardcode "Granola" anywhere. The abstraction must accept any meeting-notes source. Current implementation uses Granola, but the boundary is intentional.

### 5. Jira — two MCP instances

There are **two independent Jira instances**:
- **Afianza** via `jira-afianza` MCP
- **Previene** via `jira-previene` MCP

Skills must accept project config dynamically from `{vault}/config/connectors.yaml → jira.projects[]`; never hardcode project keys.

### 6. Calendar is read-only

Three calendars (Apple personal, Gmail Orbitant, Outlook Afianza). The system reads them via `icalbuddy`. Time-blocking is **manual in Outlook** — no automated calendar writes.

### 7. Rituals orchestrate; atomic skills are internal

Top-level user-invocable skills: `/morning`, `/close`, `/today`, `/sync`, `/wiki:*`, `/train`, `/content`. Most `sync-*` skills are **internal**, called by `/sync`.

When adding a new atomic capability, decide deliberately: top-level user skill or internal sub-routine of a ritual? Default to internal unless the user explicitly invokes it.

### 8. Configuration is path-stable

User config at `~/.config/life-os/config.yaml` is referenced by many skills via the same convention. Do not change the path without sweeping all skills.

### 9. `08 Resources/CRITICAL_FACTS.md` is operational config

It is the **identity layer** loaded by rituals (role, timezone, managers, training target). It is **not** wiki content — never move it into `wiki/`.

### 10. Knowledge capture flow

- Insight isolated → `type: concept` page in wiki.
- Topic that grows over time → `type: synthesis` page with `## Timeline`.
- Raw idea/link with no synthesis yet → `wiki:capture` → routes through `wiki:ingest`.
- Action/task → Inbox or Backlog (never the wiki).
- Meeting/daily/journal → `07 Meetings/` / `03 Daily/`.
- Always **ask before saving** — never auto-persist.

### 11. Failure isolation in sync

Each connector runs independently. If Jira fails, calendar still syncs. `/sync` returns per-connector status (✅ / ❌ / —). Don't add cross-connector dependencies.

---

## Conventions

### Git & commits

- **Conventional commits with scope:** `feat(wiki): …`, `chore(marketplace): …`, `fix(rituals): …`.
- **Feature branches:** `feat/<name>`, `fix/<name>`. Never commit directly to `main`.
- **GitHub via `gh` CLI** for PRs, issues, branches.

### Code style

- All code, variables, comments **in English**.
- DRY. Secure (OWASP top 10).
- Skill `SKILL.md` body language: match the user's preferred language (Spanish for this user) for user-facing prose; English for code blocks and headers.

### Communication

- Conversation with user: **Spanish** (per `_bmad/bmm/config.yaml → communication_language`).
- Document output: **English** (per `document_output_language`).

### Package manager

Not applicable — no Node/JS runtime in the marketplace. Python only in `_bmad/scripts/`.

---

## Workflow Rules

### Sprint / planning

- **BMad Method is the only planning system.** Use `/bmad-*` skills. Artifacts go to `_bmad-output/`.
- The legacy `.planning/` GSD folder has been deleted — don't try to read from it.
- For non-trivial scope, prefer `/ce-plan` (Compound Engineering parallel research) per global CLAUDE.md — note: those CE skills come from the global plugin, not from this repo.

### Skill development

- New skills go into the **right plugin** (`rituals/`, `sync/`, `wiki/`, `training/`, `content/`), not at repo root.
- Update the plugin's `.claude-plugin/plugin.json` if needed.
- For internal-only sub-skills: omit `description:` from frontmatter.

### Testing skills

- No formal test suite. Validation is **usage-based** — the user runs the skill and reports back.
- **Usage gate** (historical decision): validate each skill change before building the next.

---

## Common Pitfalls (LLM-specific reminders)

1. **Assuming there's a Next.js / web app.** There isn't. Repo is a Claude Code plugin marketplace + Obsidian vault.
2. **Writing to the vault via `fs` / `Write` against iCloud paths.** Use the `obsidian` CLI.
3. **Hardcoding Jira project keys or instance URLs.** Two instances; config-driven from `connectors.yaml`.
4. **Skipping the wiki log append.** Every wiki write must append a line to `wiki/log.md`.
5. **Creating duplicate wiki pages.** Always check `08 Resources/wiki/index.md` first.
6. **Adding a `description:` to a sub-skill** that should be internal-only.
7. **Auto-saving captures without asking.** Always confirm with user before persisting.
8. **Looking for `.planning/`, `STRATEGY.md`, or `.compound-engineering/`** — these were removed 2026-05-19. If you need historical context, use `git log` / `git show`.
9. **Coupling meeting ingestion to a specific provider name** (e.g. Granola). Keep source-agnostic.
10. **Editing `_bmad/` files directly.** The installer regenerates them. Use `_bmad/custom/` overrides instead.
11. **Treating `docs/` as authoritative source.** It is generated documentation, not the source of truth — `CLAUDE.md` and the actual skill files are.

---

## Reference Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Authoritative project instructions for Claude |
| `README.md` | User-facing install + skill catalogue |
| `docs/index.md` | Generated documentation entry point |
| `docs/architecture.md` | System design with diagrams |
| `docs/plugin-inventory.md` | Catalogue of every skill |
| `docs/integration-architecture.md` | Plugin ↔ vault ↔ MCP ↔ external data flows |
| `08 Resources/wiki/WIKI.md` (in vault) | Wiki schema — read before wiki edits |
| `08 Resources/CRITICAL_FACTS.md` (in vault) | Identity layer for rituals |
| `_bmad/bmm/config.yaml` | BMad module config (user_name, languages, paths) |
| `config.example.yaml` | Runtime config template |
| `connectors.example.yaml` | Connector config template |

---

_Regenerated: 2026-05-19 by `bmad-generate-project-context` after repo cleanup (Sito)._
