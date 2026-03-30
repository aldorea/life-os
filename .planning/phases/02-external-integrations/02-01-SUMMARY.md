---
phase: 02-external-integrations
plan: 01
subsystem: sync
tags: [jira, mcp, connectors, skill, obsidian]

requires:
  - phase: 01-gtd-core
    provides: "Vault structure, config.yaml pattern, sync-slack skill pattern, inbox format"
provides:
  - "sync-jira SKILL.md connector for multi-project Jira sync via MCP"
  - "connectors.example.yaml reference config with jira, reminders, slack sections"
affects: [02-02, 02-03, morning-orchestrator]

tech-stack:
  added: []
  patterns:
    - "MCP server mapping per Jira project in connectors.yaml"
    - "Ticket note template with full frontmatter and wikilink relationships"
    - "Inbox-worthy change detection (new assignment + status transition)"

key-files:
  created:
    - "skills/sync-jira/SKILL.md"
    - "connectors.example.yaml"
  modified: []

key-decisions:
  - "Jira sync uses dynamic MCP server mapping per project -- no hardcoded instances"
  - "Re-sync overwrites entire note (frontmatter + body) -- Jira is source of truth"
  - "Inbox items only for new assignments and action-requiring status transitions (To Do, In Review)"

patterns-established:
  - "Connectors config schema: each connector section in connectors.yaml with typed entries"
  - "Jira ticket note: frontmatter with 13 fields + Relationships body with wikilinks"
  - "Inbox dedup: search for [[TICKET-KEY]] wikilink before appending"

requirements-completed: [SYNC-01]

duration: 2min
completed: 2026-03-30
---

# Phase 02 Plan 01: Jira Sync Connector Summary

**Multi-project Jira sync skill via MCP with dynamic config, ticket notes with wikilink relationships, and selective inbox items for new assignments and status changes**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-30T14:45:35Z
- **Completed:** 2026-03-30T14:47:14Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments
- connectors.example.yaml with Jira, Reminders, and Slack sections as reference config
- sync-jira SKILL.md (157 lines) implementing full Jira sync flow: config reading, MCP fetching, ticket note writing, inbox change detection, wikilink enrichment, graceful degradation
- Ticket note template with 13 frontmatter fields and Relationships section using Obsidian wikilinks

## Task Commits

Each task was committed atomically:

1. **Task 1: Create connectors.example.yaml** - `9b9e06e` (feat)
2. **Task 2: Create sync-jira SKILL.md** - `7e77482` (feat)

## Files Created/Modified
- `connectors.example.yaml` - Reference config with jira projects (MCP server mapping), reminders lists, slack channels
- `skills/sync-jira/SKILL.md` - Jira sync connector skill: multi-project MCP fetch, ticket notes, selective inbox items, graceful degradation

## Decisions Made
- Followed plan exactly as specified -- no deviations needed

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required. Users copy connectors.example.yaml to their vault and fill in values.

## Next Phase Readiness
- sync-jira skill ready for invocation -- requires configured MCP servers and connectors.yaml
- connectors.example.yaml provides schema reference for all connector configs
- Morning orchestrator (Plan 03) can wire sync-jira into the daily chain
- sync-reminders (Plan 02) can use same connectors.yaml for its config section

---
*Phase: 02-external-integrations*
*Completed: 2026-03-30*
