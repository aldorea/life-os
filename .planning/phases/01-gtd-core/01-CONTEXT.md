# Phase 1: GTD Core - Context

**Gathered:** 2026-03-29
**Status:** Ready for planning

<domain>
## Phase Boundary

Complete daily productivity loop: vault schema, capture, processing, rituals, and goal tracking — zero external dependencies. User has a reliable GTD system that works entirely from local vault + CLI skills.

This phase covers: vault restructuring, frontmatter schemas, tagging conventions, content migration, all GTD workflows (capture, process, next actions, projects, someday/maybe, stalled detection), daily/weekly/quarterly rituals, and goal tracking with history.

This phase does NOT cover: external sync connectors (Phase 2), CRM (Phase 3), content pipeline (Phase 3), web dashboard (v2).

</domain>

<decisions>
## Implementation Decisions

### Vault Restructuring
- **D-01:** Restructure vault NOW in Phase 1 — define target folder structure and frontmatter schemas upfront, migrate existing content. Clean foundation for all future phases.
- **D-02:** Keep numbered PARA-ish folder convention (01, 02, 03...) with PARA-inspired categories. Rename/reorganize within that convention. Familiar pattern, Obsidian sidebar stays ordered.
- **D-03:** Strict frontmatter schemas per note type (task, person, meeting, goal, knowledge). Skills validate on read/write. Required fields enforced — enables reliable queries and future dashboard.
- **D-04:** Auto-migrate existing vault content with git backup. Git commit current state first, then auto-migrate: move files to new folders, add frontmatter to existing notes, update wikilinks. One-time migration skill.

### Skill Architecture
- **D-05:** Morning ritual (RITUAL-02) is an orchestrator that chains existing skills: sync-calendar → sync-jira → sync-slack → sync-granola → process-inbox → today. One command to start the day.
- **D-06:** Weekly review (RITUAL-03) is a guided interactive facilitator — walks through GTD review steps: retrospective → inbox zero check → backlog health → stalled projects → someday/maybe review → next week planning. Ask questions at each step, user confirms.
- **D-07:** Missing GTD views (next actions by context, projects list, someday/maybe) are separate dedicated skills: /next-actions, /projects, /someday. One skill per concern, matching existing pattern.
- **D-08:** Existing skills (capture, today, process-inbox, week, status, etc.) upgraded alongside vault migration to use new paths, new frontmatter schemas, and new conventions. Everything consistent from day one.

### Goal Tracking
- **D-09:** Goals stored in single goals.yaml file with structured fields (name, metric, target, weight, deadline, status, dimension, horizon, history[]).
- **D-10:** Goal dimensions organized as flat list with tags — each goal has a 'dimension' field (professional/personal/health) and a 'horizon' field (quarterly/annual). Filter by dimension in views.
- **D-11:** Goal progress history tracked as history[] array in goals.yaml — each entry is {date, value, note}. Weekly review appends a snapshot automatically.
- **D-12:** Quarterly reflection (GOAL-03) is an interactive CLI workflow — guided reflection: review each goal's progress, start/stop/continue analysis, set next quarter goals. Generates a quarterly note in the vault.

### Capture & Processing
- **D-13:** Phase 1 capture is CLI only (/dump and /capture). Obsidian QuickAdd and Apple Shortcuts deferred to future phases. Focus on making CLI path as fast as possible.
- **D-14:** Git auto-commit before vault mutations. Every skill that writes to the vault does a git commit of affected files first. Rollback = git revert. Automatic and transparent.
- **D-15:** AI inbox processing follows suggest & confirm pattern. AI reformulates, suggests tags, detects duplicates, presents a plan. User reviews and confirms before any changes. Matches current process-inbox behavior.
- **D-16:** GTD contexts use hashtag tags: #home, #office, #calls, #computer. Added to config.tags.contexts. Filter with /next-actions #home. Matches existing tag pattern.

### Claude's Discretion
- Implementation details of the migration skill (file traversal, wikilink updating strategy)
- Exact folder numbering and naming for new/renamed folders
- Frontmatter field names and types per note type (as long as they're strict and documented)
- Internal architecture of how skills chain in the morning orchestrator

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Configuration
- `config.example.yaml` — Current vault folder structure mapping, tag definitions, backlog sections, all config schemas
- `.planning/PROJECT.md` — Core value, constraints, key decisions, context about existing system
- `.planning/REQUIREMENTS.md` — VAULT-01 through VAULT-04, GTD-01 through GTD-06, RITUAL-01 through RITUAL-04, GOAL-01 through GOAL-05

### Existing Skills (patterns to follow and upgrade)
- `skills/capture/SKILL.md` — Capture skill pattern (GTD-01 baseline)
- `skills/process-inbox/SKILL.md` — Inbox processing pattern (GTD-02 baseline)
- `skills/today/SKILL.md` — Daily note generation pattern (RITUAL-01 baseline)
- `skills/week/SKILL.md` — Weekly note generation pattern (RITUAL-03 baseline)
- `skills/status/SKILL.md` — Status dashboard pattern (GOAL-02 baseline)
- `skills/content/SKILL.md` — Content skill (for knowledge capture pattern)
- `skills/prep/SKILL.md` — Meeting prep pattern (CRM-03 baseline, Phase 3)
- `skills/shop/SKILL.md` — Shopping skill (for config.yaml loading pattern)
- `skills/train/SKILL.md` — Training skill (for YAML data parsing pattern)

### Sync Skills (called by morning orchestrator)
- `skills/sync-calendar/SKILL.md` — Calendar sync pattern
- `skills/sync-granola/SKILL.md` — Granola sync pattern
- `skills/sync-slack/SKILL.md` — Slack sync pattern

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **config.yaml loading pattern**: Every skill starts with Step 0 loading config.yaml. Reuse this in all new skills.
- **Graceful degradation pattern**: Skills like `today` and `status` handle missing data sources gracefully. Apply same pattern to all new skills.
- **Wikilink convention**: People as `[[Name]]`, projects as `[[ProjectName]]`, Jira as `[[TICKET-123]]`. All new skills must follow this.
- **Obsidian Tasks plugin queries**: `today` and `week` use `tasks` code blocks for live queries. New views should follow this pattern where applicable.
- **Spanish-language content**: All vault content in Spanish. Skill prompts and user-facing text in Spanish.

### Established Patterns
- **One SKILL.md per skill**: Each skill is a single markdown file with frontmatter (name, description) and step-by-step instructions.
- **Config-driven paths**: All vault paths resolved through config.yaml structure section. No hardcoded paths.
- **Present-then-confirm**: Mutating skills show plan first, ask confirmation, then apply. Never auto-mutate.
- **YAML frontmatter + markdown body**: Notes use YAML frontmatter for structured data, markdown body for content.

### Integration Points
- **config.yaml**: Central config file — new folders, new tags, new goal fields all go here
- **Backlog.md**: Central task list — new GTD views read from this
- **Inbox.md**: Capture target — capture skills write here, process-inbox reads and clears
- **goals.yaml**: Goal data — to be defined with schema from D-09/D-10/D-11
- **Daily/Weekly notes**: Generated by today/week skills — morning orchestrator triggers these

</code_context>

<specifics>
## Specific Ideas

- Morning ritual = single `/morning` command that chains all syncs + inbox + daily note
- Weekly review should feel like a conversation with a facilitator, not a report generator
- Quarterly reflection mirrors weekly review pattern but at goal level (start/stop/continue)
- GTD contexts match David Allen's classic contexts but adapted: #home, #office, #calls, #computer
- Stalled project detection: no next action OR no progress in X days (X configurable)
- Migration should be a one-time `/migrate-vault` skill that can be run once and discarded

</specifics>

<deferred>
## Deferred Ideas

- Obsidian QuickAdd plugin integration for capture — future phase
- Apple Shortcuts for capture from anywhere on macOS/iOS — future phase
- Apple Reminders capture path — Phase 2 (SYNC-02)
- Web dashboard for goal visualization — v2 (DASH-01)

None beyond the above — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-gtd-core*
*Context gathered: 2026-03-29*
