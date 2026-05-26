---
title: Input Reconciliation — Brief vs PRD (Adaptive Planning Loop)
created: 2026-05-25
brief: brief-life-os-adaptive-planning-2026-05-25/brief.md
prd: prd-life-os-2026-05-25/prd.md
---

## Summary

5 gaps identified across qualitative intent, success metrics, open questions, and design constraints.

---

## Gaps

### GAP-1 — Horizon Hierarchy Unlock Mechanism (critical)

**In brief:** The brief defines an explicit `Horizon Hierarchy` table with gated unlock conditions — Daily unlocked from Day 1; Weekly unlocked only after "daily loop runs ≥4 days/week for 2 weeks"; Monthly unlocked after "weekly reviews completed ≥3 weeks/month". This is a core design principle: the hierarchy earns its way up.

**Missing in PRD:** The PRD references the horizon hierarchy only indirectly (Non-Goals mentions quarterly horizon). There is no FR, NFR, or note specifying that Weekly plan generation (F2) should be gated or that the system should detect and surface readiness to unlock the next horizon. The unlock mechanism — and whether the system enforces or merely suggests it — is fully absent.

**Severity: critical.** The brief positions this as load-bearing GTD trust-building logic; the PRD treats weekly planning as always-on from day 1. This is a silent scope change that contradicts the brief's philosophy.

---

### GAP-2 — "Coherence comes from the loop, not the command list" — UX Narrative Intent (notable)

**In brief:** The Design Principles section explicitly states: "Skills feel arbitrary today because no narrative connects them. The adaptive planning loop (morning → close → learn) is that narrative. Once the loop exists, the commands make sense as steps in a workflow." This is a design intention about perceived UX coherence.

**Missing in PRD:** The PRD does not reflect this anywhere — no NFR, no UX note, no requirement that `morning` or `close` contextually narrate the user's position in the loop (e.g., "Step 2 of today's loop — closing the day"). There is no traceability from this brief principle to any deliverable behavior.

**Severity: notable.** This is a qualitative principle, not a behavior, so it may belong in implementation guidance rather than FRs. But its total absence means future implementers have no hook to deliver on it.

---

### GAP-3 — Success Metric: "Skill Surface" (notable)

**In brief:** The success metrics table includes: "Skill surface — No new top-level skills added; loop lives within morning + close."

**In PRD:** NFR-P1 captures the no-new-skills constraint, but the success metric framing is absent from Section 7. The PRD's metrics table (7 rows) does not include a metric for skill surface stability, only behavioral metrics. The brief explicitly treats this as a measurable outcome; the PRD demotes it to an NFR with no counter-metric.

**Severity: notable.** Omitting it from Section 7 means there is no acceptance criterion for this constraint — it becomes invisible to future verification.

---

### GAP-4 — OQ-5 Substitution: Cadence Integration Sequencing Dropped (notable)

**In brief:** OQ-5 asks: "How does this loop interact with the Cadence System PRD (frentes, commitments)? Same morning/close, layered on top. Needs explicit sequencing in the architecture phase."

**In PRD:** OQ-5 is replaced with a different question about clarification step placement timing within morning. The Cadence integration sequencing concern is largely addressed in Section 1 (Relationship to Cadence PRD) and the sequence tables in Section 6 — but the original OQ is not closed/resolved, it is silently substituted. The architecture-phase sequencing instruction ("needs explicit sequencing in the architecture phase") is dropped entirely.

**Severity: notable.** The sequencing question is partially answered in the PRD, but the forward-looking note ("needs explicit sequencing in the architecture phase") was an explicit instruction to the architect — its disappearance means the downstream architecture phase may not know this work is required.

---

### GAP-5 — "Clarification Step" UX Tone (minor)

**In brief:** The clarification prompt is quoted verbatim: "Before we plan — is this list complete? Anything missing, anything that shouldn't be here?" This specific phrasing signals low-friction, conversational UX — not a formal checkpoint.

**In PRD:** FR-P1.1 reproduces this exact quote, which is good. However, no NFR or note enforces that the overall morning and close UX must remain conversational and low-friction. The brief's concern about friction is qualitative and pervasive (it motivates the whole design); the PRD addresses it only in the one quoted string. The principle "do not ask Sito to recall what he did" appears in FR-P5.4 but is not elevated to a global NFR.

**Severity: minor.** The specific quotes are preserved, but the generalized friction principle has no home in the NFR section.

---

## Verdict

All 5 gaps are addressable without structural changes to the PRD. GAP-1 is the only one that represents a substantive silent scope change — the horizon unlock mechanism needs at least a note or NFR. GAP-3 and GAP-4 are traceability holes. GAP-2 and GAP-5 are implementation guidance that belongs in a dev story rather than the PRD itself, but should be noted somewhere before architecture begins.
