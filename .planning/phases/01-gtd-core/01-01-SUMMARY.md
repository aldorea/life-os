---
phase: 01-gtd-core
plan: 01
subsystem: vault
tags: [yaml, obsidian, gtd, para, frontmatter, schema]

requires: []
provides:
  - "Canonical vault folder structure (numbered PARA convention)"
  - "Frontmatter schemas for all note types (daily, weekly, person, meeting, knowledge, goal)"
  - "GTD tagging convention (#home, #office, #calls, #computer, #next, #esperando, #someday)"
  - "Migration map from old paths to new paths"
  - "Updated config.example.yaml with all new paths, tags, and settings"
affects: [01-02, 01-03, 01-04, 01-05, 01-06, 01-07]

tech-stack:
  added: []
  patterns:
    - "PARA-inspired numbered folder convention (lower number = higher frequency)"
    - "Strict frontmatter schemas with required (*) and optional (?) fields"
    - "GTD context tags for next-action filtering"
    - "Project status lifecycle: active -> stalled -> completed -> archived"

key-files:
  created:
    - ".planning/phases/01-gtd-core/vault-schema.md"
  modified:
    - "config.example.yaml"

key-decisions:
  - "Cache files (.cache/) hidden from user — calendar-cache, granola-processed, claude-memory are not user-facing"
  - "Note type identified via frontmatter tags field (daily, weekly, person, meeting, knowledge) — single field, no separate type field"
  - "Goals stored in goals.yaml with history[] array for progress tracking — not individual markdown files"
  - "Stalled project threshold set to 14 days default via projects.stalled_days config"

patterns-established:
  - "Numbered PARA folders: 00-11 for top-level, config/ and .cache/ unnumbered"
  - "Frontmatter required/optional convention: * for required, ? for optional in schema docs"
  - "Tag categories: contexts, projects, priority, actionability, status"

requirements-completed: [VAULT-01, VAULT-02, VAULT-03]

duration: 3min
completed: 2026-03-30
---

# Phase 01 Plan 01: Vault Schema Summary

**Vault folder structure (numbered PARA convention), frontmatter schemas for 6 note types, GTD tagging system, and migration map — all in config.example.yaml + vault-schema.md canonical reference**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-30T06:35:52Z
- **Completed:** 2026-03-30T06:38:39Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments
- Updated config.example.yaml with 22 structure paths (including new archive and cache), GTD context tags replacing generic #work/#personal, #someday actionability, status tags, and stalled_days setting
- Created vault-schema.md as the single source of truth with complete folder tree, 6 frontmatter schemas (daily, weekly, person, meeting, knowledge, goal), tagging convention with 5 tag categories, and migration map

## Task Commits

Each task was committed atomically:

1. **Task 1: Update config.example.yaml** - `25708e0` (feat)
2. **Task 2: Create vault-schema.md** - `4456ad3` (docs)

## Files Created/Modified
- `config.example.yaml` - Updated vault structure paths, GTD tags, status tags, stalled_days setting
- `.planning/phases/01-gtd-core/vault-schema.md` - Canonical reference: folder structure, frontmatter schemas, tagging convention, migration map

## Decisions Made
- Cache files moved to `.cache/` directory (hidden from Obsidian sidebar) rather than keeping in `08 Resources/`
- Note types identified by single `tags` frontmatter field (e.g., `tags: daily`) — follows existing convention, no separate `type` field needed
- Goals stored as structured YAML in `goals.yaml` with history[] arrays — not markdown files — enabling reliable programmatic reading
- Stalled project detection threshold defaults to 14 days (`projects.stalled_days`)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None.

## User Setup Required
None - no external service configuration required.

## Known Stubs
None - both files are complete reference documents with no placeholder data.

## Next Phase Readiness
- vault-schema.md is ready as canonical reference for all downstream plans (01-02 through 01-07)
- config.example.yaml has all paths/tags needed by skill upgrades and new skills
- Migration map is ready for the /migrate-vault skill (Plan 01-02)

## Self-Check: PASSED

- [x] config.example.yaml exists
- [x] vault-schema.md exists
- [x] 01-01-SUMMARY.md exists
- [x] Commit 25708e0 (Task 1) found
- [x] Commit 4456ad3 (Task 2) found

---
*Phase: 01-gtd-core*
*Completed: 2026-03-30*
