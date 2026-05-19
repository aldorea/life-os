# Project Overview — Life OS

## Purpose

Life OS is a **personal operating system** that centralizes task management, knowledge capture, meeting context, learning, training, and content generation for a professional managing multiple work streams (multi-Jira, Slack, Granola meetings, training, personal projects) on top of an existing Obsidian vault.

It is delivered as a **Claude Code plugin marketplace**: each capability is an installable plugin composed of skills (markdown files) that operate against the user's vault.

## Core Value

> "Ayudar a Sito a decidir qué hacer cada semana, desglosarlo, organizarlo y ejecutarlo — resolviendo la fricción entre 'tengo muchas cosas' y 'no sé por dónde empezar'."

Three-pillar approach (reconstructed from git history; the canonical `STRATEGY.md` was deleted on 2026-05-19):

1. **Aggregation** — the agent collects information from many sources (calendar, Granola, Jira, Slack, Telegram, notes) and detects contradictions.
2. **Confrontation** — the agent challenges the user when actions diverge from stated goals.
3. **Thinking partner** — the agent questions ideas instead of merely recording them.

## Target User

A professional operating across multiple fronts simultaneously — several client projects via Jira, an internal product, initiatives, health, learning, content, personal life — whose information lives scattered across many sources.

## Key Metrics (from former STRATEGY.md — file removed but metrics still apply)

- Days/week with completed morning ritual.
- % of daily focuses aligned with a monthly goal.
- Outputs/month generated from the system (articles, menus, training programs, drafts, briefings).
- Manual organization time/week (should decrease month over month).
- Thinking-partner conversations/week.

## Tracks

| Track | Purpose | Plugin(s) |
|-------|---------|-----------|
| **Sync** | Raw ingestion: agents fetch data from external sources into the vault automatically | `sync` |
| **Wiki** | Curated knowledge layer: agents synthesize captures into atomic interlinked pages | `wiki` |
| **Rituals** | Confrontation moments: morning, today, close, weekly review | `rituals` |
| **Extraction** | Use the curated graph to produce outputs (content, menus, training plans) | `content`, `training` |

## Scope

### In Scope (active)

- Sync connectors (calendar, Jira ×2, Granola, Slack, Telegram, training CSV).
- Daily/weekly rituals (morning, today, close).
- Wiki: ingest, query, synthesize, lint, digest, approve, capture.
- Training history queries.
- Content generation drafts (LinkedIn, blog, talks) from wiki.
- Shopping list from menus.

### Out of Scope (explicit, per former `.planning/PROJECT.md` v1.1 — file removed 2026-05-19)

- Web dashboard — dropped; the vault is the interface.
- CRM / people tracking (future, not current pain).
- Mobile app — Obsidian mobile is sufficient.
- Auto-posting to social media.
- Speculative skills that don't solve a real pain.

## Constraints

- **Vault-first**: all persistent data lives as markdown in the existing Obsidian vault.
- **No database**: the vault is the store. No SQL, no ORM, no migrations.
- **Read-only calendar**: time-blocking is manual in Outlook.
- **Two Jira MCP instances** (Afianza, Previene): config-driven, not hardcoded.
- **Source-agnostic meeting notes**: the abstraction must accept any provider, even though current implementation uses Granola.

## Project History (compressed)

> Reconstructed from git history. The legacy `.planning/` folder that documented phases in detail was removed 2026-05-19.

| Phase | Outcome | Reference |
|-------|---------|-----------|
| Phase 1 | Vault schema, GTD capture, processing, daily/weekly rituals, goal tracking | git history |
| Phase 2 | Sync connectors (Jira, Calendar, Granola, Slack, Reminders) | git history |
| Phase 3 | Unified sync (`/sync` aggregates everything; `/morning` delegates to it) | commits `2914309`, `e68508f`, `cc2d4f2` |
| **v1.1 reset** | 26 skills → 4 core skills focused on planning/organizing. Usage gate enforced. | git history |
| **Wiki overhaul** (2026-05) | Drop GTD scaffolding, simplify to wiki + rituals | commit `2d239e2` |
| **Marketplace split** | Top-level repo became pure marketplace; 5 plugin sub-folders | commits `297846f`-`4ee33d2` |
| **Cleanup** (2026-05-19) | Removed legacy planning (`.planning/`, `STRATEGY.md`, `.compound-engineering/`, `docs/ideation/`, `docs/superpowers/`) | this session |

## Current State (as of 2026-05-19)

- 5 plugins published.
- 19 skill files across plugins.
- Active iteration on wiki UX (`wiki:query` returns prose; raw-notes default for single-source ingests).
- BMad v6.7.1 just installed — first BMad workflow run is this documentation generation.
- GSD legacy planning and Compound Engineering artifacts removed from the repo (2026-05-19) — only BMad remains as the planning layer.
