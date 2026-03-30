---
phase: 01-gtd-core
plan: 05
subsystem: gtd-views
tags: [gtd, next-actions, projects, someday, context-filtering, stalled-detection]

# Dependency graph
requires:
  - phase: 01-gtd-core
    provides: "config.example.yaml with structure paths, tags, and backlog_sections (Plan 01)"
provides:
  - "/next-actions skill: context-filtered task view from Backlog"
  - "/projects skill: project list with stalled detection"
  - "/someday skill: someday/maybe list with review mode"
affects: [01-06, 01-07]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Read-only view skill pattern with config-driven paths"
    - "Cross-reference pattern: projects skill reads both Projects folder and Backlog"
    - "Review mode pattern: read-only by default, mutating with git safety on explicit request"

key-files:
  created:
    - skills/next-actions/SKILL.md
    - skills/projects/SKILL.md
    - skills/someday/SKILL.md
  modified: []

key-decisions:
  - "Context filtering uses config.tags.contexts for valid values, not hardcoded list"
  - "Stalled detection checks both no-next-action AND no-progress conditions independently"
  - "Someday review mode requires git commit before mutations (D-14) and per-item confirmation (D-15)"

patterns-established:
  - "GTD view skill: Step 0 config, read vault data, filter/group, present, graceful degradation"
  - "Cross-reference pattern: one skill reads multiple vault sources to derive status"

requirements-completed: [GTD-03, GTD-04, GTD-05, GTD-06]

# Metrics
duration: 3min
completed: 2026-03-30
---

# Phase 01 Plan 05: GTD Views Summary

**Three GTD view skills -- /next-actions with context filtering, /projects with stalled detection, /someday with interactive review mode**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-30T06:45:56Z
- **Completed:** 2026-03-30T06:48:00Z
- **Tasks:** 2
- **Files created:** 3

## Accomplishments
- /next-actions skill filters Backlog tasks by GTD context (#home, #office, #calls, #computer) with priority ordering
- /projects skill cross-references Projects folder with Backlog to detect stalled projects (no next action or no progress)
- /someday skill reads "Algun dia" section and #someday tags with optional interactive review mode

## Task Commits

Each task was committed atomically:

1. **Task 1: Create /next-actions skill** - `a9b8957` (feat)
2. **Task 2: Create /projects and /someday skills** - `2c438ce` (feat)

## Files Created/Modified
- `skills/next-actions/SKILL.md` - Context-filtered next actions view from Backlog
- `skills/projects/SKILL.md` - Project list with stalled detection via Backlog cross-reference
- `skills/someday/SKILL.md` - Someday/maybe list with interactive review mode

## Decisions Made
- Context filtering uses `config.tags.contexts` for valid values rather than hardcoding specific tags
- Stalled detection evaluates two independent conditions (no next action, no progress) that can both be true
- Someday review mode follows D-14 (git safety) and D-15 (suggest & confirm) patterns for mutations

## Deviations from Plan

None -- plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None -- no external service configuration required.

## Next Phase Readiness
- Three GTD view skills ready for use alongside capture and processing skills
- /projects stalled detection ready to be referenced by weekly review (Plan 06)
- /someday review mode ready to be called from weekly review flow

---
*Phase: 01-gtd-core*
*Completed: 2026-03-30*
