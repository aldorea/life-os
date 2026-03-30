# Phase 2: External Integrations - Context

**Gathered:** 2026-03-30
**Status:** Ready for planning

<domain>
## Phase Boundary

External data flows into vault automatically. Build sync-jira and sync-reminders connectors. Wire all connectors (including existing sync-calendar, sync-granola, sync-slack) into a unified inbox flow with dedup. Existing sync skills stay as-is unless the inbox flow requires changes.

</domain>

<decisions>
## Implementation Decisions

### Jira Connector Design
- **D-01:** Jira projects configured in `connectors.yaml` with explicit MCP server mapping per project (e.g., `mcp_server: jira-afianza`, `project_key: AFNZ`). Adding a new Jira instance = add MCP server + add entry to connectors.yaml.
- **D-02:** Each synced ticket becomes an individual markdown note in `06 Jira/` with frontmatter. Enables wikilinks like `[[TICKET-123]]`, Obsidian search, and graph connections.
- **D-03:** Synced fields: key, summary, status, assignee, priority, project (core) + sprint name, due date, created date (planning) + parent epic, linked issues, subtask list (relationships). No comments.
- **D-04:** Re-sync overwrites frontmatter fields silently. Jira is source of truth. No changelog in note body.
- **D-05:** Jira sync creates inbox items ONLY for newly assigned tickets or action-requiring status changes (not every update). Ticket notes in `06 Jira/` are always updated regardless.

### Apple Reminders
- **D-06:** Integration via AppleScript (`osascript`). No extra dependencies, runs on any Mac. Claude executes directly via Bash.
- **D-07:** Read-only: pull reminders into inbox for GTD processing. Reminders app is a capture point (Siri, phone, watch). No write-back.
- **D-08:** Which Reminders lists to sync is configurable in `connectors.yaml`. Only specified lists are read.

### Existing Sync Skills
- **D-09:** Existing sync-calendar, sync-granola, sync-slack remain untouched. Focus Phase 2 effort on new connectors (Jira, Reminders) and the unified inbox flow. Only modify existing skills if the inbox flow requires it.

### Unified Inbox Flow
- **D-10:** All connectors write directly to `00 Inbox.md` (Slack already does this). Single entry point. Morning ritual processes inbox with AI suggest & confirm (D-15 from Phase 1).
- **D-11:** Dedup is wikilink-based. If a Jira ticket `[[TICKET-123]]` already exists as a vault note, don't create a duplicate inbox item for it from another source (e.g., Slack mention). Wikilinks are the dedup key.
- **D-12:** Granola already writes meeting notes + backlog items (not inbox). Calendar writes to cache. These patterns stay. Only Jira (new assignments), Slack (actions/decisions), and Reminders (captured items) write to inbox.

### Claude's Discretion
- Jira ticket note template (exact frontmatter fields and body structure)
- AppleScript implementation details for reading Reminders
- Exact inbox item format for Jira and Reminders entries
- Error handling and graceful degradation patterns (follow existing skill conventions)
- connectors.yaml schema for new sections (Jira, Reminders)

</decisions>

<specifics>
## Specific Ideas

- Jira connector uses the existing MCP servers (`mcp__jira-afianza__*` and `mcp__jira-previene__*`) — no HTTP API calls needed
- Reminders sync should feel lightweight — quick capture from Siri/Watch/Phone shows up in vault inbox next morning
- connectors.yaml already exists for Slack channel config — extend it with `jira:` and `reminders:` sections for consistency

</specifics>

<canonical_refs>
## Canonical References

### Phase 1 decisions (carry-forward)
- `.planning/phases/01-gtd-core/01-CONTEXT.md` — D-05 (morning chains syncs), D-14 (git auto-commit), D-15 (AI suggest & confirm)

### Vault structure
- `config.example.yaml` — Vault folder paths, connector config location, daily note settings
- `config.example.yaml` §structure.connectors_config — Points to `config/connectors.yaml` for connector-specific settings

### Existing sync patterns
- `skills/sync-slack/SKILL.md` — Established pattern for writing to inbox with wikilinks and dedup
- `skills/sync-granola/SKILL.md` — Established pattern for meeting notes, People enrichment, Backlog extraction
- `skills/sync-calendar/SKILL.md` — Established pattern for cache-based sync with graceful degradation

### Requirements
- `.planning/REQUIREMENTS.md` — SYNC-01 through SYNC-06 define the external integration requirements
- `.planning/ROADMAP.md` — Phase 2 scope and success criteria

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `skills/sync-slack/SKILL.md`: Inbox append pattern with wikilink enrichment and dedup — reuse for Jira inbox items and Reminders
- `skills/sync-granola/SKILL.md`: People enrichment and processed-item registry pattern — Jira could use similar dedup registry
- `config.example.yaml`: All vault paths defined — new connectors follow same `{config.structure.*}` resolution

### Established Patterns
- Config-driven: All vault paths resolved via `config.yaml`, never hardcoded
- Graceful degradation: Each connector handles missing tools/MCP/data without aborting
- Wikilink enrichment: Match names against `05 People/`, match projects against `02 Projects/`
- Morning orchestrator: Chains all sync skills in sequence, each independent

### Integration Points
- `skills/morning/SKILL.md`: Will need to call `sync-jira` and `sync-reminders` in the chain
- `connectors.yaml`: New `jira:` and `reminders:` sections alongside existing Slack config
- `00 Inbox.md`: Central inbox where Jira assignments, Slack actions, and Reminders land
- `06 Jira/`: Directory for individual Jira ticket notes (path already defined in config)

</code_context>

<deferred>
## Deferred Ideas

- Bidirectional Reminders sync (mark completed in vault → complete in Reminders) — future phase if needed
- Jira comment sync — omitted to keep notes clean; reconsider if context loss becomes an issue
- Smart merge of cross-source items (AI-based dedup beyond wikilinks) — over-engineered for now
- Jira status change changelog in note body — revisit if history tracking becomes important

</deferred>

---

*Phase: 02-external-integrations*
*Context gathered: 2026-03-30*
