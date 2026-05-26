# Correctness Review — PRD Life OS Adaptive Planning Loop
**Reviewer:** correctness  
**Date:** 2026-05-25  
**File reviewed:** `prd-life-os-2026-05-25/prd.md`  
**Cadence PRD baseline:** `prd-life-os-2026-05-24/prd.md`

---

## Verdict: PASS WITH NOTES

No requirement renders the feature set unimplementable, and there are no direct conflicts with Cadence PRD FRs that would break existing behavior. However, four issues — ranging from a dangling reference to a silent mid-week divergence — require resolution before implementation begins.

---

## Findings

### FINDING-1 — Unimplementable reference: "unless the user explicitly queries for patterns" (Severity: Medium)

**Location:** FR-P8.4  
**Text:** "pattern detection runs silently — no 'not enough data yet' message is shown **unless the user explicitly queries for patterns**."

NFR-P1 explicitly prohibits new top-level skills. No existing skill accepts a "query for patterns" invocation. There is no mechanism in `morning` or `close` — as defined in this PRD — through which the user could trigger an on-demand pattern query before 4 weeks of data exist. The clause is a dangling reference to a capability this PRD deliberately excludes.

**Impact:** The parenthetical clause is dead text; it will confuse the implementing author. Either remove it or define the exact in-flow trigger (e.g., "unless the user types 'ver patrones' during the Monday morning run").

---

### FINDING-2 — Unresolved source for blockers in standup: "or the current weekly note" (Severity: Medium)

**Location:** FR-P4.1, field 3 ("Blockers")  
**Text:** "taken from active blockers recorded in `CRITICAL_FACTS.md` **or the current weekly note**."

The weekly note schema defined in FR-P2.2 specifies only a `Day | Tasks | Notes` table body. There is no structured `blockers:` field in the weekly note format. At standup generation time, the implementing skill has no defined location in the weekly note to read blockers from. The `CRITICAL_FACTS.md` path is fully specified (Cadence PRD FR-3.3 + this PRD Section 6 Vault Paths table). The weekly note path is not.

**Impact:** The implementor will either skip the weekly note source silently or invent a format that may conflict with future changes to the weekly note schema. The fix is either (a) add a `blockers:` frontmatter field to the weekly note schema in FR-P2.2, or (b) remove the "or the current weekly note" clause and rely solely on `CRITICAL_FACTS.md`.

---

### FINDING-3 — No machine-readable daily plan record for disruption detection (Severity: Medium)

**Location:** FR-P6.1 cross-referenced with FR-P3.2  
**Text (FR-P6.1):** "tasks from the weekly plan that did not progress — it shall present them as candidate disruptions"  
**Text (FR-P3.2):** "The day's brief shall be written into the `Foco` section of the daily note."

The `Foco` section is freetext prose in the daily note (Cadence PRD FR-3.4 defines its shape as a narrative section). At close time, detecting "tasks from the weekly plan that did not progress" requires either: (a) a structured, machine-parseable record of what tasks were planned for today, or (b) re-reading the weekly note and diffing against Jira state changes.

The PRD does not specify which mechanism is used, nor does it define a structured task record format that `close` can reliably parse. If the implementor writes the daily plan as freetext prose, automated disruption detection against it will be fragile (regex over prose). If the implementor writes a structured block (e.g., YAML or a task list with IDs), that format is not specified here and must be consistent with what F5's activity inference also reads.

**Impact:** The architecture phase must define the exact structured format of the daily brief written to the `Foco` section, sufficient for machine diffing at close time. This is not a PRD contradiction but a specification gap that will produce an implementation ambiguity.

---

### FINDING-4 — Mid-week clarification cannot update a confirmed (immutable) weekly plan (Severity: Low)

**Location:** FR-P1.2 vs NFR-P5  
**FR-P1.2:** "The user may respond by adding items, removing items, or marking items as irrelevant for the current period. The confirmed list becomes the sole planning input for that morning's brief..."  
**NFR-P5:** "Once a weekly note's frontmatter has `status: confirmed`, `morning` shall not overwrite the task allocation."

On non-Monday mornings, if the user adds a new task during the clarification step (FR-P1.2), that task becomes "the sole planning input for that morning's brief" — but the confirmed weekly note is immutable (NFR-P5) and cannot be updated to reflect the addition. The result: the daily brief may include a task that does not appear in the confirmed weekly plan, creating a silent divergence. By end-of-week, the adherence metric (FR-P7.1) will under-count tasks actually worked because the added task was never written to the weekly note.

**Impact:** Clarification on non-Monday days should either (a) produce an amendment record appended to the weekly note (a new section, not overwriting the allocation table), or (b) be explicitly scoped to "today only" with a note that it does not affect the weekly plan. Currently neither is specified.

---

### FINDING-5 — NFR numbering is out of order (Severity: Cosmetic)

**Location:** Section 5, Non-Functional Requirements  
The sequence reads: NFR-P1, NFR-P2, NFR-P3, NFR-P4, NFR-P5, NFR-P6, NFR-P7, **NFR-P9**, **NFR-P10**, **NFR-P8**. NFR-P8 (Cadence PRD primacy) appears last, after NFR-P10. This has no behavioral consequence but makes the document harder to reference and may cause confusion during implementation cross-referencing.

---

## Non-Findings (Checked and Confirmed Clean)

- **F7 weekly review extension vs Cadence PRD FR-3.4**: Additive only. The four subsections are appended; the Frentes table and Compromisos section are explicitly preserved (FR-P7.2). No conflict.
- **FR-P2.3 confirmed plan skip logic**: Correctly handles the re-entry case ("already confirmed → display as-is"). No off-by-one on the Monday detection.
- **FR-P5.5 degraded mode**: Close's fallback to manual input when all connectors fail is correctly specified and consistent with NFR-P2.
- **Cadence PRD NFR-3 time budget**: NFR-P3 correctly inherits the ≤10 minute Monday budget and adds a 3-minute sub-budget for plan generation steps, which is consistent with the parent constraint.
- **No new top-level skills**: All FRs are scoped to `morning` and `close`. Confirmed against the full FR list.
- **Calendar write prohibition**: No FR implies a calendar write. NFR-P4 and Cadence PRD NFR-4 are both honored.
- **Pattern detection read-only**: FR-P8.3 and NFR-P7 are consistent — detection surfaces observations only, no automated plan changes.

---

## Testing Gaps

1. **Adherence metric correctness (FR-P7.1)**: The percentage calculation requires matching task names/IDs between the weekly note and close logs. There is no defined task identifier format — if task names drift between documents (edits, rewording), the match will silently undercount. The architecture phase should define a stable task ID or use Jira issue keys as anchors where available.

2. **First-Monday-of-week detection (FR-P2.4)**: The PRD says "Monday mornings (or the first morning run of the week if Monday was skipped)." The detection logic for "first morning run of the week when Monday was missed" is not specified. Edge case: if `morning` runs Saturday during a vacation week, it could incorrectly trigger weekly plan generation.

3. **Pattern detection week boundary (FR-P8.1)**: "4 or more full weeks of close logs" — a "full week" is not defined. Does a week with 2 close entries count? Does the current incomplete week count toward the 4? This threshold gate will produce different activation dates depending on interpretation.

---

## Residual Risks

- **Jira state-change granularity (OQ-1)**: The PRD correctly flags this as an open question. If the sync data only captures final-state snapshots rather than intra-day changes, activity inference (FR-P5.1) may miss tasks that were worked on but not transitioned. This is a data-availability risk, not a PRD logic error.
- **`otro` taxonomy coverage (OQ-4)**: The PRD correctly flags this. Five fixed types with `otro` as catch-all is a reasonable v1 decision; the risk is that a high `otro` rate makes the disruption summary (FR-P7.1) less actionable. The 70% auto-classification accuracy metric in Section 7 is the right signal to watch.
