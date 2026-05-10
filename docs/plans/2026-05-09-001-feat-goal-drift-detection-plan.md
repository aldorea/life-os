---
title: "feat: Add goal drift detection to close ritual"
type: feat
status: completed
date: 2026-05-09
origin: docs/brainstorms/2026-05-09-goal-drift-detection-requirements.md
---

# feat: Add goal drift detection to close ritual

## Summary

Extends the close ritual with a drift-detection phase (Step 8c) that reads a new `detection_goals:` section from the existing `config/goals.yaml`, scans recent daily notes for keyword absence, and flags neglected goals before the final git commit. If the user pauses a flagged goal, the pause is written back to `goals.yaml` and included in the Step 9 commit.

---

## Problem Frame

Life OS tracks a key metric — "% de focos diarios alineados con un goal mensual" — but has no mechanism to detect when a goal stops appearing in daily life. Goals can go unmentioned for weeks with no system response. This plan adds absence detection at the close ritual, the highest-signal moment of the day (see origin: `docs/brainstorms/2026-05-09-goal-drift-detection-requirements.md`).

---

## Requirements

- R1. `goals.yaml` has a `detection_goals:` section. Each entry: `name`, `keywords[]`, `alert_after_days`, `status` (`active` or `paused`), optional `paused_since`.
- R2. File ships seeded with 3 goals: entrenamiento, aprendizaje técnico, proyectos propios.
- R3. Keywords are case-insensitive, matched against free text — no special syntax required.
- R4. Close ritual runs drift detection in its final phase, before the good-night summary (Step 10).
- R5. Detection scans the last N existing daily notes (N = `alert_after_days`). Days with no note are skipped, not counted as absences.
- R6. Flagged goals produce a warning block with goal name and note-scan count. User offered pause toggle. Unactioned goals re-flag on next close if still absent.
- R7. Pause writes `status: paused` and `paused_since: YYYY-MM-DD` to `goals.yaml`. Cleared on manual resume.
- R8. If no goals are flagged, the drift-check phase produces no output.
- R9. If `goals.yaml` is missing or unreadable, drift detection is skipped silently.
- R10. Pause is a binary toggle: `active` or `paused`. No expiry, no auto-resume.
- R11. Resume is always manual — user edits `goals.yaml` directly.
- R12. Good-night summary (Step 10) always lists paused goals with their `paused_since` date.

**Origin flows:** F1 (goal drift detected at close), F2 (goal manually resumed)

**Origin acceptance examples:** AE1 (R5, R6), AE2 (R8), AE3 (R7, R10), AE4 (R9), AE5 (R3), AE6 (R1), AE7 (R2), AE8 (R4), AE9 (R11, R12)

---

## Scope Boundaries

- Detection is keyword-only — no LLM semantic alignment.
- No modifications to existing `goals.yaml` sections (`work_goals`, `growth_goals`, `habits`, `relationships`).
- No morning ritual integration.
- No Goal Covenant Engine numeric thresholds or breach protocols.
- No inline goal creation from within the close ritual.
- No historical dashboard or trend visualization.
- No automated keyword extraction.
- No automated tests — zero test infrastructure exists in this codebase.

### Deferred to Follow-Up Work

- Plugin cache sync (`~/.claude/plugins/cache/life-os/rituals/0.1.0/skills/close/SKILL.md`): must be updated after repo source edit. Mechanism TBD at implementation time — check if there is a `claude plugin` refresh command or if the file can be copied directly.
- Morning Ritual Commitment Audit (idea #2 from ideation doc): separate feature.
- Goal Covenant Engine (idea #1): builds on this feature.

---

## Context & Research

### Relevant Code and Patterns

- `rituals/skills/close/SKILL.md` — repo source of the close ritual; the file to edit. Steps 8b (wiki log append), 9 (git commit), and 10 (good-night summary) are the insertion neighborhood.
- Vault `config/goals.yaml` — already exists with `work_goals`, `growth_goals`, `habits`, `relationships` sections. This plan adds a new `detection_goals:` top-level section. Existing sections use `status` values `not_started/in_progress/completed/abandoned`; drift detection uses a separate vocabulary: `active/paused`.
- `rituals/skills/morning/SKILL.md` — reads `config/goals.yaml` via `obsidian read path="config/goals.yaml"`. Shows vault-relative path pattern for config reads.
- `config.example.yaml` — defines `structure.goals: "config/goals.yaml"` and `daily.date_format: "DD-MM-YYYY"`. Step 0 of every skill loads this as `~/.config/life-os/config.yaml`.

### Institutional Learnings

- `obsidian daily:read` reads only today's note — no date parameter exists. Past-note scanning requires: iterate backward from yesterday, format as `DD-MM-YYYY`, call `obsidian read path="{config.structure.daily_notes}/{formatted_date}.md"`, skip on not-found, count only successful reads toward N.
- `obsidian CLI property:set` operates only on Obsidian markdown files. `goals.yaml` mutation must use `yaml` library parse → field-level mutation → `yaml.stringify()` → `fs.writeFileSync()`. This avoids LLM format drift (Pitfall 7 from `.planning/research/PITFALLS.md`).
- Graceful degradation is a first-class concern in all close ritual steps — every failure maps to a safe fallback. The new drift step must follow this pattern.
- Git safety pattern (D-14): backup commit (Step 6) before mutations, result commit (Step 9) after. Pause writes occur in Step 8c; Step 9 must include `goals.yaml` in its `git add` when a pause was written.

### External References

None required — local patterns are well-established.

---

## Key Technical Decisions

- **`detection_goals:` as new top-level section**: adds drift-detection goals without touching the existing schema. The two schemas coexist in the same file under different keys. Existing skills that read `work_goals`/`growth_goals` are unaffected.
- **Step 8c inserted before Step 9 (git commit)**: AE8 says "after step 9," but the brainstorm's outstanding question explicitly deferred step ordering to planning. Inserting before Step 9 is cleaner because it allows the pause write to be included in the same commit, avoiding a second commit step. The conflict with AE8 is intentional — this is the planning resolution.
- **Step 9 `git add` extended conditionally**: when a pause was written in Step 8c, Step 9's `git add` includes `{config.structure.goals}`. When no pause was written, `git add` scope is unchanged (R8 / zero-noise).
- **Backward date walk with 30-day safety limit**: iterate from yesterday backward; count only days where `obsidian read` succeeds; stop when N notes found OR 30 days elapsed (prevents unbounded loops for new users with sparse notes).
- **Keyword matching**: `toLowerCase()` on both the note content and each keyword, then `.includes()`. No regex — keeps the skill prose simple and deterministic.
- **Pause write is field-level, not file-regeneration**: to prevent LLM format drift, the SKILL.md instruction must specify: parse YAML, locate goal by `name`, mutate fields, serialize — do not regenerate the file.

---

## Open Questions

### Resolved During Planning

- **goals.yaml location**: vault-relative `config/goals.yaml` — consistent with `config.example.yaml`'s `structure.goals` entry. Resolved before plan-write (was flagged as `[Resolve before planning]` in origin doc).
- **Step ordering (before/after Step 9)**: Step 8c before Step 9, so pause writes are included in the result commit. (see Key Technical Decisions)
- **Pause write mechanism**: yaml library + fs, not obsidian CLI. (see Institutional Learnings)

### Deferred to Implementation

- **Plugin cache sync mechanism**: whether `claude plugin` has a refresh command or requires a direct file copy. Low-risk — file copy is always available as fallback.
- **Schema validation on load**: lenient parsing for v1 — if a detection goal is missing a field, skip it and continue (don't block the ritual).
- **Step 9 goals.yaml `git add` path**: the exact path string for the `git add` command depends on runtime config resolution (`{config.vault_path}/{config.structure.goals}`). Confirm path construction at implementation time.

---

## High-Level Technical Design

> *This illustrates the intended approach and is directional guidance for review, not implementation specification. The implementing agent should treat it as context, not code to reproduce.*

```
Step 8c: Drift Detection

1. Read config.structure.goals → derive absolute path
2. obsidian read path="{config.structure.goals}" → parse YAML
   └─ on error/missing → skip step 8c (R9), proceed to Step 9

3. Extract detection_goals where status == "active"
   └─ if none → no output, proceed to Step 9 (R8)

4. For each active goal:
   a. found_notes = []
   b. date = today - 1 day
   c. while len(found_notes) < alert_after_days AND date > today - 30 days:
        content = obsidian read path="{daily_notes}/{DD-MM-YYYY(date)}.md"
        if success → found_notes.append(content)
        date -= 1 day
   d. matched = any keyword (lowercase) in any note content (lowercase)
   e. if not matched AND len(found_notes) > 0 → flagged

5. If any flagged goals:
   a. Display warning block (goal name + notes scanned count)
   b. Offer pause toggle for each flagged goal (interactive)
   c. For each chosen pause:
      - Parse goals.yaml, locate by name, set status=paused, paused_since=today
      - Serialize with yaml library, fs.writeFileSync (not obsidian CLI)
      - Set pause_written_flag = true

6. Always: display paused goals summary from detection_goals (R12)
   └─ if any paused goals → list name + paused_since

7. Proceed to Step 9
   └─ if pause_written_flag: git add includes goals.yaml path
```

---

## Output Structure

No new directories created. Two existing files are modified:

```
config/goals.yaml          ← vault (add detection_goals: section)
rituals/
  skills/
    close/
      SKILL.md             ← repo source (add Step 8c, update Step 9, update Degradation table)
```

---

## Implementation Units

### U1. Add `detection_goals:` section to `config/goals.yaml`

**Goal:** Augment the existing `config/goals.yaml` with a new `detection_goals:` top-level section containing the 3 seed goals. Existing sections are untouched.

**Requirements:** R1, R2, R3, R10

**Dependencies:** None

**Files:**
- Modify: vault `config/goals.yaml` (read via `obsidian read path="config/goals.yaml"`, write via `obsidian create path="config/goals.yaml" content="..." overwrite`)

**Approach:**
- Read the current file to preserve all existing content exactly
- Append a `detection_goals:` section after the existing sections
- Seed with 3 goals per R2: entrenamiento (`alert_after_days: 3`, keywords: `[gym, entreno, entrenamiento, workout, cardio]`, `status: active`), aprendizaje técnico (`alert_after_days: 5`, keywords: `[estudio, aprender, curso, lectura, tutorial]`, `status: active`), proyectos propios (`alert_after_days: 3`, keywords: `[life-os, orbitant, producto, side-project]`, `status: active`)
- Each entry has `name`, `keywords`, `alert_after_days`, `status`, and `paused_since: null`
- Status vocabulary is `active/paused` — distinct from the existing `not_started/in_progress/completed/abandoned` vocabulary in other sections

**Patterns to follow:**
- Existing `config/goals.yaml` comment style (Spanish, section headers with `# ===` dividers)
- `config.example.yaml` for path reference conventions

**Test scenarios:**
- Test expectation: none — this is a YAML config edit with no runtime behavior of its own. Correctness verified by reading back the file and confirming schema.

**Verification:**
- `obsidian read path="config/goals.yaml"` shows a `detection_goals:` section with 3 entries, each with `name`, `keywords` list, `alert_after_days`, `status: active`, and `paused_since: null`
- Existing sections (`work_goals`, `growth_goals`, `habits`, `relationships`) are unchanged

---

### U2. Add Step 8c (drift detection) to close SKILL.md

**Goal:** Insert the drift-detection phase as Step 8c in `rituals/skills/close/SKILL.md`, update Step 9 to include goals.yaml in git add when a pause was written, update the Graceful Degradation table, and add the paused-goals summary to Step 10.

**Requirements:** R4, R5, R6, R7, R8, R9, R10, R11, R12; flows F1, F2

**Dependencies:** U1 (detection_goals section must exist for a meaningful end-to-end test, though Step 8c degrades gracefully if missing)

**Files:**
- Modify: `rituals/skills/close/SKILL.md`

**Approach:**
- Insert Step 8c between the `### 8b. Bridge daily → wiki log` heading and the `### 9. Git commit result` heading
- Step 8c prose must specify:
  - Read `{config.structure.goals}` via `obsidian read`; on error/missing → skip silently (R9)
  - Extract `detection_goals` where `status == "active"`; if none → proceed to Step 9 with no output (R8)
  - For each active goal: walk backward from yesterday using `DD-MM-YYYY` format, call `obsidian read path="{config.structure.daily_notes}/{date}.md"` per day, skip missing days, stop when N notes found OR 30 days exhausted
  - Check case-insensitive keyword presence across found notes (R3, R5)
  - Build flagged list: goals where no keyword matched across all found notes
  - If any flagged: display warning block (goal name + notes-scanned count), offer pause toggle for each (R6)
  - For each user-selected pause: parse goals.yaml with yaml library, set `status: paused` and `paused_since: {today}`, serialize and write back via fs (not obsidian CLI) (R7)
  - Always display paused goals from detection_goals with `paused_since` date (R12)
  - Track whether any pause was written (for Step 9)
- Update Step 9 (`### 9. Git commit result`): add goals.yaml to the `git add` command when a pause was written in Step 8c — use the resolved vault-absolute path `{config.vault_path}/{config.structure.goals}`
- Update Graceful Degradation table: add row `goals.yaml missing or unreadable → Skip Step 8c silently, continue to Step 9`
- Step 10 good-night summary: note that paused goals are already listed by Step 8c (R12 is handled in 8c, not 10 — Step 10 only needs a mention if 8c was skipped due to graceful degradation)

**Patterns to follow:**
- Existing step prose style in close/SKILL.md (Spanish for user-facing output, English for technical instructions)
- Step 6 git safety pattern: `git -C VAULT add {paths} && git -C VAULT commit -m "..."` — extend Step 9 to conditionally add goals path
- Graceful Degradation table format in close/SKILL.md

**Test scenarios:**

*Covers AE1.* Happy path — goal flagged: given entrenamiento has `alert_after_days: 3` and the last 3 existing daily notes contain no keywords from its list, when close ritual runs Step 8c, the warning block shows entrenamiento with "3 notas sin mención" (or equivalent wording). No other output if other goals pass.

*Covers AE2.* Happy path — no flags: given all active detection goals have at least one keyword match in their last N existing notes, when Step 8c runs, it produces no output and proceeds immediately to Step 9.

*Covers AE3.* Pause flow: given entrenamiento is flagged, when the user selects pause, `goals.yaml` is written with `status: paused` and `paused_since: {today}` for entrenamiento. On the next close ritual run, entrenamiento does not appear in drift detection output.

*Covers AE4.* Graceful degradation: given `detection_goals:` section does not exist in goals.yaml (or the file is missing), when Step 8c runs, it produces no output and the close ritual continues normally to Step 9 and Step 10.

*Covers AE5.* Case-insensitive match: given keywords `[gym, entreno, entrenamiento]` and today's note contains "Fui al gym a las 19:00", when drift detection runs, entrenamiento is counted as present and not flagged.

*Covers AE8.* Step ordering: when close ritual completes steps 8b → 8c → 9, drift detection output (if any) appears before the good-night summary and the Step 9 git commit.

*Covers AE9.* Paused goals summary: given entrenamiento was paused on 2026-05-10, when close ritual runs on 2026-05-15, the good-night summary includes a paused goals line showing "Entrenamiento (pausado desde 2026-05-10)". Entrenamiento does not appear in the flagged-goals block.

Edge case — sparse notes: given the user has only 1 daily note in the last 30 days and a goal has `alert_after_days: 3`, when Step 8c scans, it finds 1 note (not 3). A goal is flagged only if no keyword appears in that 1 note — it is not flagged simply for having fewer notes than the threshold.

Edge case — all paused: given all detection goals are paused, when Step 8c runs, drift detection produces no warning block (R8 — no active goals to check), but the paused-goals summary still lists all paused goals with dates (R12).

Error path — yaml write failure: if the fs write back to goals.yaml fails, Step 8c should surface the error to the user but not abort the entire close ritual. The ritual continues to Step 9 without the pause applied.

**Verification:**
- Run the close ritual end-to-end with a daily note that contains no keywords for at least one detection goal. Confirm: warning block appears before "Dia cerrado." output.
- Run again after confirming the manual pause. Confirm: no warning for the paused goal; paused summary appears in good-night output.
- Temporarily rename `goals.yaml` to verify graceful degradation (ritual completes without error).
- Verify Step 9 commit message and `git log` shows goals.yaml in the committed files only when a pause was written.

---

## System-Wide Impact

- **Interaction graph:** Close ritual only. No other skill reads or writes `detection_goals:`. Morning skill reads `work_goals`/`growth_goals` — unaffected by the new section.
- **Error propagation:** Step 8c errors (goals.yaml unreadable, yaml parse failure, obsidian read timeout) are caught and silently skipped per R9. Write failures surface to the user but do not abort the ritual.
- **State lifecycle risks:** If a pause write partially fails (fs write interrupted), `goals.yaml` may be corrupted. Mitigation: write to a temp variable first, verify the serialized string is valid YAML before writing.
- **API surface parity:** None — no external API surface.
- **Integration coverage:** The pause write crosses from the skill instruction layer to the vault filesystem (via fs, not obsidian CLI). This path is not testable without a running vault — manual end-to-end verification is the only coverage available.
- **Unchanged invariants:** The close ritual's existing steps (0–8b, 9, 10) are unchanged except Step 9's conditional git add extension. `obsidian daily:read`, `obsidian tasks done`, and all task management flows are unaffected.

---

## Risks & Dependencies

| Risk | Mitigation |
|------|------------|
| `goals.yaml` write corrupts file (partial write, invalid YAML) | Read → parse → mutate in memory → validate serialized output → write. If serialization fails, surface error without writing. |
| Backward date walk runs too far on sparse-note users | 30-day safety limit prevents unbounded loops. |
| Plugin cache not updated after repo source edit | Document sync step explicitly in Deferred to Implementation. Verify at implementation time. |
| Step 9 git add path resolution differs from expected vault path | Confirm `{config.vault_path}/{config.structure.goals}` resolves correctly at runtime before finalizing Step 9 prose. |
| Detection false positives (note mentions keyword casually, not as goal activity) | Documented as known limitation in origin doc — writing absence proxy, not behavioral absence detector. Acceptable for v1. |

---

## Documentation / Operational Notes

- The `detection_goals:` section is user-maintainable — users add/remove goals and keywords directly in `config/goals.yaml` via Obsidian or any editor.
- Resume from pause is always manual (`status: active`, remove `paused_since`) — no ritual integration.
- The 30-day safety limit on backward scanning means a goal with `alert_after_days: 10` and only 2 notes in 30 days will check 2 notes, not 10. Document this behavior in comments within the SKILL.md step.

---

## Sources & References

- **Origin document:** [docs/brainstorms/2026-05-09-goal-drift-detection-requirements.md](docs/brainstorms/2026-05-09-goal-drift-detection-requirements.md)
- Close ritual: `rituals/skills/close/SKILL.md`
- Vault config: vault `config/goals.yaml`
- Config schema: `config.example.yaml`
- Pitfalls: `.planning/research/PITFALLS.md` (Pitfall 7 — LLM format drift in vault mutations)
- Stack: `.planning/research/STACK.md` (yaml v2.x for YAML config files)
