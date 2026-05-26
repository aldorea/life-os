---
title: Life OS — Adaptive Planning Loop
status: final
created: 2026-05-25
updated: 2026-05-25
---

# Life OS — Adaptive Planning Loop

## 1. Overview

This document specifies the Adaptive Planning Loop for Life OS: a set of behaviors layered on top of the existing Cadence System that closes the gap between intent and execution. The loop follows a six-step cycle: **Clarify → Plan → Execute → Log → Detect → Adapt**. All new behavior lives inside the existing `morning` and `close` skills — no new top-level skills are introduced.

The target user is Sito — a software consultant and startup co-founder managing multiple Jira instances, with a variable calendar and a need for a reliable single source of truth for what to do today, this week, and why things slipped.

### Relationship to the Cadence System PRD

This PRD extends `prd-life-os-2026-05-24` (Life OS Cadence System). The Cadence PRD defines the frente health layer, commitment tracking, weekly and monthly review rhythm, and the core morning/close structure. This PRD is additive: it inserts new steps into the existing morning and close flows, and appends new subsections to the Monday Weekly Review already defined in Cadence PRD FR-3.4. All Cadence PRD FRs remain in force; conflicts resolve in favor of the Cadence PRD unless explicitly overridden here.

**On horizon gating:** The brief proposed gating weekly planning behind demonstrated daily adherence (≥4 days/week for 2 weeks). GTD canon — and the consensus of major GTD implementations — does not support this: the Weekly Review is the trust-building mechanism itself, not a reward for having already built trust. This PRD adopts the GTD-correct position: weekly planning is active from Day 1. The data-sufficiency guard in FR-P7.3 ("Datos insuficientes...") is the appropriate lightweight signal when data is thin, without locking the user out of planning.

---

## 2. Problem Statement

The Cadence System ensures Sito tracks frentes, commitments, and check-ins. What it does not yet address is the planning quality gap:

- Each morning starts with a list of tasks but no deliberate allocation to days or calendar blocks.
- The close ritual captures energy and blockers but does not record *what actually happened* or *why plans slipped*.
- After weeks of use, there is no mechanism to surface patterns (e.g., "Wednesdays are consistently disrupted") that would inform better planning.

The result: each week is planned in isolation, slippage goes unanalyzed, and Sito lacks the data to push back on interruptions or restructure the week proactively.

---

## 3. Goals and Non-Goals

### Goals

- Surface a clarification step before any plan is proposed, so planning input is accurate.
- Generate a weekly plan every Monday and a focused daily brief every morning.
- Infer the day's activity at close from Jira, calendar, and meeting notes — never require recall as the primary input.
- Tag disruptions with a structured taxonomy and surface weekly summaries.
- After sufficient data, surface recurring patterns as read-only observations.
- Provide a ready-to-use standup summary in the daily note each morning.

### Non-Goals (Out of Scope)

- Automated replanning without user confirmation.
- Calendar writes of any kind — planning is read-only on all calendars.
- Personal frentes in the planning loop (deferred until the work loop is stable).
- Mobile or push notifications outside the CLI session.
- Multi-week planning horizons beyond the current week.
- Quarterly or annual planning (not until monthly cadence is proven per the Cadence PRD horizon hierarchy).
- Free-form disruption types — fixed taxonomy only in this version.

---

## 4. Feature Requirements

### F1 — Clarification Step (Morning)

**Trigger:** Every morning run, before any plan is proposed.

**FR-P1.1** Before generating any plan, `morning` shall present the current backlog and Jira tickets pulled by the last sync run, then ask: "Before we plan — is this list complete? Anything missing, anything that shouldn't be here?"

**FR-P1.2** The user may respond by adding items, removing items, or marking items as irrelevant for the current period. The confirmed list becomes the sole planning input for that morning's brief and, on Mondays, for the weekly plan.

**FR-P1.3** If the user explicitly skips clarification (e.g., responds with "skip" or proceeds without input), `morning` shall proceed with the existing list unchanged and write `[clarification skipped]` into the daily note's header block. This makes the skip auditable without blocking execution.

---

### F2 — Weekly Plan Generation (Morning, Mondays)

**Trigger:** Monday morning (or the first morning run of the week if Monday was skipped), after the clarification step.

**FR-P2.1** After clarification, `morning` shall propose a weekly plan that allocates confirmed tasks to Mon–Fri based on: (a) calendar availability for the week as reported by `icalbuddy`, (b) goal priority from `{vault}/config/goals.yaml`, and (c) explicit deadlines on tasks.

**FR-P2.2** The proposed plan shall be written to `{config.structure.weekly_notes}/week-{YYYY-WW}.md` with the following frontmatter:

```yaml
week: YYYY-WW
status: draft
created: YYYY-MM-DD
```

The body shall contain a table: `Day | Tasks | Notes`.

**FR-P2.3** `morning` shall present the proposed plan to the user for approval or adjustment. On confirmation, `morning` shall update the frontmatter `status` to `confirmed`. If a weekly note for the current week already exists with `status: confirmed`, `morning` shall skip re-generation and display the confirmed plan as-is.

**FR-P2.4** `morning` shall run weekly plan generation only on Monday mornings (or the first morning run of the week). On non-Monday mornings, `morning` shall read the existing confirmed weekly note for the daily slice and shall not trigger re-generation.

**FR-P2.5** Task allocation shall respect calendar load. Tasks shall not be allocated to days where calendar occupancy exceeds 80% of the workday. [ASSUMPTION: 80% occupancy is defined as ≥5 hours of calendar events in an 8-hour workday. This threshold is an implementation parameter — adjust based on observed data.]

---

### F3 — Daily Brief (Morning)

**Trigger:** Every morning run, after the weekly plan step (or after clarification on non-Monday runs).

**FR-P3.1** `morning` shall extract today's slice from the confirmed weekly note and present a focused daily brief containing: tasks planned for today, the suggested focus block (the first 2-hour unblocked window in today's calendar), and any deadlines due today or tomorrow.

**FR-P3.2** The day's brief shall be written into the `Foco` section of the daily note. This section is defined in the Cadence PRD flow — this PRD only specifies that the planning content populates it.

**FR-P3.3** If no confirmed weekly note exists for the current week, `morning` shall generate a day-scoped plan instead (tasks ranked by priority, no weekly allocation) and prepend the note: "No hay plan semanal confirmado — mostrando plan de día."

---

### F4 — Standup Summary (Daily Note)

**Trigger:** Every morning run, populated alongside the daily brief.

**FR-P4.1** The daily note shall contain a ready-to-use standup summary section with three pre-populated fields:

1. **Done yesterday** — auto-populated from the prior day's `Cierre` section in `03 Daily/{YYYY-MM-DD}.md`.
2. **Planned today** — taken from the daily brief (FR-P3.1).
3. **Blockers** — taken from active blockers recorded in `CRITICAL_FACTS.md` (the canonical source for blockers).

**FR-P4.2** The standup section shall be pre-populated with no required user action. Sito may edit one field before sharing; the expectation is that the section is usable as-is in the majority of cases.

**FR-P4.3** [ASSUMPTION: "Done yesterday" reads the prior day's daily note `Cierre` section. If that section is absent (close was not run the prior day), the field is left blank with the note "No hay cierre del día anterior."]

---

### F5 — Activity Inference (Close)

**Trigger:** Every close run, before any user questions.

**FR-P5.1** `close` shall infer the day's activity by reading three sources in order: (1) Jira ticket state changes for today via the MCP sync data, (2) calendar events attended today via `icalbuddy`, (3) Granola notes created today, if available.

**FR-P5.2** Before asking anything, `close` shall present the inferred summary: "Parece que hoy: {inferred activity list}. ¿Hay algo que añadir o corregir?" The user sees the inference first — they are never asked to recall from scratch.

**FR-P5.3** The user confirms or corrects the inference. `close` shall write the confirmed activity log to the `Cierre` section of the daily note.

**FR-P5.4** Activity inference is the primary input mechanism. Sito's confirmation or correction is the refinement layer. Manual recall is only required when inference sources return no data.

**FR-P5.5** If no Jira or calendar data is available for the day (e.g., all connectors failed), `close` shall state: "No encontré actividad registrada para hoy. ¿Qué hiciste?" and accept free-text input. This state is treated as a degraded mode, not a failure.

---

### F6 — Disruption Tagging (Close)

**Trigger:** Every close run, after activity inference is confirmed.

**FR-P6.1** When `close` detects unplanned events — calendar events not present in the morning's day brief, or tasks from the weekly plan that did not progress — it shall present them as candidate disruptions for the user to review.

**FR-P6.2** For each candidate disruption, `close` shall auto-classify a disruption type from the following fixed taxonomy:

| Type | Description |
|------|-------------|
| `reunion-extra` | An unplanned meeting or call |
| `incidente-tecnico` | A production issue, outage, or urgent technical problem |
| `peticion-cliente` | An urgent or unplanned client request |
| `bloqueo-externo` | A dependency that blocked progress on a planned task |
| `otro` | Any disruption that does not fit the above four types |

Classification is based on: calendar event title and type, Jira issue type and labels, or Granola note content.

**FR-P6.3** The user confirms or corrects each auto-classification. `close` shall store the confirmed type in the `Cierre` section alongside the disruption description.

**FR-P6.4** If auto-classification confidence is low (i.e., the system cannot reliably distinguish between types), `close` shall present the taxonomy options explicitly and ask the user to select, rather than guessing.

**FR-P6.5** Each stored disruption record shall include: date, type (from taxonomy), a one-sentence description, and the planned task it displaced (if applicable). This structured record enables weekly and pattern-level analysis.

---

### F7 — Weekly Review Extension (Monday Morning)

**Trigger:** Monday morning run. Extends Cadence PRD FR-3.4 — does not replace it.

**FR-P7.1** The Monday Weekly Review section (defined in Cadence PRD FR-3.4) shall be extended with four additional subsections, appended after the existing Frentes table and Compromisos section:

**Plan Adherence**
Auto-computed: the percentage of tasks originally in last week's confirmed weekly note (`week-{YYYY-WW}.md`, prior week) that appear as completed in the week's close logs. Only tasks present in the confirmed weekly note at the time of confirmation are counted — tasks added during mid-week clarification steps are excluded from this metric (they are today-only additions, not weekly commitments). Presented as: "Semana pasada: completaste X de Y tareas planificadas (Z%)."

**Tasks That Didn't Happen**
Auto-populated list of tasks from last week's weekly note that were not completed, with the disruption type(s) logged during that week from close records. Format: task name | disruption type(s) | number of times displaced.

**Disruption Summary**
Auto-computed count per disruption type across all close logs for the prior week. Example output:

```
reunion-extra:       3
peticion-cliente:    1
bloqueo-externo:     1
```

**Proposed Adjustments**
One or two sentences generated by the system based on the disruption pattern (e.g., "Los miércoles tuviste 2 reuniones no planificadas — considera dejar ese día con menos tareas comprometidas."). User may edit or discard. This is the only point where the system offers a forward-looking suggestion derived from data.

**FR-P7.2** The extension is purely additive. It does not modify or remove the Frentes table, commitment counts, or any other element defined in Cadence PRD FR-3.4.

**FR-P7.3** If fewer than 3 days of close logs exist for the prior week, all four subsections shall display: "Datos insuficientes para calcular adherencia — menos de 3 cierres registrados la semana pasada." No partial computation is shown.

---

### F8 — Pattern Detection (Monday Morning)

**Trigger:** Monday morning run, after 4 or more full weeks of close logs exist. Displayed above the weekly plan step.

**FR-P8.1** After 4 or more weeks of close logs exist, `morning` shall surface a Patterns section on Monday mornings, before the weekly plan is proposed.

**FR-P8.2** Pattern detection shall identify recurring conditions across weeks, specifically:

- Day-of-week disruption frequency (e.g., "los lunes tienen de media 1.5 disrupciones por semana").
- Disruption types that recur in 2 or more consecutive weeks.
- Frentes with consistent inactivity gaps of more than 3 days across multiple weeks.

**FR-P8.3** Patterns are surfaced as read-only, factual observations. The system shall not make automated planning changes based on patterns. No action is implied or requested. Example output: "Las últimas 3 semanas, los martes tienes de media 1.5 reuniones no planificadas."

**FR-P8.4** If fewer than 4 weeks of close log data exist, pattern detection runs silently — no "not enough data yet" message is shown.

**FR-P8.5** [ASSUMPTION: Pattern detection reads disruption records from the `Cierre` sections of daily notes in `03 Daily/`, filtered by the structured disruption fields written by F6. The exact parsing strategy (regex, YAML block, or structured marker) is an implementation decision — the PRD requires only that the data format written in F6 is sufficient to support the computation defined here.]

---

## 5. Non-Functional Requirements

**NFR-P1 — No new top-level skills.** All adaptive planning behavior lives inside `morning` and `close`. The skills `/plan`, `/weekly`, `/log`, and `/review` shall not be introduced.

**NFR-P2 — Non-blocking inference.** Activity inference (F5) is non-blocking. If the Jira MCP is unavailable, `close` proceeds with calendar-only inference and logs the gap. If calendar is also unavailable, `close` falls back to manual input (FR-P5.5). Close shall not hang waiting for an unavailable connector.

**NFR-P3 — Monday morning runtime budget.** Weekly plan generation (F2) shall complete within the total Monday morning runtime budget of ≤10 minutes established in Cadence PRD NFR-3. Plan generation, clarification, and brief generation together shall not exceed 3 minutes of this budget.

**NFR-P4 — Spanish user-facing output.** All text displayed to Sito during morning and close runs shall be in Spanish. Internal logs (daily note `Cierre` section, weekly note frontmatter) may use English field names for parsing stability.

**NFR-P5 — Immutable confirmed weekly plans.** Once a weekly note's frontmatter has `status: confirmed`, `morning` shall not overwrite the task allocation. Re-generation requires an explicit user action (e.g., deleting the note or changing its status back to `draft` manually).

**NFR-P6 — Fixed disruption taxonomy.** The five disruption types defined in FR-P6.2 are the complete set for this version. Free-form descriptions are captured alongside the type, but the `type` field must always contain one of the five defined values. [ASSUMPTION: `otro` is sufficient to cover edge cases in the initial version. A 6th type may be added only via an explicit scope change, not via configuration.]

**NFR-P7 — Read-only pattern detection.** Pattern detection (F8) surfaces observations only. No automated replanning, no automated weekly note modification, and no automated task reordering results from pattern detection.

**NFR-P8 — Cadence PRD primacy.** This PRD layers on top of the Cadence System PRD (`prd-life-os-2026-05-24`). All FRs from the Cadence PRD remain in force. In any conflict between this PRD and the Cadence PRD, the Cadence PRD takes precedence unless this document explicitly states otherwise.

**NFR-P9 — Low-friction, conversational UX.** The loop's narrative coherence depends on morning and close feeling like a natural workflow, not a form to fill. All user-facing prompts shall be phrased conversationally (as shown in FR-P1.1 and FR-P5.2). The principle "inference first, recall never" (FR-P5.4) applies globally — no step shall ask the user to produce information the system can already read. This principle is intentionally qualitative; the implementing skill author shall apply it across every prompt.

**NFR-P10 — Loop narrative.** Morning and close are steps in a coherent daily loop, not independent commands. The skills shall not need to explicitly announce "you are at step N" — but the flow order, the framing of prompts, and the structure of the daily note together shall make the loop self-evident to the user over time. This is an implementation-quality concern, not a strict behavioral requirement.

---

## 6. Integration Notes

### Morning Sequence (Monday)

The complete Monday morning run order, combining the Cadence PRD and this PRD:

| Step | Source | Description |
|------|--------|-------------|
| 1 | Cadence PRD | Sync — external data pull via `/sync` |
| 2 | Cadence PRD | Inbox processing |
| 3 | Cadence PRD FR-3.1 | Frente snapshot |
| 4 | Cadence PRD FR-3.2 | Commitment display |
| 5 | Cadence PRD FR-3.3 | Blocker surfacing |
| 6 | Cadence PRD FR-3.4 + this PRD FR-P7.1 | Weekly Review (Cadence base + Adaptive extensions) |
| 7 | **This PRD FR-P1.x** | **Clarification step** |
| 8 | **This PRD FR-P8.x** | **Pattern detection** (only if ≥4 weeks of close logs) |
| 9 | **This PRD FR-P2.x** | **Weekly plan generation** |
| 10 | **This PRD FR-P3.x** | **Day's brief** |
| 11 | **This PRD FR-P4.x** | **Standup summary** written to daily note |

### Morning Sequence (Non-Monday)

| Step | Source | Description |
|------|--------|-------------|
| 1–5 | Cadence PRD | Sync, inbox, frente snapshot, commitments, blockers |
| 6 | **This PRD FR-P1.x** | **Clarification step** |
| 7 | **This PRD FR-P3.x** | **Daily brief** (slice from confirmed weekly note) |
| 8 | **This PRD FR-P4.x** | **Standup summary** written to daily note |

### Close Sequence

| Step | Source | Description |
|------|--------|-------------|
| 1 | Cadence PRD FR-4.1 | Commitment check-in |
| 2 | Cadence PRD FR-4.2 | Presencia check-in (non-skippable) |
| 3 | **This PRD FR-P5.x** | **Activity inference** — presented before any user questions |
| 4 | **This PRD FR-P6.x** | **Disruption tagging** — auto-classify, user confirms |
| 5 | Cadence PRD | Reflection questions |
| 6 | Cadence PRD | Task decisions (carry-forward, remove) |
| 7 | Cadence PRD | Wiki log append |

### Vault Paths

| Artifact | Path |
|----------|------|
| Weekly notes | `{config.structure.weekly_notes}/week-{YYYY-WW}.md` (resolves to `04 Weekly/`) |
| Daily notes | `03 Daily/{YYYY-MM-DD}.md` |
| Goals | `{vault}/config/goals.yaml` |
| Connectors | `{vault}/config/connectors.yaml` |
| Identity layer | `08 Resources/CRITICAL_FACTS.md` |

All vault reads and writes go through the `obsidian` CLI — never through direct filesystem access against iCloud paths.

---

## 7. Success Metrics

| Metric | Target | Counter-metric |
|--------|--------|----------------|
| Weekly plan confirmed every Monday | ≥3 out of 4 Mondays/month | If weekly plan generation adds >3 min to Monday morning, the implementation is over-engineered |
| Close activity log matches Jira/calendar reality | ≥80% of events captured without manual additions | If Sito adds >2 manual items per close, inference quality is insufficient |
| Standup used as-is | Sito edits ≤1 field before sharing, ≥50% of days | If rewrite rate >50%, standup format doesn't match Sito's communication style |
| Daily loop adherence | morning + close completed ≥4 days/week | — |
| Pattern signal quality | ≥1 actionable pattern surfaced within 6 weeks of consistent use | If all surfaced patterns are too generic to act on, detection logic needs refinement |
| Disruption auto-classification accuracy | Auto-classification accepted without correction ≥70% of the time | If correction rate >30%, auto-classify should default to showing options rather than guessing |
| Skill surface stability | No new top-level skills introduced; all loop behavior lives inside `morning` and `close` | Any new top-level skill added for planning, logging, or review is a scope violation |

---

## 8. Open Questions

| # | Question | Urgency |
|---|----------|---------|
| OQ-1 | What is the minimum Jira/calendar data needed for reliable activity inference? Is the current sync output (state changes + event titles) sufficient, or do we need richer metadata such as time spent or PR descriptions? | Phase 1 |
| OQ-2 | How should the weekly plan handle tasks with no explicit deadline — ranked by goal weight from `goals.yaml`, by a manual priority field, or by asking the user once during the clarification step? | Phase 1 |
| OQ-3 | Pattern detection threshold: 4 full weeks is assumed. Should this threshold be configurable in `config.yaml`? And should patterns appear inline in Monday morning or as a dedicated report accessible via an explicit command? | Phase 2 |
| OQ-4 | Disruption taxonomy: 5 fixed types assumed. Is `otro` sufficient to cover genuine edge cases across the first 6 weeks of use, or should a mechanism to propose a new type (requiring Sito's confirmation) be included from the start? | Phase 1 |
| OQ-5 | Monday morning sequencing: the clarification step (FR-P1.1) is currently placed after the full Cadence weekly review block (step 6). Should clarification happen earlier — before the frente snapshot and commitment display — so the confirmed task list informs those views as well? **Architecture phase flag:** the full morning sequence (Cadence PRD steps + this PRD steps) requires explicit sequencing work in the architecture phase to avoid conflicts. | Phase 1 |
| OQ-6 | What structured format should the daily brief's `Foco` section use to enable machine-readable disruption detection at close time? F6 needs to diff "tasks planned today" vs "tasks that progressed" — freetext prose is insufficient. The architecture phase must define a parseable format (e.g., a YAML block, a checkbox list with a marker, or a dedicated section header). | Phase 1 |

---

## 9. Appendix — Disruption Taxonomy Reference

| Type | When to use | Example signals |
|------|-------------|-----------------|
| `reunion-extra` | Unplanned meeting or call that consumed calendar time | Calendar event not in morning brief; Granola note for an unknown meeting |
| `incidente-tecnico` | Production issue, outage, or urgent debugging session | Jira issue type = Bug with high priority; Slack incident channel activity |
| `peticion-cliente` | Urgent or unplanned client request requiring immediate attention | Jira issue created by an external user; email or Slack from a client contact |
| `bloqueo-externo` | External dependency that blocked a planned task from progressing | Jira ticket moved to "blocked" status; waiting-on label applied |
| `otro` | Any disruption that does not match the above four types | Use when confident the other types don't apply; a one-sentence description is required |

Auto-classification confidence is considered **low** when: the event or issue title is ambiguous, multiple types could plausibly apply, or the data source returned no title or description. In low-confidence cases, `close` presents the full options table and asks the user to select.
