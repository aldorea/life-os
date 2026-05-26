---
title: Life OS Cadence System
status: final
created: 2026-05-24
updated: 2026-05-25
---

## Overview

Life OS already captures tasks, syncs external sources, and runs daily rituals. What it lacks is a **cadence layer**: a system that connects daily execution to the domains and commitments that actually matter to Sito, surfaces drift before it compounds, and does all of this without adding friction or requiring new skills to invoke.

This PRD defines requirements for enhancing the existing `morning`, `close`, and `today` rituals to serve as a complete cadence operating system — covering daily, weekly, and monthly horizons — across Sito's two life domains (Personal and Work).

---

## Problem Statement

The current ritual layer (morning/close/today) operates at task and goal granularity. It answers "what should I do today?" but not "is the system working?" — it has no visibility into domain health, no mechanism for tracking behavioral commitments (as distinct from tasks), and no weekly or monthly review rhythm. The result: the system can be perfectly consistent at the task level while quietly drifting at the frente or commitment level.

Symptoms:
- No way to know if a frente (e.g. Salud, Casa) is on track without manual review
- Behavioral commitments ("close at 18:00", "phone down with partner") are invisible to the system — they exist in CRITICAL_FACTS.md but are never surfaced or checked
- Weekly drift accumulates silently — there is no weekly review rhythm
- Known blockers (e.g. funding milestones unknown, Portal del Cliente team TBD) have no place in the daily system

---

## Users & Context

**Single user**: Sito — professional managing two concurrent high-autonomy work contexts (Orbitant consulting + Previene startup) and six personal frentes.

**Two domains** (treated as separate cognitive zones):

| Domain | Frentes | Notes |
|--------|---------|-------|
| Personal | Organización (foundational), Casa, Salud, Relación, Aprendizaje IA, Presencia | Casa has hard deadline (June 2026 move); training plan and IA curriculum are undefined — frente exists but path is fuzzy |
| Work | Orbitant / Portal del Cliente, Previene MVP | Portal del Cliente referente role is mid-transition (not formally assumed); Previene MVP horizon depends on funding milestone not yet known — treat both as "transición" frentes |

**Operating heuristic**: Organización is the constraint that conditions everything else. If the cadence system fails, it fails here first.

---

## Goals & Success Metrics

**North star**: *"Hago las cosas que me propongo."* The system succeeds when Sito executes on his commitments — not when it merely exists.

| Metric | Target |
|--------|--------|
| Daily ritual adherence | morning + close run ≥5 days/week |
| Commitment visibility | Every active commitment surfaced in morning or close |
| Weekly review coverage | Every frente reviewed at least once per week |
| Frente drift detection | No frente goes >7 days without at least one qualifying task completed |
| Friction | No new top-level skills required; all cadence within existing ritual surface |

**Counter-metric**: If average ritual run time increases by more than 3 minutes, the implementation has over-engineered.

---

## Features & Requirements

### F1 — Frente Model (Convention, Not New Data Structure)

Frentes are grouping labels applied to existing tasks and goals — not a new vault schema.

**FR-1.1** Each task in the backlog MAY carry a `frente:` tag (e.g. `frente: salud`, `frente: portal-del-cliente`). This is opt-in; untagged tasks appear under "Sin frente."

**FR-1.2** Goals in `goals.yaml` MAY carry a `frente:` field. The scoring algorithm in `today` is unaffected; the field is metadata only.

**FR-1.3** The two top-level domains are `personal` and `work`. Frentes belong to one domain. This mapping is declared in a new `frentes:` section in `goals.yaml`.

**FR-1.4** No migration of existing tasks is required at launch. Frente tagging is adopted incrementally.

---

### F2 — Commitment Tracking

Behavioral commitments (time boundaries, presence habits, focus block protection) are distinct from tasks. They have no due date and no "done" state — they are daily recurring promises.

**FR-2.1** Active commitments shall be declared in `CRITICAL_FACTS.md` under a `## Compromisos activos` section. Format: one commitment per line with a short ID, e.g.:
```
- [C1] Cerrar a las 18:00
- [C2] Teléfono abajo con la pareja
- [C3] Bloque de foco protegido
```
The Relación frente is the primary driver for commitment tracking — behavioral anchors like "phone down during partner time" and "quality time scheduled proactively" are canonical examples of commitments that belong here, not in the backlog.

**FR-2.2** `morning` shall surface the active commitments list at the start of each day ("Hoy me comprometí a: …"). No user action required — display only.

**FR-2.3** `close` shall prompt one binary check-in per commitment: "¿Lo cumpliste?" (sí / no / parcial). Responses are recorded in the daily note's Cierre section.

**FR-2.4** When ≥2 consecutive "no" responses are recorded for the same commitment, `morning` shall surface a low-friction alert the following day ("Este compromiso lleva 2 días sin cumplirse — ¿sigue activo?"). The alert is informational, not blocking.

---

### F3 — Morning Enhancements

The existing `morning` flow (sync → inbox → daily note) is preserved. The following additions are layered on top.

**FR-3.1 — Domain health snapshot (daily)**
After the daily note is generated, `morning` shall append a compact frente snapshot showing:
- For each active frente: last task completed + days since last activity (computed from backlog completed tasks filtered by `frente:` tag)

Display format: a 2-column table (frente, last activity). Frentes with >3 days inactivity are flagged with ⚠️.

**FR-3.2 — Active commitments display (daily)**
Immediately after the frente snapshot, the system shall display the active commitments list (from FR-2.1). Read-only.

**FR-3.3 — Blocker surfacing (daily)**
When `CRITICAL_FACTS.md` or a `blockers:` section in `connectors.yaml` contains active blockers, `morning` shall surface them as one line per blocker: "🔴 [BLOCKER] Portal del Cliente: team composition unknown."

**FR-3.4 — Weekly review section (Monday mornings only)**
When `morning` detects the current day is Monday, it shall prepend a **Weekly Review** section to the daily note before the standard Foco section. Structure:

```
## Weekly Review — Semana del DD/MM

### Frentes
| Frente | Estado | Nota |
|--------|--------|------|
| Organización | ... | ... |
| Casa | ... | ... |
...

### Compromisos (semana pasada)
- [C1] Cumplido X/5 días
- [C2] Cumplido X/5 días

### Foco de la semana
[single sentence — what matters most this week]
```

`morning` auto-populates the frente table (last activity per frente) and commitment counts (sí/no/parcial from the past week's daily notes). The user fills in "Estado" (one word: verde/amarillo/rojo/transición) and "Foco de la semana" (one sentence). "Transición" is a valid Estado for frentes in active flux — e.g. Portal del Cliente (role not formally assumed) or Previene (funding horizon unknown); it signals "moving, not stuck" without forcing a health judgment.

**FR-3.5 — Monthly horizon review (1st of each month only)**
When `morning` detects it is the 1st of a month, it shall prepend a **Monthly Horizon** section. Structure:

```
## Horizonte — Mes de [MES]

### ¿Sigo yendo hacia donde quiero ir?
(freetext — filled by user)

### Frentes: progreso último mes
[auto-populated from commitments + frente activity]

### Ajustes
(freetext — filled by user)
```

This section replaces the Weekly Review on months where the 1st is a Monday; otherwise both coexist.

---

### F4 — Close Enhancements

The existing `close` flow (reflection questions → task decisions → wiki log → commit) is preserved.

**FR-4.1 — Commitment check-in**
Before the existing reflection questions, `close` shall run one round of commitment check-ins (FR-2.3). Each active commitment gets a one-line prompt; the user answers sí/no/parcial. Maximum 3 active commitments total (system ceiling, not display rotation).

**FR-4.2 — Presencia check-in (daily)**
`close` shall include one fixed reflection question: *"¿Estuviste presente hoy?"* (sí / no / parcial). The response is stored in the Cierre section alongside commitment responses. This prompt serves both the Presencia frente and the Relación frente; it is not skippable even if other prompts are.

**FR-4.3 — Frente note (weekly, on Fridays only)**
When `close` detects it is Friday, it shall add one optional prompt: "Una frase sobre cómo fue la semana para cada frente activo." The user may skip. If answered, the response is stored in the daily note's Cierre section and used to pre-populate next Monday's weekly review frente table (FR-3.4).

**FR-4.4 — Commitment streak tracking**
`close` shall record commitment responses in the daily note. A lightweight aggregation reads the last 7 days of daily notes to compute streaks for FR-2.4.

---

### F5 — Today Enhancements

`today` is the lightweight, read-only daily view (no external API calls). Changes are minimal to preserve its speed.

**FR-5.1 — Frente filter on task sections**
The existing "Trabajo" and "Personal" task sections in the daily note shall group tasks by frente within each domain. Untagged tasks appear under "Sin frente."

**FR-5.2 — Active frentes summary**
A compact one-liner shall appear above the task sections: "Frentes activos hoy: [list of frentes that have tasks scheduled today]." If none, the line is omitted.

---

## Non-Functional Requirements

**NFR-1 — No new top-level skills.** All cadence behavior lives inside `morning`, `close`, and `today`. No `/weekly`, `/review`, or `/frentes` skill.

**NFR-2 — Graceful degradation.** When frente tags are absent, rituals run without the frente layer — same behavior as today. No hard dependency on tagging completeness.

**NFR-3 — Time budget.** Monday morning (with weekly review) must complete in under 10 minutes. All other days must stay within 2 minutes of the current baseline.

**NFR-4 — No calendar writes.** The system reads calendar data; it never writes.

**NFR-5 — Vault-only persistence.** All cadence data (commitment responses, frente status) lives in daily notes and CRITICAL_FACTS.md. No new vault files or schemas required.

**NFR-6 — Incremental adoption.** Phase 1 launches commitments + Monday weekly review. Frente tagging and domain health snapshot follow once the commitment flow is stable.

---

## Out of Scope

- Automated enforcement of any commitment (the system surfaces, never enforces)
- New standalone skills (see NFR-1)
- Writing to calendar
- Notifications or push alerts outside the CLI session
- Multi-user support
- Frente progress percentages (qualitative status only — avoids false precision)
- A separate "blocker management" skill or tracker

---

## Open Questions

| # | Question | Owner | Urgency |
|---|----------|-------|---------|
| OQ-1 | Work brief has two known blockers (team composition, funding milestones). Should these be stored in CRITICAL_FACTS.md or a dedicated blockers section? | Sito | Phase 2 |
| OQ-2 | Should commitment responses feed back into the `wiki/log.md` for long-term pattern analysis? | Sito | Post-launch |
| OQ-3 | Salud frente: goal is -8 kg + aerobic capacity, but no training plan exists. Should the system prompt for a plan definition, or just track what happens? | Sito | Phase 2 |
| OQ-4 | Aprendizaje IA: destination clear (agents, workflows, orchestration) but no curriculum. Should morning surface a "define next learning step" prompt until a path exists? | Sito | Phase 2 |
| OQ-5 | Judo re-entry: currently on hold. Should it appear as a frente (with Estado: transición) or be excluded until the condition is met? | Sito | Phase 2 |
