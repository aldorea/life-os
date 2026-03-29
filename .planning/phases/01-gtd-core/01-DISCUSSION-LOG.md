# Phase 1: GTD Core - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-29
**Phase:** 01-gtd-core
**Areas discussed:** Vault restructuring strategy, Skill gap analysis & upgrade path, Goal tracking model, Capture & processing flow

---

## Vault Restructuring Strategy

### When to restructure

| Option | Description | Selected |
|--------|-------------|----------|
| Restructure now | Define target folder structure and frontmatter schemas upfront. Migrate existing content. | ✓ |
| Minimal changes now, restructure later | Keep current structure mostly intact. Only add missing folders. Full restructure after 4 weeks. | |
| Freeze structure as-is | Current structure works. Just document conventions and standardize frontmatter. | |

**User's choice:** Restructure now
**Notes:** Clean foundation for all future phases.

### Folder organization

| Option | Description | Selected |
|--------|-------------|----------|
| Keep numbered PARA-ish | Keep current numbered prefix pattern (01, 02...) with PARA-inspired categories. | ✓ |
| Pure PARA | Projects / Areas / Resources / Archive — strict PARA. | |
| Functional grouping | Group by function: GTD/, Rituals/, People/, Knowledge/, Goals/. | |

**User's choice:** Keep numbered PARA-ish
**Notes:** Familiar pattern, Obsidian sidebar stays ordered.

### Frontmatter strictness

| Option | Description | Selected |
|--------|-------------|----------|
| Strict per note type | Each note type has defined required frontmatter schema. Skills validate on read/write. | ✓ |
| Flexible with conventions | Document recommended fields but don't enforce. Skills tolerate missing ones. | |
| Minimal frontmatter | Only tags and date. Rely on folder location and content parsing. | |

**User's choice:** Strict per note type
**Notes:** Enables reliable queries and future dashboard.

### Migration approach

| Option | Description | Selected |
|--------|-------------|----------|
| Auto-migrate with backup | Git commit current state, then auto-migrate: move files, add frontmatter, update wikilinks. | ✓ |
| Manual migration with guide | Generate a migration guide/checklist. User moves files themselves. | |
| Gradual migration | New notes use new structure. Old notes stay where they are. Migrate naturally over time. | |

**User's choice:** Auto-migrate with backup
**Notes:** One-time migration skill.

---

## Skill Gap Analysis & Upgrade Path

### Morning ritual architecture

| Option | Description | Selected |
|--------|-------------|----------|
| Orchestrator that chains existing | Morning calls sync-calendar, sync-jira, sync-slack, sync-granola, then process-inbox, then today. | ✓ |
| Standalone monolith | Morning is self-contained — does all syncing, processing, and daily note inline. | |
| User chains manually | No morning skill. User runs each skill themselves. | |

**User's choice:** Orchestrator that chains existing
**Notes:** Reuses existing skills. One command to start the day.

### Weekly review interactivity

| Option | Description | Selected |
|--------|-------------|----------|
| Guided interactive | Walk through GTD review steps with questions at each step. User confirms. | ✓ |
| Semi-auto with checkpoints | Auto-generate most content but pause at 2-3 key decision points. | |
| Fully automated report | Generate the whole weekly review as a report. No interaction. | |

**User's choice:** Guided interactive
**Notes:** Should feel like a conversation with a facilitator.

### GTD view organization

| Option | Description | Selected |
|--------|-------------|----------|
| Separate skills each | Dedicated skills: /next-actions, /projects, /someday. One skill per concern. | ✓ |
| Single /gtd skill with subcommands | /gtd next, /gtd projects, /gtd someday. One entry point. | |
| Obsidian-native views only | Use Dataview/Tasks plugin queries in template notes instead of CLI skills. | |

**User's choice:** Separate skills each
**Notes:** Matches existing pattern of one-skill-per-concern.

### Existing skill upgrades

| Option | Description | Selected |
|--------|-------------|----------|
| Upgrade alongside vault migration | Update all existing skills to use new paths, schemas, and conventions. | ✓ |
| Upgrade incrementally | Keep existing skills working as-is. Upgrade one by one post-migration. | |
| Backward-compatible layer | Add a compatibility layer that maps old paths to new paths. | |

**User's choice:** Upgrade alongside vault migration
**Notes:** Everything consistent from day one.

---

## Goal Tracking Model

### Goal storage

| Option | Description | Selected |
|--------|-------------|----------|
| Single goals.yaml | All goals in one YAML file with structured fields. Simple, one file to manage. | ✓ |
| One markdown file per goal | Each goal is a separate .md file with frontmatter. More Obsidian-native. | |
| Goals in frontmatter of project notes | Goals live as frontmatter in related project notes. | |

**User's choice:** Single goals.yaml
**Notes:** Matches current config pattern.

### Progress history

| Option | Description | Selected |
|--------|-------------|----------|
| History array in goals.yaml | Each goal has history[] array with dated snapshots: [{date, value, note}]. | ✓ |
| Separate progress log file | A goal-progress.yaml with timestamped entries. | |
| Extract from daily notes | Reconstruct history by scanning dailies. | |

**User's choice:** History array in goals.yaml
**Notes:** Weekly review appends a snapshot automatically.

### Goal dimensions

| Option | Description | Selected |
|--------|-------------|----------|
| Flat list with tags | All goals in one flat list. dimension field + horizon field. Filter by dimension in views. | ✓ |
| Nested by quarter | Goals grouped under quarters (Q1-2026, Q2-2026). | |
| OKR-style hierarchy | Objectives contain Key Results, each with own metric and target. | |

**User's choice:** Flat list with tags
**Notes:** Simple, covers professional + personal + health.

### Quarterly reflection format

| Option | Description | Selected |
|--------|-------------|----------|
| Interactive CLI workflow | Guided reflection: review progress, start/stop/continue, set next quarter goals. | ✓ |
| Auto-generated report + manual reflection | Skill generates report. User writes reflection manually. | |
| Obsidian template only | Provide template note. User fills in manually. | |

**User's choice:** Interactive CLI workflow
**Notes:** Mirrors weekly review pattern but at goal level.

---

## Capture & Processing Flow

### Capture paths for Phase 1

| Option | Description | Selected |
|--------|-------------|----------|
| CLI only for Phase 1 | Focus on making /dump and /capture as fast as possible. Defer other paths. | ✓ |
| CLI + Obsidian QuickAdd plugin | Two paths: CLI + QuickAdd. Requires documenting setup. | |
| CLI + Apple Shortcuts | Create Apple Shortcut that appends to Inbox.md. Capture from anywhere. | |

**User's choice:** CLI only for Phase 1
**Notes:** Keep scope tight. QuickAdd and Shortcuts are future phases.

### Vault mutation safety

| Option | Description | Selected |
|--------|-------------|----------|
| Git auto-commit before mutations | Every skill that writes to vault commits affected files first. Rollback = git revert. | ✓ |
| Manual git backup | User runs git commit themselves before mutating skills. | |
| Obsidian version history only | Rely on Obsidian's built-in file recovery. No git. | |

**User's choice:** Git auto-commit before mutations
**Notes:** Automatic and transparent.

### AI processing aggressiveness

| Option | Description | Selected |
|--------|-------------|----------|
| Suggest & confirm | AI reformulates, suggests tags, detects duplicates, presents plan. User confirms. | ✓ |
| Auto-process with undo | AI processes everything automatically. Shows what it did. User can undo. | |
| Classify only, user moves | AI only suggests classification. User manually moves items. | |

**User's choice:** Suggest & confirm
**Notes:** Matches current process-inbox behavior.

### GTD context tags

| Option | Description | Selected |
|--------|-------------|----------|
| Hashtag contexts | #home, #office, #calls, #computer as tags on tasks. Filter with /next-actions #home. | ✓ |
| Frontmatter field | 'context' frontmatter field per task note. | |
| Folder-based contexts | Separate folders per context. | |

**User's choice:** Hashtag contexts
**Notes:** Matches existing tag pattern. Added to config.tags.contexts.

---

## Claude's Discretion

- Migration skill implementation details
- Exact folder numbering and naming
- Frontmatter field names and types per note type
- Morning orchestrator internal architecture

## Deferred Ideas

- Obsidian QuickAdd plugin integration for capture
- Apple Shortcuts for capture from anywhere
- Apple Reminders capture path (Phase 2)
- Web dashboard for goal visualization (v2)
