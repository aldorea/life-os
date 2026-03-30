# Vault Schema - Canonical Reference

**Created:** 2026-03-30
**Phase:** 01-gtd-core, Plan 01
**Purpose:** Single source of truth for vault folder structure, frontmatter schemas, tagging conventions, and migration map. All downstream plans reference this document.

---

## Target Folder Structure

Lower numbers = higher frequency access. Inbox and Backlog at top (00, 01). Daily use folders (Projects, Daily, Weekly) in 02-04. Reference folders (People, Jira, Meetings) in 05-07. Knowledge and growth in 08-10. Archive last. Cache files hidden in `.cache/` since they are not user-facing.

```
vault/
  00 Inbox.md                      # Quick capture landing zone (GTD inbox)
  01 Backlog.md                    # All tasks: sections by context/project, includes Someday/Maybe
  02 Projects/                     # Active project notes (PARA: Projects)
  03 Daily/                        # Daily notes (DD-MM-YYYY format)
  04 Weekly/                       # Weekly notes (YYYY-W## format)
  05 People/                       # Person notes / mini-CRM (PARA: Areas - relationships)
  06 Jira/                         # Synced Jira ticket notes
  07 Meetings/                     # Meeting notes (from Granola or manual)
  08 Knowledge/                    # Knowledge base notes (evergreen/growing)
    content-ideas.md               # Content pipeline ideas
  09 Health/                       # Health tracking
    training-log.md                # Gym/training log
    menus/                         # Meal plans
  10 Content/                      # Content drafts (LinkedIn, blog, etc.)
  11 Archive/                      # Completed/inactive projects and notes (PARA: Archive)
  config/
    goals.yaml                     # Goal definitions with metrics and history
    constraints.yaml               # Work hours, hard stops
    connectors.yaml                # Sync connector configurations
    voice.md                       # Writing voice profile
  .cache/
    calendar-cache.md              # Calendar sync cache (not user-facing)
    granola-processed.md           # Processed Granola meeting IDs
    claude-memory/                 # Claude memory/context files
```

### Folder Numbering Rationale

| Number | Folder | Access Frequency | PARA Category |
|--------|--------|-----------------|---------------|
| 00 | Inbox.md | Multiple times daily | Inbox |
| 01 | Backlog.md | Multiple times daily | Actions |
| 02 | Projects/ | Daily | Projects |
| 03 | Daily/ | Daily | Resources |
| 04 | Weekly/ | Weekly | Resources |
| 05 | People/ | Several times weekly | Areas |
| 06 | Jira/ | Daily (sync) | Resources |
| 07 | Meetings/ | Several times weekly | Resources |
| 08 | Knowledge/ | Weekly | Resources |
| 09 | Health/ | Several times weekly | Areas |
| 10 | Content/ | Weekly | Projects |
| 11 | Archive/ | Rarely | Archive |

---

## Frontmatter Schemas

Strict schemas per note type. Required fields marked with `*`, optional with `?`.

### Daily Note

```yaml
---
tags: daily          # * note type identifier
date: 2026-03-29     # * ISO date format (YYYY-MM-DD)
week: 2026-W13       # * ISO week format
day: Saturday        # * English day name
---
```

**File location:** `{vault}/{config.structure.daily_notes}/DD-MM-YYYY.md`
**File naming:** Uses `config.daily.date_format` (default `DD-MM-YYYY`)

### Weekly Note

```yaml
---
tags: weekly            # * note type identifier
week: 2026-W13          # * ISO week format
date_start: 2026-03-24  # * Monday of the week
date_end: 2026-03-30    # * Sunday of the week
---
```

**File location:** `{vault}/{config.structure.weekly_notes}/YYYY-W##.md`

### Person

```yaml
---
tags: person                     # * note type identifier
name: Full Name                  # * person's full name
company: Company Name            # ? employer/organization
role: Role Title                 # ? job title or role
last_interaction: 2026-03-29     # * updated by sync skills on interaction
created: 2026-03-29              # * date the note was created
---
```

**File location:** `{vault}/{config.structure.people}/Name.md`
**Wikilink convention:** `[[Full Name]]`

### Meeting

```yaml
---
tags: meeting               # * note type identifier
date: 2026-03-29            # * ISO date of the meeting
attendees:                  # * list of attendee names
  - Name One
  - Name Two
source: granola|manual      # * origin of the meeting note
project: ProjectName        # ? associated project (if any)
---
```

**File location:** `{vault}/{config.structure.meetings}/YYYY-MM-DD - Meeting Title.md`
**Wikilink convention:** Attendees as `[[Name]]`, project as `[[ProjectName]]`

### Knowledge Note

```yaml
---
tags: knowledge                  # * note type identifier
topic: Topic Name                # * subject of the knowledge note
last_updated: 2026-03-29         # * last time content was added/revised
entries: 5                       # * count of entries/sections
maturity: growing                # * seed | growing | ready
---
```

**File location:** `{vault}/{config.structure.knowledge}/Topic Name.md`
**Maturity levels:**
- `seed` -- initial capture, minimal content
- `growing` -- actively being developed, multiple entries
- `ready` -- comprehensive, stable reference

### Goal (in goals.yaml)

Goals are stored in `{vault}/config/goals.yaml`, NOT as individual markdown files.

```yaml
goals:
  - id: goal-unique-id       # * unique identifier (kebab-case)
    name: "Goal name"        # * human-readable goal description
    dimension: professional   # * professional | personal | health
    horizon: quarterly        # * quarterly | annual
    metric: "Metric name"    # * what is being measured
    target: 100              # * numeric target value
    current: 45              # * current numeric value
    unit: "%"                # ? unit of measurement
    weight: 0.3              # * 0.0-1.0, weights within same horizon sum to 1.0
    deadline: 2026-06-30     # * target completion date (ISO format)
    status: in_progress      # * not_started | in_progress | completed | abandoned
    history:                 # progress snapshots (appended by weekly review)
      - date: 2026-03-22    # * snapshot date
        value: 40            # * value at that point
        note: "Description"  # * context for the change
```

**Validation rules:**
- All `weight` values within the same `horizon` should sum to 1.0
- `current` should be between 0 and `target` (warn if exceeded)
- `history` entries should be chronologically ordered
- `id` must be unique across all goals

### Task (inline in Backlog.md)

Tasks are NOT individual files. They live as checkbox lines in Backlog.md with inline tags:

```
- [ ] Task description #project #context #actionability
- [x] Completed task #project #context 2026-03-29
```

**Tag usage in tasks:**
- Project tag: `#miportal`, `#sherpa`, etc.
- Context tag: `#home`, `#office`, `#calls`, `#computer`
- Actionability: `#next`, `#esperando`, `#someday`
- Priority: `#urgente`, `#importante`

---

## Tagging Convention

All tags use the `#hashtag` format for Obsidian compatibility.

### GTD Contexts

Tags that indicate WHERE or HOW a task can be done (per David Allen's GTD methodology):

| Tag | Meaning | Use when |
|-----|---------|----------|
| `#home` | At home | Task requires being at home |
| `#office` | At the office | Task requires office presence/resources |
| `#calls` | Phone/video calls | Task is a call to make |
| `#computer` | At any computer | Task only needs a computer (location-independent) |

### Project Tags

Tags that associate items with a specific project:

| Tag | Project |
|-----|---------|
| `#miportal` | Mi Portal project |
| `#sherpa` | Sherpa project |
| `#previene` | Previene project |
| `#orbitant` | Orbitant project |
| `#lifeos` | Life OS project |
| `#salud` | Health/wellness project |

### Priority Tags

| Tag | Meaning |
|-----|---------|
| `#urgente` | Time-sensitive, needs immediate attention |
| `#importante` | High impact, but not necessarily urgent |

### Actionability Tags

| Tag | GTD List | Meaning |
|-----|----------|---------|
| `#next` | Next Actions | Ready to do now, no blockers |
| `#esperando` | Waiting For | Blocked on someone else's action |
| `#someday` | Someday/Maybe | Want to do eventually, not committed |

### Status Tags (for projects)

| Tag | Meaning |
|-----|---------|
| `#active` | Currently being worked on |
| `#stalled` | No progress in >{config.projects.stalled_days} days or no next action |
| `#completed` | Finished, ready for archive |
| `#archived` | Moved to 11 Archive/ |

### Note Type Tags

Used in frontmatter `tags` field to identify note type:

| Tag | Note Type | Location |
|-----|-----------|----------|
| `daily` | Daily note | 03 Daily/ |
| `weekly` | Weekly note | 04 Weekly/ |
| `person` | Person/CRM note | 05 People/ |
| `meeting` | Meeting note | 07 Meetings/ |
| `knowledge` | Knowledge note | 08 Knowledge/ |

---

## Migration Map

Old path to new path mapping for the one-time vault migration skill (`/migrate-vault`).

**Critical safety note:** Obsidian resolves wikilinks by filename, not path. Moving files to new folders does NOT break `[[wikilinks]]` as long as filenames stay the same. Only renaming files would break links.

| Old Path | New Path | Action | Notes |
|----------|----------|--------|-------|
| `02 Inbox.md` | `00 Inbox.md` | Rename | Top of sidebar for quick access |
| `05 Projects/` | `02 Projects/` | Rename folder | Move up -- daily use |
| `09 People/` | `05 People/` | Rename folder | Move up -- frequent access |
| `08 Resources/meetings/` | `07 Meetings/` | Move + promote | Out of Resources, own top-level folder |
| `08 Resources/knowledge/` | `08 Knowledge/` | Move + promote | Out of Resources, own top-level folder |
| `08 Resources/content-drafts/` | `10 Content/` | Move + promote | Out of Resources, own top-level folder |
| `08 Resources/calendar-cache.md` | `.cache/calendar-cache.md` | Move to cache | Not user-facing, hide in .cache |
| `08 Resources/granola-processed.md` | `.cache/granola-processed.md` | Move to cache | Not user-facing, hide in .cache |
| `08 Resources/claude-memory/` | `.cache/claude-memory/` | Move to cache | Not user-facing, hide in .cache |
| `10 Health/` | `09 Health/` | Rename folder | Renumber after People moved up |
| _(new)_ | `11 Archive/` | Create | PARA Archive folder |
| _(new)_ | `.cache/` | Create | Cache directory for non-user-facing files |

### Unchanged Paths

These paths stay the same (numbers happen to match the new convention):

| Path | Reason |
|------|--------|
| `01 Backlog.md` | Already at correct number |
| `03 Daily/` | Already at correct number |
| `04 Weekly/` | Already at correct number |
| `06 Jira/` | Already at correct number |
| `config/` | Unnumbered, stays in place |
| `11 Nutrition/shopping-lists/` | Stays at current location |

### Post-Migration Cleanup

After migration completes:
1. Remove empty `08 Resources/` folder (all contents moved out)
2. Verify all wikilinks still resolve (filenames unchanged)
3. Update `config.yaml` structure section to match new paths
4. Git commit the migration with message: `feat(vault): migrate to PARA-inspired structure`

---

*This document is the canonical reference for vault structure. All skills and plans in Phase 01-gtd-core should reference this file for paths, schemas, and conventions.*
