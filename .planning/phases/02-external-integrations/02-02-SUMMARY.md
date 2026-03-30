---
phase: 02-external-integrations
plan: 02
subsystem: connectors
tags: [apple-reminders, applescript, osascript, gtd-capture, inbox]

requires:
  - phase: 01-gtd-core
    provides: "Inbox append pattern, config.yaml structure, connectors.yaml pattern"
provides:
  - "Apple Reminders sync connector skill (skills/sync-reminders/SKILL.md)"
  - "Reminders dedup via processed registry (.cache/reminders-processed.md)"
affects: [morning-orchestrator, inbox-processing]

tech-stack:
  added: [osascript, applescript]
  patterns: [processed-registry-dedup, graceful-degradation-table, iso-date-construction]

key-files:
  created:
    - skills/sync-reminders/SKILL.md
  modified: []

key-decisions:
  - "ISO date construction in AppleScript to avoid locale-dependent formatting"
  - "Dedup key is reminder_name|creation_date_ISO (matches granola pattern)"
  - "Read-only: skill never modifies Reminders app data (D-07)"

patterns-established:
  - "AppleScript ISO date pattern: explicit year/month/day extraction with zero-padding"
  - "Dual dedup: processed registry + inbox text search (D-11)"

requirements-completed: [SYNC-02]

duration: 1min
completed: 2026-03-30
---

# Phase 02 Plan 02: Apple Reminders Sync Summary

**Apple Reminders connector via osascript with configurable lists, processed registry dedup, and graceful degradation for permission errors**

## Performance

- **Duration:** 1 min
- **Started:** 2026-03-30T14:45:37Z
- **Completed:** 2026-03-30T14:46:30Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Created sync-reminders skill that reads Apple Reminders via AppleScript and writes to vault inbox
- Configurable list selection via connectors.yaml (not hardcoded)
- Dual dedup strategy: processed registry (.cache/reminders-processed.md) + inbox text search
- Comprehensive graceful degradation table covering 10 failure scenarios
- Read-only constraint enforced (D-07): never modifies Reminders app

## Task Commits

Each task was committed atomically:

1. **Task 1: Create sync-reminders SKILL.md** - `2bf198e` (feat)

## Files Created/Modified

- `skills/sync-reminders/SKILL.md` - Apple Reminders sync connector skill (181 lines)

## Decisions Made

- ISO date construction in AppleScript (explicit year/month/day extraction with zero-padding) to avoid locale-dependent date formatting per research Pitfall 5
- Dedup key format `name|creation_date_ISO` matches the pattern established by sync-granola
- Read-only constraint (D-07): skill only reads from Reminders, never modifies/completes/deletes

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## Known Stubs

None - skill is fully specified with all data sources, formats, and error handling documented.

## User Setup Required

None - no external service configuration required. macOS automation permission will be prompted on first run.

## Next Phase Readiness

- Reminders connector ready for integration into morning orchestrator
- connectors.yaml needs reminders section added by user for their specific lists

---
*Phase: 02-external-integrations*
*Completed: 2026-03-30*
