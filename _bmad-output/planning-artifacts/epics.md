---
stepsCompleted: ["step-01", "step-02", "step-03"]
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

---

## Epic 2: Morning Situational Awareness

Sito starts each morning with a clear picture: frente health snapshot, active commitments, active blockers, and on Mondays a full Weekly Review with frente status and commitment adherence counts. Context before planning.

**FRs covered:** FR-2.2, FR-2.4, FR-3.1, FR-3.2, FR-3.3, FR-3.4, FR-3.5

---

### Story 2.1: Frente health snapshot in morning

As a Life OS user,
I want `morning` to show me a compact frente health snapshot at the start of each day,
So that I immediately know which areas of focus have gone stale without needing to search.

**Acceptance Criteria:**

**Given** the backlog (`01 Backlog.md`) contains tasks with `frente:` tags
**When** I run `/morning`
**Then** morning reads all tasks in the backlog and groups them by `frente:` tag to find the most recent completion or modification per frente
**And** a 2-column table is shown: `Frente | Última actividad` (task name + "X días" since last activity)
**And** any frente with >3 days since last activity is flagged with ⚠️ in the frente column

**Given** some tasks carry `frente:` tags and others do not
**When** the snapshot is computed
**Then** only tagged frentes appear in the table; untagged tasks are ignored for this section

**Given** no tasks in the backlog carry any `frente:` tag
**When** morning runs
**Then** the frente snapshot section is silently omitted — no error, no empty table shown

*Covers: FR-3.1*

---

### Story 2.2: Active commitments display and streak alert in morning

As a Life OS user,
I want `morning` to show me my active commitments and warn me when I'm repeatedly missing one,
So that I stay accountable without needing to consult CRITICAL_FACTS.md manually each day.

**Acceptance Criteria:**

**Given** `CRITICAL_FACTS.md` contains a `## Compromisos activos` section with one or more entries
**When** I run `/morning`
**Then** morning displays immediately after the frente snapshot: "Hoy me comprometí a:" followed by each commitment on its own line (e.g., "[C1] Cerrar a las 18:00")

**Given** a commitment (e.g., `[C1]`) has at least 2 consecutive "no" responses recorded in the last 7 days of Cierre sections in `03 Daily/`
**When** morning runs
**Then** an informational alert is shown next to that commitment: "⚠️ Llevas 2+ días sin cumplirlo."
**And** the alert is non-blocking — morning continues normally after displaying it

**Given** morning reads the last 7 days of daily notes to check Cierre streak data
**When** fewer than 2 daily notes with Cierre sections exist
**Then** no streak alert is shown (insufficient data — degrade gracefully, no error)

**Given** the `## Compromisos activos` section is absent from CRITICAL_FACTS.md
**When** morning runs
**Then** the commitments display section is silently omitted — no error shown

*Covers: FR-2.2, FR-2.4, FR-3.2*

---

### Story 2.3: Blocker surfacing in morning

As a Life OS user,
I want `morning` to surface any active blockers from CRITICAL_FACTS.md at the start of my day,
So that I don't start planning without knowing what's actively blocking my work.

**Acceptance Criteria:**

**Given** `CRITICAL_FACTS.md` contains a `blockers:` section with one or more active blocker entries
**When** I run `/morning`
**Then** each blocker is displayed on its own line in the format: "🔴 [BLOCKER] {description}"
**And** the blockers section appears after the commitments display and before any planning content

**Given** `connectors.yaml` contains a `blockers:` section (as an alternative/additional blocker source)
**When** morning reads for blockers
**Then** blockers from both CRITICAL_FACTS.md and connectors.yaml are merged and displayed together (deduplicated by description)

**Given** no blockers exist in either CRITICAL_FACTS.md or connectors.yaml
**When** morning runs
**Then** the blockers section is silently omitted — no message, no empty list

**Given** CRITICAL_FACTS.md is unreadable or the `obsidian` CLI returns an error
**When** morning attempts to read blockers
**Then** morning continues without the blockers section and logs a one-line warning in the daily note header: "[blockers: unable to read CRITICAL_FACTS.md]"

*Covers: FR-3.3*

---

### Story 2.4: Monday Weekly Review and Monthly Horizon in morning

As a Life OS user,
I want `morning` to automatically prepend a Weekly Review section on Mondays and a Monthly Horizon section on the 1st of the month,
So that I have structured retrospective and planning context built into my daily note without manual setup.

**Acceptance Criteria:**

**Given** today is a Monday
**When** I run `/morning`
**Then** a Weekly Review section is prepended to the daily note BEFORE the Foco section
**And** the section contains: an auto-populated frente table (last activity per frente from backlog) and a commitment counts table (sí/no/parcial per commitment ID, counted from the past 7 days of Cierre sections)
**And** two fields are left blank for manual entry: "Estado de la semana:" (accepted values: verde/amarillo/rojo/transición) and "Foco de la semana:"

**Given** today is the 1st of the month but NOT a Monday
**When** morning runs
**Then** only a Monthly Horizon section is prepended to the daily note
**And** the section contains: auto-populated frente activity summary for the prior month and commitment adherence counts for the prior month

**Given** today is the 1st of the month AND a Monday
**When** morning runs
**Then** both sections appear: Monthly Horizon first (above), then Weekly Review below it
**And** neither section overwrites the other

**Given** fewer than 3 days of Cierre sections exist across the prior week's daily notes
**When** the commitment counts table is computed for the Weekly Review
**Then** the commitment counts table shows "Sin datos" for all commitments rather than an error

**Given** the daily note for today already exists and already contains a Weekly Review section
**When** morning runs again on the same Monday
**Then** morning skips prepending a new Weekly Review section (idempotent — no duplicate sections)

*Covers: FR-3.4, FR-3.5*

---

## Epic 3: Daily Planning Loop (Morning)

Sito confirms his task list each morning (clarification step), gets a weekly plan every Monday written to `04 Weekly/`, a focused daily brief in the `Foco` section, and a ready-to-use standup summary — all pre-populated, no recall required.

**FRs covered:** FR-P1.1, FR-P1.2, FR-P1.3, FR-P2.1, FR-P2.2, FR-P2.3, FR-P2.4, FR-P2.5, FR-P3.1, FR-P3.2, FR-P3.3, FR-P4.1, FR-P4.2, FR-P4.3

---

### Story 3.1: Clarification step before planning

As a Life OS user,
I want `morning` to show me the current task list and ask if it's complete before generating any plan,
So that the daily brief and weekly plan are always built on a confirmed, accurate list — not stale data.

**Acceptance Criteria:**

**Given** the backlog (`01 Backlog.md`) and the last Jira sync data are available
**When** I run `/morning` (any day of the week)
**Then** morning presents the combined task list (backlog tasks + Jira tickets from last sync) as a numbered list
**And** then asks: "Before we plan — is this list complete? Anything missing, anything that shouldn't be here?"

**Given** I respond to the clarification prompt by adding, removing, or marking items as irrelevant
**When** the clarification exchange is complete
**Then** the confirmed list (after my edits) becomes the sole planning input for that morning's brief and, on Mondays, for the weekly plan
**And** any removed or irrelevant items are excluded from planning output for this session only (not permanently deleted from backlog)

**Given** I skip the clarification step (e.g., respond "skip" or press enter without changes)
**When** morning proceeds
**Then** it uses the existing list unchanged
**And** writes `[clarification skipped]` in the daily note's header block
**And** continues normally to the next planning step

**Given** Jira MCP is unavailable (both instances down)
**When** morning presents the task list for clarification
**Then** it shows only the backlog tasks and prepends a note: "[Jira no disponible — mostrando solo backlog]"
**And** clarification step still runs with the available data

*Covers: FR-P1.1, FR-P1.2, FR-P1.3*

---

### Story 3.2: Weekly plan generation on Monday mornings

As a Life OS user,
I want `morning` to generate a Mon–Fri task allocation plan every Monday and save it to my weekly note,
So that I have a concrete roadmap for the week that accounts for my calendar load and priorities.

**Acceptance Criteria:**

**Given** today is Monday (or the first morning run of the week) and the clarification step is complete
**When** morning proceeds to planning
**Then** it reads calendar events for the full week via `icalbuddy` and reads goal priorities from `goals.yaml`
**And** proposes a `Day | Tasks | Notes` table allocating confirmed tasks Mon–Fri based on: (a) calendar availability, (b) goal priority, (c) explicit task deadlines

**Given** a day in the week has ≥5 hours of calendar events (≥80% of an 8-hour workday)
**When** morning allocates tasks
**Then** no tasks are assigned to that day in the weekly plan table
**And** a note is added in the Notes column for that day: "Día lleno — sin tareas asignadas"

**Given** the weekly plan proposal is presented for approval
**When** I confirm it
**Then** morning writes the plan to `04 Weekly/week-{YYYY-WW}.md` with frontmatter: `week: {YYYY-WW}`, `status: confirmed`, `created: {today}`
**And** if I reject or edit it, morning writes the amended version with `status: confirmed` only after re-confirmation

**Given** a confirmed weekly note for the current week already exists at `04 Weekly/week-{YYYY-WW}.md`
**When** morning runs on Monday
**Then** morning skips plan generation entirely and displays the existing confirmed plan with a note: "Ya existe un plan confirmado para esta semana."

**Given** today is not Monday (Tuesday through Sunday)
**When** morning runs
**Then** no weekly plan generation or proposal occurs — morning reads the existing weekly note for the daily slice only (Story 3.3)

*Covers: FR-P2.1, FR-P2.2, FR-P2.3, FR-P2.4, FR-P2.5*

---

### Story 3.3: Daily brief from weekly plan

As a Life OS user,
I want `morning` to extract my tasks for today from the confirmed weekly plan and write a focused daily brief into the Foco section,
So that each day I start with a clear, pre-populated list of what to do — no re-planning required.

**Acceptance Criteria:**

**Given** a confirmed weekly note exists for the current week at `04 Weekly/week-{YYYY-WW}.md`
**When** morning extracts today's slice
**Then** the Foco section of the daily note is written with: tasks assigned to today from the weekly plan, the first 2-hour unblocked calendar window as "Bloque de foco sugerido: {HH:MM–HH:MM}", and any tasks with deadlines due today or tomorrow

**Given** there are no tasks assigned to today in the weekly plan (e.g., it was a full-calendar day)
**When** morning writes the Foco section
**Then** the section shows "Sin tareas planificadas para hoy." and still includes the suggested focus block if one exists in the calendar

**Given** no confirmed weekly note exists for the current week
**When** morning runs (any day)
**Then** it generates a day-scoped plan: tasks ranked by priority from the confirmed list (Story 3.1 output)
**And** prepends to the Foco section: "No hay plan semanal confirmado — mostrando plan de día."
**And** still calculates and includes the suggested focus block from calendar data

**Given** `icalbuddy` returns no calendar data or fails
**When** morning computes the focus block suggestion
**Then** the focus block line is omitted from the Foco section (degraded gracefully — tasks still shown)

*Covers: FR-P3.1, FR-P3.2, FR-P3.3*

---

### Story 3.4: Standup summary pre-population

As a Life OS user,
I want `morning` to auto-populate a Standup section in the daily note using yesterday's Cierre and today's brief,
So that I can copy my standup directly from my daily note without recalling anything from memory.

**Acceptance Criteria:**

**Given** morning has completed the daily brief (Story 3.3) and can read prior day's daily note
**When** the daily note is written
**Then** it includes a `## Standup` section with three pre-populated fields:
- `Done yesterday:` — populated from the prior day's Cierre section activity log
- `Planned today:` — populated from today's daily brief (Foco section tasks)
- `Blockers:` — populated from active blockers in CRITICAL_FACTS.md (same source as Story 2.3)

**Given** the prior day's daily note exists but has no Cierre section
**When** morning writes the Standup section
**Then** `Done yesterday:` shows: "No hay cierre del día anterior."
**And** the remaining two fields are still populated normally

**Given** no active blockers exist in CRITICAL_FACTS.md
**When** morning writes the Standup section
**Then** `Blockers:` shows: "Sin bloqueos activos."

**Given** morning runs and the daily note already contains a Standup section
**When** morning attempts to write the standup
**Then** morning does not overwrite the existing Standup section (idempotent — preserves any manual edits)

*Covers: FR-P4.1, FR-P4.2, FR-P4.3*

---

## Epic 4: Close Ritual & Activity Logging

Sito closes each day with commitment check-ins (sí/no/parcial), a non-skippable presence check, an inferred activity log built from Jira + calendar + Granola (confirmed, not recalled), and disruptions auto-classified and stored in the Cierre section.

**FRs covered:** FR-2.3, FR-4.1, FR-4.2, FR-4.3, FR-4.4, FR-P5.1, FR-P5.2, FR-P5.3, FR-P5.4, FR-P5.5, FR-P6.1, FR-P6.2, FR-P6.3, FR-P6.4, FR-P6.5

---

### Story 4.1: Commitment check-ins and Presencia in close

As a Life OS user,
I want `close` to run me through my commitment check-ins and a presence check before reflection,
So that commitment adherence is captured consistently every day and available for streak tracking.

**Acceptance Criteria:**

**Given** CRITICAL_FACTS.md contains a `## Compromisos activos` section with 1–3 commitments
**When** I run `/close`
**Then** close prompts one check-in per commitment BEFORE any reflection questions, in the format: "¿Lo cumpliste? [{ID}] {description} (sí/no/parcial)"
**And** after all commitment check-ins, shows the non-skippable prompt: "¿Estuviste presente hoy? (sí/no/parcial)"
**And** all responses are written to the Cierre section of today's daily note

**Given** close has written today's commitment responses
**When** the session ends
**Then** close reads the last 7 days of daily note Cierre sections to compute per-commitment streaks
**And** streak data is available in-memory for the current session (consumed by morning's Story 2.2 on next run via daily note data)

**Given** no `## Compromisos activos` section exists in CRITICAL_FACTS.md
**When** close runs
**Then** commitment check-ins are skipped entirely
**And** only the Presencia check-in runs: "¿Estuviste presente hoy? (sí/no/parcial)"
**And** the Presencia response is still recorded in the Cierre section

**Given** the user answers Presencia as "no"
**When** the response is recorded
**Then** the Cierre section records exactly: `presencia: no` (no additional prompt, no judgment)

*Covers: FR-2.3, FR-4.1, FR-4.2, FR-4.4*

---

### Story 4.2: Friday frente reflection in close

As a Life OS user,
I want `close` to ask me for a brief frente reflection on Fridays,
So that my weekly frente status is captured in my own words and available for the next Monday's Weekly Review.

**Acceptance Criteria:**

**Given** today is a Friday
**When** I run `/close` and the commitment check-ins and Presencia check are complete
**Then** close shows one additional optional prompt: "Una frase sobre cómo fue la semana para cada frente activo"
**And** the active frentes are listed (from backlog tags / goals.yaml) to make it easy to respond per frente

**Given** I answer the Friday frente reflection prompt
**When** my response is recorded
**Then** it is stored in the Cierre section of today's daily note under a `frente-reflection:` field
**And** on the following Monday, morning reads this field to pre-populate the Estado column in the Weekly Review frente table

**Given** I skip the Friday frente reflection (respond "skip" or leave blank)
**When** close continues
**Then** no reflection is stored — close proceeds normally without error
**And** the following Monday's Weekly Review Estado column will show the blank-fill UI (no pre-population)

**Given** today is NOT a Friday
**When** close runs
**Then** the Friday frente reflection prompt is not shown at all

*Covers: FR-4.3*

---

### Story 4.3: Activity inference in close

As a Life OS user,
I want `close` to infer what I did today from Jira, calendar, and Granola before asking me anything,
So that I confirm rather than recall — minimizing cognitive effort at end of day.

**Acceptance Criteria:**

**Given** Jira MCP data (afianza and/or previene), calendar events (via `icalbuddy`), and Granola notes are available for today
**When** I run `/close` (after commitment check-ins)
**Then** close reads in priority order: (1) Jira ticket state changes for today, (2) calendar events attended today, (3) Granola notes created today
**And** presents the inferred summary BEFORE asking anything: "Parece que hoy: {inferred list}. ¿Hay algo que añadir o corregir?"

**Given** I confirm the inferred activity list without changes
**When** the confirmation is received
**Then** the confirmed activity log is written to the Cierre section of the daily note verbatim

**Given** I correct the inferred activity list (add items, remove items, or edit descriptions)
**When** I submit the corrections
**Then** the corrected list is written to the Cierre section — the user's version, not the inferred version

**Given** Jira MCP is unavailable (both instances) but calendar data is available
**When** close runs activity inference
**Then** it proceeds using calendar events and Granola notes only
**And** notes in the Cierre section: "[Jira no disponible — inferencia basada en calendario y notas]"
**And** the prompt still fires: "Parece que hoy: {calendar-only list}. ¿Hay algo que añadir o corregir?"

**Given** ALL inference sources (Jira, calendar, Granola) return empty or are unavailable
**When** close attempts inference
**Then** close shows: "No encontré actividad registrada para hoy. ¿Qué hiciste?"
**And** accepts free-text input from the user as the activity log
**And** writes the free-text response to the Cierre section

*Covers: FR-P5.1, FR-P5.2, FR-P5.3, FR-P5.4, FR-P5.5*

---

### Story 4.4: Disruption detection and tagging

As a Life OS user,
I want `close` to identify unplanned events from my day, auto-classify them by type, and store them in the Cierre section,
So that disruption patterns can be tracked over time without requiring manual logging.

**Acceptance Criteria:**

**Given** close has confirmed today's activity log (Story 4.3) and has access to the morning's daily brief (Foco section)
**When** close compares today's activity against the morning's plan
**Then** it identifies candidate disruptions as: (a) calendar events not present in the morning's daily brief, and (b) tasks from the weekly plan for today that show no progress in the activity log

**Given** candidate disruptions are identified
**When** close auto-classifies each one
**Then** each disruption is assigned a type from the fixed taxonomy: `reunion-extra` | `incidente-tecnico` | `peticion-cliente` | `bloqueo-externo` | `otro`
**And** classification signals used are: calendar event title/type for calendar events, Jira issue type and labels for Jira items, and Granola note content for meeting notes

**Given** auto-classification confidence is HIGH for a disruption
**When** close presents it to the user
**Then** it shows: "Detecté una disrupción: '{description}' → tipo: {type}. ¿Correcto? (sí / corregir)"

**Given** auto-classification confidence is LOW for a disruption
**When** close presents it to the user
**Then** it shows the full taxonomy table and asks: "¿Qué tipo de disrupción fue '{description}'?" with the 5 options listed
**And** waits for the user to select one before continuing

**Given** the user confirms or corrects the disruption type
**When** the response is recorded
**Then** each stored disruption record in the Cierre section includes: `date`, `type` (one of the five fixed values), `description` (one sentence), and `displaced-task` (the weekly plan task it displaced, or `null` if none)

**Given** no unplanned events are detected (today matched the plan exactly)
**When** close completes the comparison
**Then** the disruption detection section is silently omitted — no message shown

*Covers: FR-P6.1, FR-P6.2, FR-P6.3, FR-P6.4, FR-P6.5*

---

## Epic 5: Weekly Intelligence & Pattern Detection

The loop closes: the Monday Weekly Review shows plan adherence, tasks that didn't happen (with disruption types), a disruption summary, and proposed adjustments. After 4 weeks of data, morning surfaces recurring patterns as read-only observations.

**FRs covered:** FR-P7.1, FR-P7.2, FR-P7.3, FR-P8.1, FR-P8.2, FR-P8.3, FR-P8.4, FR-P8.5

---

### Story 5.1: Weekly Review extension with plan adherence and disruption summary

As a Life OS user,
I want the Monday Weekly Review to include plan adherence stats, a list of tasks that didn't happen, a disruption breakdown, and a system-generated adjustment suggestion,
So that I can understand my planning accuracy and disruption load from the prior week in one place.

**Acceptance Criteria:**

**Given** today is Monday and the Weekly Review section is being built (Story 2.4)
**When** morning computes the Weekly Review extension
**Then** four new subsections are appended AFTER the existing frente table and compromisos section (additive, no existing content modified):
1. **Plan Adherence**: "Semana pasada: completaste X de Y tareas planificadas (Z%)" — computed only from tasks originally in the prior week's confirmed weekly note
2. **Tasks That Didn't Happen**: bulleted list of unfinished confirmed-plan tasks, each annotated with the disruption type(s) that displaced them (from Cierre disruption records)
3. **Disruption Summary**: count per disruption type (e.g., "reunion-extra: 3, peticion-cliente: 1, otro: 0")
4. **Proposed Adjustments**: 1–2 system-generated sentences based on the disruption pattern (e.g., "Los miércoles tuviste 2 reuniones no planificadas — considera dejar ese día con menos tareas")

**Given** fewer than 3 days of Cierre sections exist in the prior week's daily notes
**When** morning attempts to compute any of the four subsections
**Then** all four subsections show the same message: "Datos insuficientes para calcular adherencia — menos de 3 cierres registrados la semana pasada."
**And** the existing Weekly Review content (frente table, compromisos) is unaffected

**Given** the prior week had a confirmed weekly note but no disruption records in any Cierre section
**When** the Disruption Summary is computed
**Then** all five disruption type counts show 0 (e.g., "reunion-extra: 0, incidente-tecnico: 0, ...")
**And** Proposed Adjustments shows: "Sin disrupciones registradas la semana pasada."

**Given** the Proposed Adjustments are generated
**When** they are written to the daily note
**Then** they are strictly read-only informational text — no links, no action items, no tasks added to the backlog
**And** the Weekly Review extension is idempotent: if the Monday daily note already contains the four subsections, morning does not re-compute or overwrite them

*Covers: FR-P7.1, FR-P7.2, FR-P7.3*

---

### Story 5.2: Recurring pattern detection in morning

As a Life OS user,
I want `morning` to surface recurring disruption and inactivity patterns as factual observations on Monday mornings once enough data exists,
So that I can make informed decisions about my planning without the system making changes on my behalf.

**Acceptance Criteria:**

**Given** fewer than 4 full weeks of Cierre logs exist in `03 Daily/` (counting weeks where ≥3 Cierre entries are present)
**When** morning runs (any day, including Monday)
**Then** pattern detection runs silently — no Patterns section is shown, no message about missing data

**Given** 4 or more full weeks of Cierre logs exist
**When** morning runs on a Monday
**Then** a `## Patrones detectados` section is shown BEFORE the weekly plan (after blockers, before clarification step)
**And** it contains only read-only factual observations, formatted as bullet points

**Given** pattern detection is computing day-of-week disruption frequency
**When** at least 4 weeks of data are available
**Then** if a specific day averages ≥1.0 disruptions/week across those weeks, it is surfaced as: "Los {día} tienes de media {N} disrupciones por semana"
**And** no planning change is made — observation only

**Given** pattern detection is checking for recurring disruption types
**When** a disruption type appears in ≥2 consecutive weeks
**Then** it is surfaced as: "{tipo} ha aparecido {N} semanas consecutivas"

**Given** pattern detection is checking for frente inactivity
**When** a frente has shown >3-day gaps consistently across ≥3 of the 4+ weeks analyzed
**Then** it is surfaced as: "El frente '{frente}' ha tenido inactividad recurrente (>3 días) en las últimas {N} semanas"

**Given** the Patterns section is generated
**When** it is written to the daily note or displayed in the terminal
**Then** no automated changes are made to the weekly plan, backlog, or any other vault file
**And** the section contains a footer line: "Estas son observaciones — no se ha modificado nada automáticamente."

*Covers: FR-P8.1, FR-P8.2, FR-P8.3, FR-P8.4, FR-P8.5*
