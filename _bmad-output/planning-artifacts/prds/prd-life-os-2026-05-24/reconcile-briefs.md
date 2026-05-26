# PRD–Briefs Reconciliation: Life OS Cadence System

**Date**: 2026-05-25  
**PRD**: `prd.md` (2026-05-24)  
**Source Briefs**:
- Personal Scope Brief (2026-05-22)
- Work Scope Brief (2026-05-19)

---

## Executive Summary

The PRD successfully captures the core cadence mechanics (frentes, commitments, weekly/monthly reviews) but **silently drops qualitative intent and transition-state nuance** from the briefs. Three areas of material underrepresentation:

1. **Presencia as a measurable practice** (Personal Brief) — not surfaced in rituals
2. **Relationship frente specifics** (Personal Brief) — phone-down commitment absent; quality time scheduling not addressed
3. **Work domain transition states** (Work Brief) — role assumption, client process flux, and team-composition risk not reflected in frente definitions or success signals

---

## Gap Analysis: Personal Domain

### GAP 1: Presencia Practice Omitted

**Brief statement:**
> "In a week, this shows up as a brief end-of-day check-in — not a separate ritual slot, but a question appended to the close ritual: *was I present today?*"
> — Personal Brief, Presencia section

**PRD coverage**: None. Presencia frente exists in the domain but has no corresponding FR.

**Impact**: The system defines frentes (Salud, Relación, etc.) but has no mechanism to surface or track the one habit (Presencia) the brief explicitly recommends. This is a silent drop of intent.

**Recommendation**: Add to F3 or F4:
- **FR-3.6** or **FR-4.4**: Append a presencia check-in to the `close` ritual: "¿Estuviste presente hoy?" (sí/no/parcial). Auto-populate in the Cierre section. This feeds into the relationship frente health signal.

---

### GAP 2: Relación Frente Lacks Specificity

**Brief statement:**
> "Regular quality time without the phone"  
> "Proactive plans oriented around what motivates her [partner], not defaulting to what's easy"  
> "Shared weekly time is not currently calendared — needs to be created deliberately"  
> — Personal Brief, Relación section

**PRD coverage**: Relación is listed as an active frente but has:
- No commitment tied to it (phone-down, quality time, weekly calendar block)
- No weekly review guidance specific to relationship health
- No success signal that distinguishes "ongoing" from "thriving"

**Impact**: The frente exists but lacks the behavioral anchors the brief identifies. The cadence system can report "Relación is active" without surfacing whether the core commitments (phone down, proactive planning, calendared time) are being honored.

**Recommendation**: 
- Define at least two commitments for Relación: phone-down commitment (linked to presencia) and scheduled quality time
- FR-4.2 (Frente note on Friday) should explicitly prompt: "Relación: how consistent was quality time this week?"

---

### GAP 3: Fuzzy Personal Gaps Not Acknowledged

**Brief gaps flagged** (Known Gaps section):
- Judo re-entry criteria undefined `[fuzzy]`
- No specific training plan `[fuzzy]`
- IA learning path undefined `[fuzzy]`

**PRD coverage**: These gaps are not listed in the Open Questions section.

**Impact**: The PRD's success metrics assume Salud and Aprendizaje IA goals are stable, but the brief flags them as undefined. If the cadence system is tracking progress on undefined targets, it will report "no drift" while the person is actually stalled.

**Recommendation**: 
Add to Open Questions:
- OQ-1: Salud frente has a clear goal (-8 kg, run without breathlessness) but no structured training plan. Should weekly review surface this ambiguity, or is plan-refinement outside the cadence system's scope?
- OQ-2: Judo re-entry criteria ("when the body can sustain it") is qualitative. Should `morning` flag when Salud frente is inactive >14 days with a prompt to reassess judo readiness?
- OQ-3: IA learning path is architecture + agents + orchestration, but no curriculum. Should Aprendizaje frente tracking prompt weekly: "What did I learn / ship this week?"

---

### GAP 4: Escritura Date Blocker Not Surfaced

**Brief blocker**:
> "Escritura date unknown — depends on parties outside Sito's control. Cannot plan post-move life until this is confirmed."
> — Personal Brief, Known Gaps section

**PRD coverage**: Not mentioned. Casa frente is treated as a standard frente with no blocker surfacing.

**Impact**: The cadence system will ask "is Casa progressing?" but will not surface the fundamental blocker that makes post-move planning impossible. This is a case where "no activity" is justified, but the system will not know why.

**Recommendation**: 
- Add Escritura date as a blocker in `CRITICAL_FACTS.md` or connectors.yaml (per FR-3.3)
- FR-3.3 should surface this blocker until escritura is signed, so the weekly review contextualizes Casa frente inactivity

---

## Gap Analysis: Work Domain

### GAP 5: Role Assumption Transition State Not Modeled

**Brief statement**:
> "The referente transition for Portal del Cliente has not formally started. The undefined state is not a blocker — the role can be stepped into before it is fully described."
> — Work Brief, Known Gaps section #4

**PRD coverage**: Portal del Cliente is listed as an active work frente with full FRs (commits, frente tagging), but the PRD does not acknowledge:
- The role has not formally started
- The frente may shift in scope or priority once the role is formalized
- Success signals assume the role is already active

**Impact**: The cadence system will begin tracking Portal del Cliente commitments and frente health before the role is defined, creating a false sense of clarity. The success signal ("Portal del Cliente has a clear process") is aspirational, not current state.

**Recommendation**: 
Update FR-1.3 or add a new FR:
- **FR-6.1 — Transition-state modeling**: Frentes may be tagged with a status (active, pending, on-hold). Portal del Cliente should initially be marked `pending` until the referente role is formally assumed. `morning` should surface pending frentes with a neutral note: "Portal del Cliente (pending role start) — not yet tracking."

---

### GAP 6: Client-Facing Process Flux Not Acknowledged

**Brief blocker**:
> "Client-facing working process in flux — Both sides are defining ticketing, refinement cadence, roadmap horizon, and planning rhythm simultaneously. Opportunities to shape the process exist — but only before the defaults are set."
> — Work Brief, Known Gaps section #2

**PRD coverage**: Not mentioned in Open Questions.

**Impact**: The PRD defines cadence at Sito's task level but does not acknowledge that the client's cadence (ticketing, refinement rhythm) is still being co-created. Once defaults are set, the cadence system should adapt; the PRD should explicitly plan for this pivot.

**Recommendation**: 
Add to Open Questions:
- OQ-6: Portal del Cliente's client-facing cadence (refinement frequency, planning horizon) is still being defined. Should the weekly review include a one-line note on client process maturity ("e.g., refinement cadence set, team roles clear"), or is this outside the system's scope?

---

### GAP 7: Startup Funding Horizon Deliberately Undefined

**Brief statement**:
> "Startup horizon is intentionally unset pending resolution of Gap #3 (funding milestones unknown). Do not design cadence rituals against this outcome until the horizon is known."
> — Work Brief, Outcomes section

**PRD coverage**: Previene MVP is listed as a work frente with no caveat about the undefined horizon.

**Impact**: The cadence system may surface "Previene MVP is on track" while the funding milestones (which determine the actual timeline) are still unknown. The brief explicitly warns against this; the PRD does not heed it.

**Recommendation**: 
- Update the definition of Previene MVP frente in goals.yaml to include a status flag: `status: horizon-pending`
- FR-3.3 (Blocker surfacing) should surface: "🔴 [BLOCKER] Previene: government funding milestones unknown — cannot set MVP deadline."
- Add to Open Questions:
  - OQ-7: Once funding milestones are confirmed, Previene MVP frente definitions and success signals must be re-drafted. Should the cadence system include a quarterly checkpoint to revisit startup-domain assumptions?

---

## Summary: Three Critical Underrepresentations

| Area | Brief Intent | PRD Coverage | Gap Size |
|------|-------------|--------------|----------|
| **Presencia practice** | Weekly end-of-day check-in tied to relationship + self-boundary work | None — frente exists, mechanism absent | **Major** |
| **Relación specificity** | Phone-down + scheduled quality time + proactive planning | Listed as frente, no behavioral anchors | **Major** |
| **Work transition states** | Role assumption pending, process still co-evolving | Treated as active/stable | **Moderate–Major** |
| **Personal fuzzy gaps** | Training plan, judo criteria, IA path undefined | Not acknowledged | **Moderate** |
| **Startup horizon** | Deliberately unset pending funding clarification | Treated as active frente | **Moderate** |

---

## Recommended Actions

**Phase 1 (pre-launch)**:
1. Add FR for presencia check-in (F4 or new)
2. Define at least two commitments for Relación frente
3. Add OQ-1, OQ-2, OQ-3 (Personal domain fuzzy gaps)
4. Add OQ-6 (Client-facing process flux)
5. Update Previene MVP to `status: horizon-pending` and surface as blocker

**Phase 2 (post-launch)**:
1. Implement FR-6.1 (Transition-state modeling for frentes)
2. Revisit Relación and Salud success signals once weekly review data accumulates
3. Confirm escritura date and re-assess Casa frente

---

## Notes

- The PRD's FRs are technically sound; gaps are primarily **what is not said rather than what is said incorrectly**.
- The brief's warnings about undefined horizons and transition states are intentional risk signals; the PRD's silence on them is not accidental — it's a design choice to treat Portal del Cliente and Previene as "active frentes now" rather than "active frentes pending clarification." This choice should be explicit.
