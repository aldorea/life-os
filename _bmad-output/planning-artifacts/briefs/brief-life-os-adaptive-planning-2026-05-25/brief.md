---
title: Life OS — Adaptive Planning Loop
status: final
created: 2026-05-25
updated: 2026-05-25
---

## Problem

Life OS captures tasks and syncs external data — but it doesn't help Sito plan. It answers "what is there to do?" but not "what should I actually do today, and did I do it?" Without a clarification step before execution, Sito jumps into work, gets disrupted, loses track of what was planned, and ends the day without a clear picture of what happened or why things changed.

The deeper failure: no activity log, no pattern detection, no learning. The same disruptions recur — unexpected meetings, client incidents, scope shifts — but the system has no memory of them. No feedback loop exists between what is planned and what actually happens, so the workflow never improves.

The secondary symptom — "too many skills, hard to find" — follows from this. Skills feel arbitrary because they don't map to a clear workflow. When the workflow is coherent, the tools make sense.

---

## Vision

Life OS becomes a **planning system that learns**. Every day starts with clarity (what are we actually doing, and why), every day ends with a record (what happened, what changed, what the disruption was), and over time the system surfaces patterns that help Sito improve his own workflow — not because someone told him to, but because the data is there.

The loop:

```
Clarify → Plan → Execute → Log (infer + confirm) → Detect patterns → Adapt
```

Grounded in GTD: build trust in the daily system first, then raise the abstraction level. Daily reliability unlocks weekly planning. Weekly reliability unlocks monthly goals. The hierarchy earns its way up.

---

## Users & Context

**Single user**: Sito — software consultant and startup co-founder, operating across two concurrent work contexts (Orbitant consulting, Previene startup). Runs Life OS in the terminal (Claude Code CLI) primarily when at the computer.

**Scope for this brief**: Work and computer tasks. Personal frentes are out of scope for the initial loop — they are part of the roadmap once the work loop is stable.

**External data available**: Jira (two instances), Google Calendar, Granola (meeting notes), Slack, Telegram captures.

---

## Core Loop

### 1. Morning — Clarify + Plan

`morning` pulls existing data (backlog, Jira, calendar for the week) and **requires a clarification step** before generating any plan:

> *"Before we plan — is this list complete? Anything missing, anything that shouldn't be here?"*

Once the list is trusted, the system proposes a **weekly plan**: tasks allocated to days based on calendar availability, goal priority, and deadlines. Sito approves or adjusts. The approved plan anchors the week's daily note.

Each morning also generates the **day's brief**: what is planned for today specifically, with a suggested focus block.

### 2. During the day — Standup output

The daily note contains a ready-to-use **standup summary**: what was done yesterday, what is planned today, any blockers. Sito reads it, optionally edits one line, and uses it as-is.

### 3. Close — Log + Infer

`close` **infers activity** by reading Jira (tickets updated/closed), calendar (meetings attended), and sync data from the day — it does not ask Sito to recall what he did. It presents a summary:

> *"Parece que hoy: cerraste PR #42, tuviste 2 reuniones no planificadas, no avanzaste en X. ¿Hay algo que añadir?"*

Sito confirms or adds context — interruptions, blockers, reasons a task didn't happen. Any unplanned event receives a **type tag**: `reunion-extra`, `incidente-tecnico`, `peticion-cliente`, `bloqueo-externo`, or `otro`.

The confirmed log is written to the daily note's Cierre section.

### 4. Weekly — Adapt + Review

On Monday morning, `morning` prepends a **weekly review** before the plan:

- Commitment adherence from last week
- Tasks that were planned but didn't happen (and why, from the close logs)
- Disruption summary: types and frequency
- Proposed adjustments to this week's plan based on last week's reality

### 5. Pattern detection — Learn

After sufficient data accumulates, `morning` or a dedicated report surfaces recurring patterns:

> *"Las últimas 3 semanas, los lunes tienes de media 1.5 reuniones no planificadas. Tu planning del lunes sobreestima consistentemente la capacidad disponible."*

This is read-only intelligence — no automated replanning. Sito uses it to adjust his own workflow. See OQ-3 for the data threshold and implementation cadence.

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Daily loop adherence | morning + close run ≥4 days/week |
| Clarification step completion | Weekly plan confirmed every Monday |
| Standup quality | Sito uses the system-generated standup without rewriting it |
| Activity log completeness | Close log matches Jira/calendar reality ≥80% of the time without manual additions |
| Pattern signal | At least one actionable pattern identified within 6 weeks of consistent use |
| Skill surface | No new top-level skills added; loop lives within morning + close |

**North star**: Sito ends each week knowing what he did, why the plan changed, and what to do differently next week — without effort.

---

## Horizon Hierarchy

The system supports increasing abstraction, earned by reliability at the lower level:

| Horizon | Cadence | Unlocked when |
|---------|---------|---------------|
| Daily | Every morning/close | Day 1 |
| Weekly | Monday review | Daily loop runs ≥4 days/week for 2 weeks |
| Monthly | 1st of month | Weekly reviews completed ≥3 weeks/month |
| Quarterly | Future | Monthly reviews completed ≥2 months/quarter |

---

## Design Principles & Constraints

**Skill surface is already correct.** Claude Code's documented best practice is one-skill-per-function — exactly what Life OS does. No hub command, no sub-command syntax, no redesign needed.

**User-facing commands stay minimal.** The only commands a user needs to know:

| Command | Does |
|---------|------|
| `/morning` | Start of day — sync, clarify, plan |
| `/rituals:close` | End of day — log, reflect, adapt |
| `/sync` | Manual sync of all sources |
| `/wiki:query` | Search personal knowledge base |
| `/wiki:ingest` | Add a source to the wiki |

Everything else is internal. `sync-jira`, `sync-granola`, `sync-telegram` and similar sub-skills must have no `description:` field — they disappear from the user menu and are only invoked by `/sync` internally. This is the fix, not a redesign.

**Coherence comes from the planning loop, not from the command list.** Skills feel arbitrary today because no narrative connects them. The adaptive planning loop (morning → close → learn) is that narrative. Once the loop exists, the commands make sense as steps in a workflow.

**No new top-level skills.** All new capabilities build inside existing commands. The planning loop, activity log, and pattern detection live inside `morning` and `close`.

---

## Out of Scope

- Personal frentes (health, relationship, learning) — roadmap, not MVP
- Automated replanning without user confirmation
- Calendar writes of any kind
- Mobile or non-CLI interfaces
- Multi-user or team planning
- Quarterly/annual horizon (not until monthly is proven)
- Automated workflow improvements — the system surfaces patterns, Sito decides what to change

---

## Open Questions

| # | Question | Urgency |
|---|----------|---------|
| OQ-1 | What is the minimum Jira/calendar data needed to generate a reliable activity inference? Is the current sync sufficient? | Phase 1 |
| OQ-2 | How should the weekly plan handle tasks with no clear deadline — rank by goal weight, or ask Sito explicitly? | Phase 1 |
| OQ-3 | Pattern detection threshold: how many weeks of data before surfacing patterns, and should this run as a periodic report or inside `morning`? (4 weeks assumed as starting point.) | Phase 2 |
| OQ-4 | Should disruption type tags be free-form or a fixed taxonomy? Fixed reduces noise but may not cover everything. | Phase 1 |
| OQ-5 | How does this loop interact with the Cadence System PRD (frentes, commitments)? Same morning/close, layered on top. Needs explicit sequencing in the architecture phase. | Phase 1 |
