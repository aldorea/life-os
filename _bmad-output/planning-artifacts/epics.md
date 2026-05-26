---
stepsCompleted: ["step-01", "step-02"]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-life-os-2026-05-24/prd.md
  - _bmad-output/planning-artifacts/prds/prd-life-os-2026-05-25/prd.md
---

# Life OS — Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for Life OS, decomposing requirements from the Cadence System PRD and the Adaptive Planning Loop PRD into implementable stories for the `morning`, `close`, and `today` skills.

---

## Requirements Inventory

### Functional Requirements

#### From Cadence System PRD (prd-life-os-2026-05-24)

FR-1.1: Each task in the backlog MAY carry a `frente:` tag (e.g. `frente: salud`, `frente: portal-del-cliente`). Opt-in; untagged tasks appear under "Sin frente."
FR-1.2: Goals in `goals.yaml` MAY carry a `frente:` field. Metadata only — scoring algorithm unaffected.
FR-1.3: The two top-level domains are `personal` and `work`. Frentes belong to one domain. Mapping declared in a new `frentes:` section in `goals.yaml`.
FR-1.4: No migration of existing tasks required at launch. Frente tagging adopted incrementally.
FR-2.1: Active commitments shall be declared in `CRITICAL_FACTS.md` under `## Compromisos activos`. Format: one commitment per line with short ID (e.g. `- [C1] Cerrar a las 18:00`). Maximum 3 active commitments.
FR-2.2: `morning` shall surface the active commitments list at the start of each day ("Hoy me comprometí a: …"). Display only.
FR-2.3: `close` shall prompt one binary check-in per commitment: "¿Lo cumpliste?" (sí / no / parcial). Responses recorded in daily note Cierre section.
FR-2.4: When ≥2 consecutive "no" responses recorded for the same commitment, `morning` shall surface a low-friction alert the following day. Informational, not blocking.
FR-3.1: `morning` shall append a compact frente snapshot after the daily note is generated: last task + days since last activity per active frente. Frentes with >3 days inactivity flagged with ⚠️.
FR-3.2: Immediately after the frente snapshot, system shall display the active commitments list. Read-only.
FR-3.3: When `CRITICAL_FACTS.md` or a `blockers:` section in `connectors.yaml` contains active blockers, `morning` shall surface them one line each: "🔴 [BLOCKER] …".
FR-3.4: On Monday mornings, `morning` shall prepend a Weekly Review section to the daily note: auto-populated frente table (last activity) + commitment counts (sí/no/parcial from past week). User fills Estado (verde/amarillo/rojo/transición) and Foco de la semana.
FR-3.5: On the 1st of each month, `morning` shall prepend a Monthly Horizon section with auto-populated frente activity and commitment data. Replaces Weekly Review when the 1st is a Monday; otherwise both coexist.
FR-4.1: `close` shall run commitment check-ins before reflection questions. Each active commitment gets a one-line prompt; user answers sí/no/parcial. Maximum 3 commitments.
FR-4.2: `close` shall include one fixed, non-skippable reflection question: "¿Estuviste presente hoy?" (sí / no / parcial). Response stored in Cierre section.
FR-4.3: On Fridays, `close` shall add one optional prompt: "Una frase sobre cómo fue la semana para cada frente activo." If answered, response stored in Cierre and used to pre-populate next Monday's Weekly Review frente table.
FR-4.4: `close` shall record commitment responses in the daily note. Lightweight aggregation reads last 7 days of daily notes to compute streaks for FR-2.4.
FR-5.1: The existing "Trabajo" and "Personal" task sections in the daily note shall group tasks by frente within each domain. Untagged tasks appear under "Sin frente."
FR-5.2: A compact one-liner shall appear above task sections: "Frentes activos hoy: [list]." Omitted if none.

#### From Adaptive Planning Loop PRD (prd-life-os-2026-05-25)

FR-P1.1: Before generating any plan, `morning` shall present the current backlog and Jira tickets and ask: "Before we plan — is this list complete? Anything missing, anything that shouldn't be here?"
FR-P1.2: User may add items, remove items, or mark items as irrelevant. Confirmed list becomes the sole planning input for that morning's brief and, on Mondays, for the weekly plan.
FR-P1.3: If user skips clarification, `morning` shall proceed with the existing list unchanged and write `[clarification skipped]` into the daily note's header block.
FR-P2.1: After clarification, `morning` shall propose a weekly plan allocating confirmed tasks to Mon–Fri based on: (a) calendar availability via `icalbuddy`, (b) goal priority from `goals.yaml`, (c) explicit task deadlines.
FR-P2.2: The proposed plan shall be written to `{config.structure.weekly_notes}/week-{YYYY-WW}.md` with frontmatter `week`, `status: draft`, `created`; body contains a `Day | Tasks | Notes` table.
FR-P2.3: `morning` shall present the plan for approval. On confirmation, `morning` shall update frontmatter `status` to `confirmed`. If a confirmed weekly note already exists, skip re-generation and display as-is.
FR-P2.4: Weekly plan generation runs only on Monday mornings (or first morning run of the week). Non-Monday mornings read existing confirmed note for daily slice only.
FR-P2.5: Task allocation shall not assign tasks to days where calendar occupancy exceeds 80% (≥5 hours of calendar events in an 8-hour workday).
FR-P3.1: `morning` shall extract today's slice from the confirmed weekly note and present a focused daily brief: tasks for today, suggested focus block (first 2-hour unblocked window), deadlines due today or tomorrow.
FR-P3.2: The daily brief shall be written into the `Foco` section of the daily note.
FR-P3.3: If no confirmed weekly note exists, `morning` shall generate a day-scoped plan (tasks ranked by priority) and prepend: "No hay plan semanal confirmado — mostrando plan de día."
FR-P4.1: The daily note shall contain a standup summary section with three pre-populated fields: (1) Done yesterday — from prior day's Cierre; (2) Planned today — from daily brief; (3) Blockers — from CRITICAL_FACTS.md.
FR-P4.2: Standup section shall be pre-populated with no required user action.
FR-P4.3: If prior day's Cierre is absent, "Done yesterday" field is left blank with note "No hay cierre del día anterior."
FR-P5.1: `close` shall infer the day's activity by reading in order: (1) Jira ticket state changes today via MCP, (2) calendar events attended via `icalbuddy`, (3) Granola notes created today.
FR-P5.2: Before asking anything, `close` shall present: "Parece que hoy: {inferred activity list}. ¿Hay algo que añadir o corregir?" Inference always comes first.
FR-P5.3: User confirms or corrects. `close` shall write confirmed activity log to the Cierre section of the daily note.
FR-P5.4: Activity inference is the primary input mechanism. Manual recall only when inference sources return no data.
FR-P5.5: If no Jira or calendar data available, `close` shall state: "No encontré actividad registrada para hoy. ¿Qué hiciste?" and accept free-text input.
FR-P6.1: When `close` detects unplanned events (calendar events not in morning's brief, or weekly plan tasks that did not progress), it shall present them as candidate disruptions.
FR-P6.2: `close` shall auto-classify each disruption from the taxonomy: `reunion-extra`, `incidente-tecnico`, `peticion-cliente`, `bloqueo-externo`, `otro`. Classification based on event title/type, Jira issue type, or Granola content.
FR-P6.3: User confirms or corrects each auto-classification. `close` shall store the confirmed type in Cierre.
FR-P6.4: If auto-classification confidence is low, `close` shall present the taxonomy options explicitly.
FR-P6.5: Each disruption record shall include: date, type, one-sentence description, and the planned task it displaced (if applicable).
FR-P7.1: The Monday Weekly Review (Cadence FR-3.4) shall be extended with four subsections: Plan Adherence (% of confirmed tasks completed), Tasks That Didn't Happen (with disruption types), Disruption Summary (count per type for the week), Proposed Adjustments (1-2 system-generated sentences from disruption pattern).
FR-P7.2: The extension is additive only — does not modify or remove any Cadence PRD FR-3.4 content.
FR-P7.3: If fewer than 3 days of close logs exist for the prior week, all four subsections display: "Datos insuficientes para calcular adherencia — menos de 3 cierres registrados la semana pasada."
FR-P8.1: After 4+ weeks of close logs, `morning` shall surface a Patterns section on Monday mornings before the weekly plan.
FR-P8.2: Pattern detection shall identify: day-of-week disruption frequency, disruption types recurring in ≥2 consecutive weeks, frentes with consistent >3-day inactivity gaps.
FR-P8.3: Patterns are read-only factual observations. No automated planning changes.
FR-P8.4: If fewer than 4 weeks of close logs, pattern detection runs silently.
FR-P8.5: Pattern detection reads Cierre sections of daily notes in `03 Daily/` filtered by structured disruption fields from F6.

---

### Non-Functional Requirements

#### From Cadence System PRD

NFR-1: No new top-level skills. All cadence behavior inside `morning`, `close`, and `today`.
NFR-2: Graceful degradation — when frente tags absent, rituals run without frente layer.
NFR-3: Time budget — Monday morning ≤10 min; all other days within 2 min of current baseline.
NFR-4: No calendar writes. System reads calendar; never writes.
NFR-5: Vault-only persistence — all cadence data in daily notes and CRITICAL_FACTS.md.
NFR-6: Incremental adoption — Phase 1: commitments + Monday review. Frente tagging follows.

#### From Adaptive Planning Loop PRD

NFR-P1: No new top-level skills — `/plan`, `/weekly`, `/log`, `/review` shall not be introduced.
NFR-P2: Activity inference non-blocking — if Jira MCP unavailable, close proceeds with calendar-only; if both unavailable, falls back to manual input.
NFR-P3: Weekly plan generation + clarification + brief ≤3 min within Monday's 10-min budget.
NFR-P4: All user-facing output in Spanish. Internal logs may use English field names.
NFR-P5: Confirmed weekly notes are immutable — re-generation only via explicit user action.
NFR-P6: Fixed disruption taxonomy — `type` field must always be one of the five defined values.
NFR-P7: Pattern detection read-only — no automated replanning, note modification, or reordering.
NFR-P8: Cadence PRD primacy — all Cadence PRD FRs remain in force; conflicts favor Cadence PRD.
NFR-P9: Low-friction conversational UX — inference first, never ask for information the system can read.
NFR-P10: Loop narrative — flow order and prompt framing shall make the daily loop self-evident over time.

---

### Additional Requirements

- Vault I/O exclusively via `obsidian` CLI — never direct filesystem access against iCloud paths
- Jira data available via two MCP instances: `mcp__jira-afianza__` and `mcp__jira-previene__`
- Calendar data via `icalbuddy` (read-only across 3 calendars)
- Meeting notes via Granola (source-agnostic abstraction)
- Weekly notes path: `04 Weekly/week-{YYYY-WW}.md` (config.structure.weekly_notes)
- Daily notes path: `03 Daily/{YYYY-MM-DD}.md`
- Goals config: `{vault}/config/goals.yaml`
- Identity layer: `08 Resources/CRITICAL_FACTS.md`
- Skills are markdown SKILL.md files — no compiled code, no web app

---

### UX Design Requirements

N/A — CLI-only tool. No UX design document. All interaction is conversational text in the Claude Code terminal.

---

### FR Coverage Map

FR-1.1: Epic 1 — frente tag on tasks (opt-in)
FR-1.2: Epic 1 — frente field on goals.yaml (metadata)
FR-1.3: Epic 1 — frentes: section in goals.yaml with domain mapping
FR-1.4: Epic 1 — no migration required at launch
FR-2.1: Epic 1 — commitments declared in CRITICAL_FACTS.md
FR-2.2: Epic 2 — morning surfaces commitments list daily
FR-2.3: Epic 4 — close prompts binary check-in per commitment
FR-2.4: Epic 2 — morning alerts when ≥2 consecutive "no" for a commitment
FR-3.1: Epic 2 — morning appends frente snapshot (last activity + ⚠️ flag)
FR-3.2: Epic 2 — morning displays commitments list after frente snapshot
FR-3.3: Epic 2 — morning surfaces blockers from CRITICAL_FACTS.md
FR-3.4: Epic 2 — Monday morning Weekly Review section (frente table + commitment counts)
FR-3.5: Epic 2 — 1st of month Monthly Horizon section
FR-4.1: Epic 4 — close runs commitment check-ins before reflection questions
FR-4.2: Epic 4 — close non-skippable Presencia check-in
FR-4.3: Epic 4 — close Friday optional frente note prompt
FR-4.4: Epic 4 — close records commitment responses; aggregates streaks
FR-5.1: Epic 1 — today groups tasks by frente within domain sections
FR-5.2: Epic 1 — today shows "Frentes activos hoy:" one-liner
FR-P1.1: Epic 3 — morning presents backlog + Jira and asks clarification question
FR-P1.2: Epic 3 — user can add/remove/mark irrelevant items; confirmed list = planning input
FR-P1.3: Epic 3 — skip clarification → log [clarification skipped] in daily note
FR-P2.1: Epic 3 — morning proposes weekly plan (calendar + goals + deadlines)
FR-P2.2: Epic 3 — weekly note written to 04 Weekly/week-YYYY-WW.md with frontmatter
FR-P2.3: Epic 3 — morning presents plan for approval; status: confirmed on approval; immutable after
FR-P2.4: Epic 3 — weekly plan generation only on Monday mornings
FR-P2.5: Epic 3 — no task allocation to days with >80% calendar occupancy
FR-P3.1: Epic 3 — morning extracts daily slice from weekly note (tasks + focus block + deadlines)
FR-P3.2: Epic 3 — daily brief written into Foco section of daily note
FR-P3.3: Epic 3 — fallback: day-scoped plan if no confirmed weekly note
FR-P4.1: Epic 3 — standup section in daily note (done yesterday, planned today, blockers)
FR-P4.2: Epic 3 — standup pre-populated, no user action required
FR-P4.3: Epic 3 — done yesterday from prior day Cierre; blank + note if absent
FR-P5.1: Epic 4 — close reads Jira state changes + calendar events + Granola notes
FR-P5.2: Epic 4 — close presents inferred summary before asking anything
FR-P5.3: Epic 4 — confirmed activity log written to Cierre section
FR-P5.4: Epic 4 — inference-first always; recall only as last resort
FR-P5.5: Epic 4 — degraded mode if no data: ask free-text
FR-P6.1: Epic 4 — close detects unplanned events as candidate disruptions
FR-P6.2: Epic 4 — auto-classify disruption type from 5-type taxonomy
FR-P6.3: Epic 4 — user confirms/corrects; confirmed type stored in Cierre
FR-P6.4: Epic 4 — low confidence → show taxonomy options explicitly
FR-P6.5: Epic 4 — disruption record: date, type, description, displaced task
FR-P7.1: Epic 5 — Monday Weekly Review extended: plan adherence, tasks didn't happen, disruption summary, proposed adjustments
FR-P7.2: Epic 5 — extension additive only (no changes to Cadence PRD FR-3.4 content)
FR-P7.3: Epic 5 — <3 close logs → "Datos insuficientes..." message for all four subsections
FR-P8.1: Epic 5 — morning surfaces Patterns section on Mondays after 4+ weeks of data
FR-P8.2: Epic 5 — detect: day-of-week disruption frequency, recurring types, frente inactivity
FR-P8.3: Epic 5 — read-only observations only; no automated changes
FR-P8.4: Epic 5 — silent if <4 weeks of data
FR-P8.5: Epic 5 — reads Cierre disruption records from 03 Daily/ (parsing strategy = implementation decision)

---

## Epic List

### Epic 1: Frente & Commitment Foundation
Sito can tag tasks and goals with frentes (areas of focus), declare behavioral commitments in CRITICAL_FACTS.md, and see tasks grouped by frente in the daily note. This is the semantic layer all subsequent epics build on.
**FRs covered:** FR-1.1, FR-1.2, FR-1.3, FR-1.4, FR-2.1, FR-5.1, FR-5.2

### Epic 2: Morning Situational Awareness
Sito starts each morning with a clear picture: frente health snapshot, active commitments, active blockers, and on Mondays a full Weekly Review with frente status and commitment adherence counts. Context before planning.
**FRs covered:** FR-2.2, FR-2.4, FR-3.1, FR-3.2, FR-3.3, FR-3.4, FR-3.5

### Epic 3: Daily Planning Loop (Morning)
Sito confirms his task list each morning (clarification step), gets a weekly plan every Monday written to `04 Weekly/`, a focused daily brief in the `Foco` section, and a ready-to-use standup summary — all pre-populated, no recall required.
**FRs covered:** FR-P1.1, FR-P1.2, FR-P1.3, FR-P2.1, FR-P2.2, FR-P2.3, FR-P2.4, FR-P2.5, FR-P3.1, FR-P3.2, FR-P3.3, FR-P4.1, FR-P4.2, FR-P4.3

### Epic 4: Close Ritual & Activity Logging
Sito closes each day with commitment check-ins (sí/no/parcial), a non-skippable presence check, an inferred activity log built from Jira + calendar + Granola (confirmed, not recalled), and disruptions auto-classified and stored in the Cierre section.
**FRs covered:** FR-2.3, FR-4.1, FR-4.2, FR-4.3, FR-4.4, FR-P5.1, FR-P5.2, FR-P5.3, FR-P5.4, FR-P5.5, FR-P6.1, FR-P6.2, FR-P6.3, FR-P6.4, FR-P6.5

### Epic 5: Weekly Intelligence & Pattern Detection
The loop closes: the Monday Weekly Review shows plan adherence, tasks that didn't happen (with disruption types), a disruption summary, and proposed adjustments. After 4 weeks of data, morning surfaces recurring patterns as read-only observations.
**FRs covered:** FR-P7.1, FR-P7.2, FR-P7.3, FR-P8.1, FR-P8.2, FR-P8.3, FR-P8.4, FR-P8.5

---

## Epic 1: Frente & Commitment Foundation

Sito can tag tasks and goals with frentes (areas of focus), declare behavioral commitments in CRITICAL_FACTS.md, and see tasks grouped by frente in the daily note. This is the semantic layer all subsequent epics build on.

**FRs covered:** FR-1.1, FR-1.2, FR-1.3, FR-1.4, FR-2.1, FR-5.1, FR-5.2

---

### Story 1.1: Define frente domain mapping in goals.yaml

As a Life OS user,
I want frentes declared in `goals.yaml` with their domain assignments,
So that all skills can read which frentes belong to `work` vs `personal`.

**Acceptance Criteria:**

**Given** `goals.yaml` exists in `{vault}/config/`
**When** I add a `frentes:` section with entries (e.g., `- name: portal-del-cliente, domain: work`)
**Then** the section is parseable by skills reading `goals.yaml`
**And** any frente not listed is treated as unmapped — skills degrade gracefully (NFR-2)
**And** the schema supports exactly two domains: `personal` and `work`

---

### Story 1.2: Declare active commitments in CRITICAL_FACTS.md

As a Life OS user,
I want a standardized `## Compromisos activos` section in CRITICAL_FACTS.md,
So that `morning` and `close` can reliably parse my active behavioral commitments.

**Acceptance Criteria:**

**Given** CRITICAL_FACTS.md exists in `08 Resources/`
**When** I add `## Compromisos activos` with entries like `- [C1] Cerrar a las 18:00`
**Then** skills can parse each commitment's ID (`[C1]`) and description text
**And** the system supports 0 to 3 commitments (system ceiling per FR-2.1)
**And** if the section is absent, skills that read it proceed without error (graceful degradation)

---

### Story 1.3: Today skill groups tasks by frente

As a Life OS user,
I want the `today` skill to group my tasks by frente within Work and Personal sections,
So that I can see at a glance which area of focus each task belongs to.

**Acceptance Criteria:**

**Given** the daily note contains tasks with `frente:` tags on some and none on others
**When** I run `/today`
**Then** tasks with a `frente:` tag are grouped under their frente label within each domain section
**And** tasks without a `frente:` tag appear under a "Sin frente" subgroup
**And** a one-liner "Frentes activos hoy: [list]" appears above the task sections
**And** if no tasks have frente tags, the one-liner is omitted (full graceful degradation, NFR-2)
**And** the existing task priority ordering is preserved within each frente group
