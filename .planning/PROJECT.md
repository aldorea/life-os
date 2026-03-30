# Life OS

## What This Is

A personal operating system that centralizes task management, goal tracking, knowledge capture, CRM, and content generation into a unified system. Runs as Claude Code CLI skills + Obsidian vault + web dashboard, built on GTD methodology. Designed for a professional managing multiple projects across different Jira instances who wants one place to think, plan, and execute.

## Core Value

A reliable GTD system that captures everything, centralizes all task sources, and ensures nothing falls through the cracks — the foundation everything else builds on.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Full GTD implementation (inbox, capture, processing, weekly review, contexts, next actions, projects, someday/maybe)
- [ ] Task centralization from Jira (dynamic projects), Apple Reminders, and Calendar events
- [ ] Obsidian vault as the single source of truth for all notes and knowledge
- [ ] Goal setting framework with periodic reflection (quarterly/annual) + active tracking with metrics
- [ ] Mini CRM: people notes with meeting history, context, follow-ups, and relation to goals
- [ ] Meeting notes sync (source-agnostic — Granola today, replaceable tomorrow)
- [ ] Knowledge capture from work sessions into structured vault notes
- [ ] Content generation pipeline: topic suggestions → drafts → review → scheduling (LinkedIn, blog)
- [ ] Web dashboard for status overview, goals progress, and task visibility
- [ ] Vault redesign: reorganize existing daily notes, meeting notes, and knowledge base

### Out of Scope

- Mobile app — CLI + Obsidian mobile + web dashboard is sufficient
- Real-time collaboration — this is a personal system
- Built-in meeting recording/transcription — delegate to specialized tools (Granola, etc.)
- Social media auto-posting — generate content, publish manually or via separate tool

## Context

- Already has a Claude Code plugin with ~15 skills (morning, capture, dump, review, sync-jira, sync-granola, sync-slack, sync-calendar, content, shop, train, etc.)
- MCP servers configured: Jira (Afianza + Previene), Slack, Notion, Granola, Calendar
- Existing Obsidian vault with daily notes, meeting notes, and knowledge base — needs restructuring
- User follows GTD methodology for personal productivity
- Jira projects are dynamic — need easy add/remove without code changes
- User manages multiple professional projects simultaneously

## Constraints

- **Interaction model**: CLI (Claude Code skills) + Obsidian (visualization/navigation) + Web (dashboard)
- **Vault**: Obsidian-first — all persistent data lives as markdown in the vault
- **Meeting notes**: Source-agnostic design — abstract away from any specific provider
- **Jira**: Support dynamic project configuration (not hardcoded instances)
- **Stack (web)**: To be determined by research — no strong preference

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| GTD as core methodology | User already follows GTD, natural fit for task organization | — Pending |
| Obsidian as single source of truth | Markdown-based, local-first, extensible, user already has a vault | — Pending |
| Source-agnostic meeting sync | Avoid vendor lock-in, user may switch from Granola | — Pending |
| CLI + Obsidian + Web triple interface | CLI for actions, Obsidian for deep work, Web for quick overview | — Pending |
| GTD + Tasks as first priority | Foundation that everything else builds on | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-03-29 after Phase 1 (GTD Core) completion — 15 skills delivered, daily productivity loop operational*
