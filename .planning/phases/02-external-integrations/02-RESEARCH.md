# Phase 2: External Integrations - Research

**Researched:** 2026-03-30
**Domain:** Sync connectors (Jira MCP, Apple Reminders AppleScript), unified inbox flow, deduplication
**Confidence:** HIGH

## Summary

Phase 2 builds two new sync connectors (Jira and Apple Reminders) and wires all connectors into a unified inbox flow with wikilink-based deduplication. The technical landscape is favorable: both Jira MCP servers are already configured and operational (mcp/atlassian Docker via FastMCP), AppleScript access to Reminders is confirmed working with full read access to all lists, and three existing sync skills (calendar, granola, slack) establish clear patterns to follow.

The primary risk is Jira MCP tool reliability -- community reports from early 2026 indicate occasional JQL execution errors with the mcp/atlassian image. The skill must handle MCP failures gracefully per the established degradation pattern. The Reminders connector is low-risk since osascript is native macOS and already confirmed working.

**Primary recommendation:** Build sync-jira and sync-reminders as new SKILL.md files following the exact pattern of sync-slack (config-driven, wikilink-enriched, graceful degradation). Extend connectors.yaml with jira: and reminders: sections. Wire both into morning orchestrator. Implement wikilink-based dedup as a shared check across all inbox-writing connectors.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** Jira projects configured in `connectors.yaml` with explicit MCP server mapping per project (e.g., `mcp_server: jira-afianza`, `project_key: AFNZ`). Adding a new Jira instance = add MCP server + add entry to connectors.yaml.
- **D-02:** Each synced ticket becomes an individual markdown note in `06 Jira/` with frontmatter. Enables wikilinks like `[[TICKET-123]]`, Obsidian search, and graph connections.
- **D-03:** Synced fields: key, summary, status, assignee, priority, project (core) + sprint name, due date, created date (planning) + parent epic, linked issues, subtask list (relationships). No comments.
- **D-04:** Re-sync overwrites frontmatter fields silently. Jira is source of truth. No changelog in note body.
- **D-05:** Jira sync creates inbox items ONLY for newly assigned tickets or action-requiring status changes (not every update). Ticket notes in `06 Jira/` are always updated regardless.
- **D-06:** Integration via AppleScript (`osascript`). No extra dependencies, runs on any Mac. Claude executes directly via Bash.
- **D-07:** Read-only: pull reminders into inbox for GTD processing. Reminders app is a capture point (Siri, phone, watch). No write-back.
- **D-08:** Which Reminders lists to sync is configurable in `connectors.yaml`. Only specified lists are read.
- **D-09:** Existing sync-calendar, sync-granola, sync-slack remain untouched. Focus Phase 2 effort on new connectors (Jira, Reminders) and the unified inbox flow. Only modify existing skills if the inbox flow requires it.
- **D-10:** All connectors write directly to `00 Inbox.md` (Slack already does this). Single entry point. Morning ritual processes inbox with AI suggest & confirm.
- **D-11:** Dedup is wikilink-based. If a Jira ticket `[[TICKET-123]]` already exists as a vault note, don't create a duplicate inbox item for it from another source (e.g., Slack mention). Wikilinks are the dedup key.
- **D-12:** Granola already writes meeting notes + backlog items (not inbox). Calendar writes to cache. These patterns stay. Only Jira (new assignments), Slack (actions/decisions), and Reminders (captured items) write to inbox.

### Claude's Discretion
- Jira ticket note template (exact frontmatter fields and body structure)
- AppleScript implementation details for reading Reminders
- Exact inbox item format for Jira and Reminders entries
- Error handling and graceful degradation patterns (follow existing skill conventions)
- connectors.yaml schema for new sections (Jira, Reminders)

### Deferred Ideas (OUT OF SCOPE)
- Bidirectional Reminders sync (mark completed in vault -> complete in Reminders)
- Jira comment sync
- Smart merge of cross-source items (AI-based dedup beyond wikilinks)
- Jira status change changelog in note body
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SYNC-01 | User can sync Jira tickets to vault with dynamic project configuration | Jira MCP servers confirmed (jira-afianza, jira-previene). connectors.yaml schema designed. D-01 through D-05 define behavior. |
| SYNC-02 | User can sync Apple Reminders to vault inbox | osascript confirmed working. All 7 lists accessible. D-06 through D-08 define behavior. |
| SYNC-03 | User can sync calendar events to vault | Already implemented in sync-calendar skill. D-09 says untouched. No work needed. |
| SYNC-04 | User can sync Slack messages extracting actions and decisions | Already implemented in sync-slack skill. D-09 says untouched. No work needed. |
| SYNC-05 | User can sync meeting notes from Granola | Already implemented in sync-granola skill. D-09 says untouched. No work needed. |
| SYNC-06 | All synced items flow into a unified inbox for GTD processing | D-10 through D-12 define inbox flow. Wikilink-based dedup needed. Morning orchestrator wiring needed. |
</phase_requirements>

## Standard Stack

### Core (no new libraries)

This phase introduces NO new npm dependencies. All connectors are Claude Code skills (SKILL.md files) that use:

| Tool | Version | Purpose | Why Standard |
|------|---------|---------|--------------|
| mcp/atlassian (Docker) | FastMCP 2.14.5 | Jira data access | Already configured for both Jira instances. Provides jira_search (JQL), jira_get_issue tools. |
| osascript (macOS native) | System | Apple Reminders read access | Native macOS, no install needed. Confirmed working with full list access. |
| Obsidian vault (filesystem) | N/A | Data store | All output is markdown files. No database. |

### Available Jira MCP Servers

| Server Name | Instance | Docker Image |
|-------------|----------|-------------|
| `jira-afianza` | afianza-ac.atlassian.net | mcp/atlassian |
| `jira-previene` | adomenechreal.atlassian.net | mcp/atlassian |

### Available Reminders Lists (confirmed via osascript)

| List Name | Likely Use |
|-----------|-----------|
| Inbox | Quick captures from Siri/Watch/Phone |
| Work | Work-related reminders |
| Personal | Personal reminders |
| Salud | Health-related reminders |
| Someday/Maybe | GTD someday items |
| Waiting For | GTD waiting-for items |
| Cumples | Birthdays (probably skip) |

**Recommended default sync lists:** Inbox, Work, Personal (configurable via connectors.yaml per D-08).

## Architecture Patterns

### Skill File Structure

Each connector is a single `SKILL.md` file in its own directory under `skills/`. No code files, no build step. Claude reads the skill and executes it.

```
skills/
  sync-jira/
    SKILL.md           # New - Jira connector
  sync-reminders/
    SKILL.md           # New - Reminders connector
  sync-slack/
    SKILL.md           # Existing - untouched (D-09)
  sync-granola/
    SKILL.md           # Existing - untouched (D-09)
  sync-calendar/
    SKILL.md           # Existing - untouched (D-09)
  morning/
    SKILL.md           # Existing - update to wire new connectors
```

### Pattern 1: Connector Skill Structure (established by sync-slack)

Every sync skill follows this exact structure:

```markdown
## Step 0 -- Load configuration
Read config.yaml, set VAULT path.

## Overview
What this connector does.

## Prerequisites
What must be available (MCP server, tool, etc.)

## Process
### 1. Read config (from connectors.yaml)
### 2. Fetch data (from external source)
### 3. Enrich with wikilinks (people, projects)
### 4. Write output (to inbox / cache / notes)
### 5. Graceful degradation table

## Output
What files are created/modified.
```

### Pattern 2: Jira Ticket Note Template (Claude's Discretion)

Recommended frontmatter and body structure for `06 Jira/{KEY}.md`:

```markdown
---
key: TICKET-123
summary: "Ticket title"
status: In Progress
assignee: Alfonso Domenech
priority: Medium
project: AFNZ
sprint: Sprint 42
due_date: 2026-04-15
created: 2026-03-01
epic: TICKET-100
type: Story
last_sync: 2026-03-30T09:00:00
tags: jira
---
# TICKET-123: Ticket title

**Status:** In Progress | **Priority:** Medium | **Sprint:** Sprint 42

## Relationships

- **Epic:** [[TICKET-100]]
- **Linked issues:** [[TICKET-456]], [[TICKET-789]]
- **Subtasks:**
  - [[TICKET-124]] - Subtask description
  - [[TICKET-125]] - Another subtask
```

Key design choices:
- `last_sync` frontmatter field enables staleness detection (Success Criteria 4)
- `tags: jira` enables Obsidian filtering and Dataview queries
- Relationships section uses wikilinks for graph connectivity
- Body is minimal -- Jira is source of truth, note is a reference/link point
- Re-sync overwrites entire frontmatter (D-04); body relationships section also regenerated

### Pattern 3: Inbox Item Format

Established by sync-slack:
```markdown
- Slack #channel -- [[Persona]]: description (YYYY-MM-DD)
```

Recommended for new connectors:
```markdown
- Jira [[TICKET-123]] -- Asignado: summary (YYYY-MM-DD)
- Jira [[TICKET-456]] -- Status changed to "In Review": summary (YYYY-MM-DD)
- Reminder (list) -- reminder text (YYYY-MM-DD)
```

### Pattern 4: Wikilink-Based Dedup (D-11)

Before writing an inbox item, scan existing `00 Inbox.md` for the primary wikilink:
- Jira: check if `[[TICKET-123]]` already in inbox
- Reminders: check if exact reminder text already in inbox (no wikilink key available)
- Cross-source: if `[[TICKET-123]]` exists as a vault note in `06 Jira/`, don't create inbox item from Slack mention of same ticket

Implementation: simple string search in inbox content for the wikilink pattern.

### Pattern 5: Connectors Config Schema (Claude's Discretion)

Recommended `config/connectors.yaml` structure:

```yaml
# Slack channels (existing pattern)
slack:
  channels:
    - name: general
      extract: [decisions, actions]
    - name: dev-team
      extract: [actions, questions]

# Jira projects (new - D-01)
jira:
  projects:
    - mcp_server: jira-afianza
      project_key: AFNZ
      assignee_filter: "Alfonso Domenech"     # Only sync tickets assigned to this name
      sync_statuses: ["To Do", "In Progress", "In Review"]  # Which statuses to sync
    - mcp_server: jira-previene
      project_key: PREV
      assignee_filter: "Alfonso Domenech"
      sync_statuses: ["To Do", "In Progress", "In Review"]

# Apple Reminders (new - D-08)
reminders:
  lists:
    - "Inbox"
    - "Work"
    - "Personal"
  completed: false   # Only sync incomplete reminders
```

### Anti-Patterns to Avoid

- **Direct Jira HTTP API calls:** Use MCP servers exclusively. They handle authentication and Docker isolation.
- **Storing sync state in memory:** All state must be in vault files (filesystem is the database). Use `last_sync` frontmatter and processed registries.
- **Modifying existing sync skills:** D-09 explicitly says leave them untouched unless inbox flow requires changes.
- **Complex dedup algorithms:** Wikilink string matching is sufficient (D-11). No AI-based dedup (deferred).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Jira API access | HTTP client with auth | MCP servers (jira-afianza, jira-previene) | Already configured, handles auth, Docker isolated |
| Reminders access | Third-party library | osascript (AppleScript) | Native macOS, zero dependencies, confirmed working |
| JQL query building | Custom query builder | Raw JQL strings in MCP tool calls | JQL is simple enough: `project = X AND assignee = Y` |
| Markdown frontmatter | Custom parser | gray-matter pattern (read/write manually in skill) | Skills are instruction files, not code -- Claude parses markdown natively |
| Dedup across sources | Graph database or index | String search for wikilinks in inbox file | Inbox is a single small file, grep is instant |

## Common Pitfalls

### Pitfall 1: Jira MCP Tool Failures
**What goes wrong:** The mcp/atlassian Docker container occasionally fails JQL queries or times out.
**Why it happens:** Docker cold starts, network issues, FastMCP version bugs (reported Jan 2026).
**How to avoid:** Graceful degradation per established pattern. Wrap MCP calls in try/catch logic. If Jira sync fails, log warning, continue with other syncs. Never abort morning ritual.
**Warning signs:** Empty results when tickets are known to exist; error messages mentioning FastMCP.

### Pitfall 2: Reminders Permission Dialog
**What goes wrong:** First osascript call triggers macOS permission dialog, which may not be visible in CLI context.
**Why it happens:** macOS requires explicit permission for automation access to Reminders.
**How to avoid:** Document that user must grant permission on first run. The skill should detect "not authorized" errors and provide clear instructions.
**Warning signs:** osascript returns error code 1 with "not authorized" message.

### Pitfall 3: Inbox Dedup Race Condition
**What goes wrong:** Two connectors write the same item to inbox because they run in sequence and both check before either writes.
**Why it happens:** Morning orchestrator runs connectors sequentially but dedup check happens before write.
**How to avoid:** This is actually a non-issue because morning orchestrator runs connectors sequentially (not parallel). Each connector reads current inbox state before writing. The sequence is: Jira writes -> Slack writes (checks what Jira already wrote) -> Reminders writes (checks what both wrote).
**Warning signs:** Duplicate wikilinks in inbox.

### Pitfall 4: Jira Ticket Key Collision Across Instances
**What goes wrong:** Two Jira instances could theoretically have overlapping ticket keys if project keys overlap.
**Why it happens:** Project keys are instance-scoped, not globally unique.
**How to avoid:** Use unique project keys per instance in connectors.yaml. Current setup (AFNZ, PREV) has no collision risk. If collision occurs, prefix with instance name in filename.
**Warning signs:** File overwrites in `06 Jira/` directory.

### Pitfall 5: AppleScript Date Parsing
**What goes wrong:** osascript returns dates in locale-dependent format ("Monday, 9 February 2026 at 09:00:30"), which varies by macOS language settings.
**Why it happens:** AppleScript uses system locale for date formatting.
**How to avoid:** Use `date string of (current date)` cautiously. For inbox items, only the creation date matters (for context), and the format "YYYY-MM-DD" should be constructed explicitly in the AppleScript or parsed by Claude.
**Warning signs:** Dates appearing in unexpected formats in inbox items.

### Pitfall 6: Large Jira Result Sets
**What goes wrong:** JQL query returns hundreds of tickets, overwhelming the sync process.
**Why it happens:** Broad filters (e.g., all tickets in a project regardless of status).
**How to avoid:** Use `sync_statuses` filter in connectors.yaml. Default to active statuses only (To Do, In Progress, In Review). Completed/Closed tickets are not synced unless explicitly configured.
**Warning signs:** Sync taking unusually long, excessive file creation in `06 Jira/`.

## Code Examples

### AppleScript: Read Incomplete Reminders from a List

Verified working on this machine (2026-03-30):

```applescript
tell application "Reminders"
    set theList to list "Inbox"
    set output to ""
    repeat with r in (reminders of theList whose completed is false)
        set rName to name of r
        set rDate to creation date of r as string
        set rBody to "none"
        try
            set rBody to body of r
        end try
        set output to output & rName & " | " & rDate & " | " & rBody & linefeed
    end repeat
    return output
end tell
```

**Available Reminders properties:** name, creation date, due date (optional), priority (0=none, 1=high, 5=medium, 9=low), body (optional), completed (boolean).

### AppleScript: List All Reminders Lists

```applescript
tell application "Reminders" to get name of every list
```

Returns: `Inbox, Cumples, Work, Personal, Salud, Someday/Maybe, Waiting For`

### Jira MCP: Search Tickets (JQL)

The MCP tool call pattern (Claude executes via MCP):

```
Tool: mcp__jira-afianza__jira_search
Parameters:
  jql: "project = AFNZ AND assignee = 'Alfonso Domenech' AND status in ('To Do', 'In Progress', 'In Review')"
```

### Jira MCP: Get Single Ticket Details

```
Tool: mcp__jira-afianza__jira_get_issue
Parameters:
  issue_key: "AFNZ-123"
```

### Inbox Append Pattern (from sync-slack)

```markdown
- Jira [[AFNZ-123]] -- Asignado: Implementar feature X (2026-03-30)
- Reminder (Work) -- Responder Luis TMA (2026-03-30)
```

### Wikilink Dedup Check

Before appending to inbox, search for the wikilink in current inbox content:
- For Jira: search for `[[AFNZ-123]]` in `00 Inbox.md`
- For Reminders: search for exact reminder text (normalized) in `00 Inbox.md`
- Cross-source: search for `[[TICKET-KEY]]` in `00 Inbox.md` before adding from any source

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Jira REST API with personal tokens | MCP servers (mcp/atlassian via Docker) | 2025 | No direct API code needed; MCP handles auth and transport |
| Reminders via third-party npm packages | AppleScript via osascript | Always available | Zero dependencies, native macOS |
| Separate sync scripts per connector | Claude Code skills (SKILL.md) | Life OS design | No build step, no runtime, Claude executes instructions directly |

## Open Questions

1. **Jira MCP tool exact parameter names**
   - What we know: Tools are `jira_search` (JQL) and `jira_get_issue` (by key). FastMCP 2.14.5.
   - What's unclear: Exact parameter names and response shapes. May be `jql` or `query` for search.
   - Recommendation: Have the skill try `jira_search` first and document the actual parameter names on first successful call. Graceful degradation handles discovery failures.

2. **Inbox item format for "action-requiring status changes" (D-05)**
   - What we know: New assignments trigger inbox items. Ticket notes always updated.
   - What's unclear: How to detect "action-requiring" status changes vs. routine updates. Requires tracking previous status.
   - Recommendation: Store `status` in Jira note frontmatter. On re-sync, compare new status vs. stored status. Trigger inbox item only for transitions TO statuses like "In Review" or "To Do" (configurable).

3. **Reminders dedup key**
   - What we know: Jira uses wikilinks ([[TICKET-123]]) for dedup. Reminders have no unique external ID exposed via AppleScript.
   - What's unclear: How to avoid re-adding the same reminder to inbox on next sync.
   - Recommendation: Use a processed registry file (like granola-processed.md) at `.cache/reminders-processed.md`. Store reminder name + creation date as the dedup key. Mark reminders as processed after adding to inbox.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| osascript | sync-reminders | Yes | macOS native | -- |
| Docker | Jira MCP servers | Yes (assumed, servers configured) | -- | -- |
| mcp/atlassian | sync-jira | Yes (configured) | FastMCP 2.14.5 | Skip Jira sync gracefully |
| icalBuddy | sync-calendar (existing) | Yes | /opt/homebrew/bin | -- |
| Reminders.app | sync-reminders | Yes | macOS native | -- |
| Slack MCP | sync-slack (existing) | Configured | -- | -- |
| Granola MCP | sync-granola (existing) | Configured | -- | -- |

**Missing dependencies with no fallback:** None.

**Missing dependencies with fallback:** None -- all dependencies are available.

## Sources

### Primary (HIGH confidence)
- Verified osascript Reminders access on local machine -- returned all 7 lists and reminder properties (name, date, priority, body)
- Verified Jira MCP server configuration in `~/.claude.json` -- both jira-afianza and jira-previene configured with mcp/atlassian Docker
- Existing skill files (sync-slack, sync-granola, sync-calendar, morning) -- established patterns confirmed
- `config.example.yaml` -- vault structure paths confirmed, connectors_config path confirmed

### Secondary (MEDIUM confidence)
- [mcp/atlassian Docker Hub](https://hub.docker.com/r/mcp/atlassian) -- Docker image details
- [sooperset/mcp-atlassian GitHub](https://github.com/sooperset/mcp-atlassian) -- Tool names (jira_search, jira_get_issue) confirmed via README

### Tertiary (LOW confidence)
- Jira MCP tool exact parameter schemas -- inferred from community docs, needs runtime verification

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All tools verified locally (osascript tested, MCP servers confirmed in config)
- Architecture: HIGH - Follows established skill patterns from Phase 1 with no architectural changes
- Pitfalls: MEDIUM - Jira MCP reliability based on community reports; AppleScript permission issue is well-known macOS behavior
- Connectors schema: MEDIUM - Designed to match existing slack pattern; exact fields are Claude's discretion

**Research date:** 2026-03-30
**Valid until:** 2026-04-30 (stable -- no fast-moving dependencies)
