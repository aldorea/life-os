# Phase 1: GTD Core - Research

**Researched:** 2026-03-29
**Domain:** Claude Code CLI skills, Obsidian vault manipulation, GTD methodology
**Confidence:** HIGH

## Summary

Phase 1 is entirely about Claude Code CLI skills (SKILL.md files) that read and write an Obsidian vault. There are no npm packages, no TypeScript compilation, no web framework code in this phase. The "technology" is: markdown skill definitions that Claude interprets at runtime, YAML configuration, and filesystem operations against the vault.

The existing codebase already has 13 skills following a mature pattern: YAML frontmatter (name, description), Step 0 config loading, vault path resolution, graceful degradation, present-then-confirm for mutations. Phase 1 upgrades existing skills (capture, process-inbox, today, week, status) and creates new ones (morning orchestrator, next-actions, projects, someday, end-of-day, quarterly reflection, migrate-vault, goal management).

**Primary recommendation:** Follow the existing skill pattern exactly. Every new skill starts with Step 0 config loading, uses config-driven paths, supports graceful degradation, and requires user confirmation before mutations. The vault restructuring and migration skill should run first as a prerequisite for all other work.

<user_constraints>

## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** Restructure vault NOW in Phase 1 -- define target folder structure and frontmatter schemas upfront, migrate existing content.
- **D-02:** Keep numbered PARA-ish folder convention (01, 02, 03...) with PARA-inspired categories. Rename/reorganize within that convention.
- **D-03:** Strict frontmatter schemas per note type (task, person, meeting, goal, knowledge). Skills validate on read/write. Required fields enforced.
- **D-04:** Auto-migrate existing vault content with git backup. Git commit current state first, then auto-migrate: move files to new folders, add frontmatter to existing notes, update wikilinks. One-time migration skill.
- **D-05:** Morning ritual (RITUAL-02) is an orchestrator that chains existing skills: sync-calendar -> sync-jira -> sync-slack -> sync-granola -> process-inbox -> today. One command to start the day.
- **D-06:** Weekly review (RITUAL-03) is a guided interactive facilitator -- walks through GTD review steps: retrospective -> inbox zero check -> backlog health -> stalled projects -> someday/maybe review -> next week planning.
- **D-07:** Missing GTD views (next actions by context, projects list, someday/maybe) are separate dedicated skills: /next-actions, /projects, /someday.
- **D-08:** Existing skills upgraded alongside vault migration to use new paths, new frontmatter schemas, and new conventions.
- **D-09:** Goals stored in single goals.yaml file with structured fields (name, metric, target, weight, deadline, status, dimension, horizon, history[]).
- **D-10:** Goal dimensions organized as flat list with tags -- each goal has a 'dimension' field (professional/personal/health) and a 'horizon' field (quarterly/annual).
- **D-11:** Goal progress history tracked as history[] array in goals.yaml -- each entry is {date, value, note}.
- **D-12:** Quarterly reflection (GOAL-03) is an interactive CLI workflow -- guided reflection: review each goal's progress, start/stop/continue analysis, set next quarter goals.
- **D-13:** Phase 1 capture is CLI only (/dump and /capture). Obsidian QuickAdd and Apple Shortcuts deferred.
- **D-14:** Git auto-commit before vault mutations. Every skill that writes to the vault does a git commit of affected files first. Rollback = git revert.
- **D-15:** AI inbox processing follows suggest & confirm pattern. AI suggests, user confirms before changes.
- **D-16:** GTD contexts use hashtag tags: #home, #office, #calls, #computer. Added to config.tags.contexts.

### Claude's Discretion
- Implementation details of the migration skill (file traversal, wikilink updating strategy)
- Exact folder numbering and naming for new/renamed folders
- Frontmatter field names and types per note type (as long as they're strict and documented)
- Internal architecture of how skills chain in the morning orchestrator

### Deferred Ideas (OUT OF SCOPE)
- Obsidian QuickAdd plugin integration for capture -- future phase
- Apple Shortcuts for capture from anywhere on macOS/iOS -- future phase
- Apple Reminders capture path -- Phase 2 (SYNC-02)
- Web dashboard for goal visualization -- v2 (DASH-01)

</user_constraints>

<phase_requirements>

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| VAULT-01 | Consistent folder structure (PARA-inspired) with documented conventions | D-01, D-02: Vault restructuring with numbered PARA convention. Architecture Patterns section defines target structure. |
| VAULT-02 | Standardized frontmatter schemas per note type | D-03: Strict schemas documented in Frontmatter Schemas section below. |
| VAULT-03 | Tagging system defined and documented | D-16: GTD contexts (#home, #office, #calls, #computer) plus existing tags from config. Tagging Convention section below. |
| VAULT-04 | Existing content migrated without data loss | D-04: Auto-migrate skill with git backup. Migration Pattern section below. |
| GTD-01 | Quick-capture to inbox in under 5 seconds | D-13: Existing /capture + /dump skills upgraded. Skill already exists, needs path/schema updates. |
| GTD-02 | Process inbox with AI-powered clarification | D-15: Existing /process-inbox skill upgraded with suggest & confirm. Already works, needs schema updates. |
| GTD-03 | View next actions filtered by GTD context | D-07, D-16: New /next-actions skill filtering by #home, #office, #calls, #computer. |
| GTD-04 | Manage projects list with next actions | D-07: New /projects skill. Reads vault projects folder, checks each has a next action. |
| GTD-05 | Maintain Someday/Maybe list | D-07: New /someday skill. Separate backlog section or dedicated file. |
| GTD-06 | Detect stalled projects | New logic in /projects skill: no next action OR no progress in X days (configurable). |
| RITUAL-01 | Generate daily note with agenda, focus, tasks, goal alignment | D-08: Existing /today skill upgraded with goal alignment from goals.yaml. |
| RITUAL-02 | Morning GTD ritual (sync, clean, process) | D-05: New /morning orchestrator chaining sync-calendar -> sync-jira -> sync-slack -> sync-granola -> process-inbox -> today. |
| RITUAL-03 | Weekly review (retrospective, inbox zero, backlog health, plan) | D-06: Existing /week skill expanded into interactive facilitator with GTD review steps. |
| RITUAL-04 | End-of-day close ritual | New /close skill: reflection prompts, energy check, move completed tasks, log summary. |
| GOAL-01 | Define goals with metrics, weights, deadlines, status | D-09, D-10: goals.yaml schema with structured fields. New /goal skill for CRUD. |
| GOAL-02 | View goal progress across all dimensions | Existing /status skill already reads goals. Upgrade to show dimensions and progress bars. |
| GOAL-03 | Quarterly reflection workflow | D-12: New /quarterly skill -- interactive guided reflection. |
| GOAL-04 | Daily note shows focus aligned with current goals | D-08: /today skill already suggests focus from goals. Ensure it reads goals.yaml properly. |
| GOAL-05 | Track goal progress over time (history) | D-11: history[] array in goals.yaml. Weekly review auto-appends snapshot. |

</phase_requirements>

## Architecture Patterns

### Existing Skill Pattern (MUST follow)

Every skill in this project follows this exact pattern:

```markdown
---
name: skill-name
description: Use when [trigger conditions]. Use when user says "[trigger phrases]".
---

# skill-name

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

[What the skill does in 1-2 sentences]

## Process

### 1. [First step]
[Instructions Claude follows at runtime]

### 2. [Second step]
...
```

Key conventions:
- **Config-driven paths:** All vault paths resolved through `config.yaml` `structure` section. Zero hardcoded paths.
- **Graceful degradation:** Every data source has a fallback if missing (show message, skip section, continue).
- **Present-then-confirm:** All mutating skills show a plan, ask confirmation, then apply. Never auto-mutate.
- **Wikilinks everywhere:** People as `[[Name]]`, projects as `[[ProjectName]]`, Jira as `[[TICKET-123]]`.
- **Spanish content:** All vault content and user-facing text in Spanish.
- **`${CLAUDE_PLUGIN_ROOT}`:** This variable resolves to the project root at runtime. All config loading uses it.

### Target Vault Structure (VAULT-01)

Restructured PARA-ish numbered folders. New/renamed folders marked with `*`:

```
vault/
  00 Inbox.md                    # Renamed from "02 Inbox.md"
  01 Backlog.md                  # Renamed from "01 Backlog.md"
  02 Projects/                   # Renamed from "05 Projects" -- PARA: Projects
  03 Daily/                      # Renamed from "03 Daily"
  04 Weekly/                     # Renamed from "04 Weekly"
  05 People/                     # Renamed from "09 People" -- PARA: Areas (relationships)
  06 Jira/                       # Renamed from "06 Jira"
  07 Meetings/                   # Renamed from "08 Resources/meetings"
  08 Knowledge/                  # Renamed from "08 Resources/knowledge"
  09 Health/                     # Renamed from "10 Health"
  10 Content/                    # Renamed from "08 Resources/content-drafts"
  11 Archive/                    # New -- PARA: Archive
  config/
    goals.yaml                   # Goal definitions (D-09)
    constraints.yaml             # Work hours, hard stops
    connectors.yaml              # Sync connector configs
    voice.md                     # Writing voice profile
  .cache/
    calendar-cache.md            # From "08 Resources/calendar-cache.md"
    granola-processed.md         # From "08 Resources/granola-processed.md"
```

**Rationale:** Lower numbers = higher frequency access. Inbox and Backlog at top (00, 01). Daily use folders (Projects, Daily, Weekly) in 02-04. Reference folders (People, Jira, Meetings) in 05-07. Knowledge and growth in 08-10. Archive last. Cache files hidden in `.cache/` since they are not user-facing.

**Note:** Exact numbering and naming is Claude's discretion per CONTEXT.md. The above is a recommendation. The planner should define the final structure.

### Frontmatter Schemas (VAULT-02)

Strict schemas per note type. Required fields marked with `*`.

**Task (in Backlog.md -- inline, not files):**
Tasks live as checkbox lines in Backlog.md, not as individual files. Tags are inline: `- [ ] Task description #work #next #2026-W14`

**Daily Note:**
```yaml
---
tags: daily          # *
date: 2026-03-29     # * ISO format
week: 2026-W13       # * ISO week
day: Saturday        # * English day name
---
```

**Weekly Note:**
```yaml
---
tags: weekly         # *
week: 2026-W13       # * ISO week format
date_start: 2026-03-24  # * Monday
date_end: 2026-03-30    # * Sunday
---
```

**Person:**
```yaml
---
tags: person         # *
name: Full Name      # *
company: Company     # optional
role: Role           # optional
last_interaction: 2026-03-29  # * updated by sync skills
created: 2026-03-29  # *
---
```

**Meeting:**
```yaml
---
tags: meeting        # *
date: 2026-03-29     # *
attendees:           # * list of names
  - Name One
  - Name Two
source: granola|manual  # *
project: ProjectName    # optional
---
```

**Goal (in goals.yaml -- NOT markdown):**
```yaml
goals:
  - id: goal-unique-id       # *
    name: "Goal name"        # *
    dimension: professional   # * professional|personal|health
    horizon: quarterly        # * quarterly|annual
    metric: "Metric name"    # *
    target: 100              # * numeric target
    current: 45              # * current value
    unit: "%"                # optional
    weight: 0.3              # * 0.0-1.0, all weights sum to 1.0
    deadline: 2026-06-30     # *
    status: in_progress      # * not_started|in_progress|completed|abandoned
    history:                 # progress snapshots
      - date: 2026-03-22
        value: 40
        note: "Completed module 3"
```

**Knowledge Note:**
```yaml
---
tags: knowledge      # *
topic: Topic Name    # *
last_updated: 2026-03-29  # *
entries: 5           # * count
maturity: growing    # * seed|growing|ready
---
```

### Tagging Convention (VAULT-03)

Updated `config.tags` section for GTD contexts:

```yaml
tags:
  contexts: ["#home", "#office", "#calls", "#computer"]   # GTD contexts (D-16)
  projects: ["#miportal", "#sherpa", "#previene", "#orbitant", "#lifeos", "#salud"]
  priority: ["#urgente", "#importante"]
  actionability: ["#next", "#esperando", "#someday"]       # Added #someday
  status: ["#active", "#stalled", "#completed", "#archived"]  # Project status tags
```

### Git Safety Pattern (D-14)

Every mutating skill includes this step before writing:

```
### Pre-mutation: Git safety snapshot
1. Run `git add [files that will be modified]`
2. Run `git commit -m "backup: before [skill-name] mutation"`
3. If commit fails (nothing to commit), continue anyway
4. Proceed with mutations
5. After mutations complete, run `git add [modified files]` + `git commit -m "skill([skill-name]): [description of changes]"`
```

This enables rollback via `git revert` for any skill operation.

### Orchestrator Pattern (D-05 - Morning Ritual)

The /morning skill chains other skills sequentially with error isolation:

```markdown
## Process

### 1. Chain skills with graceful degradation

Execute each skill in order. If any fails, log the error and continue to the next:

| Order | Skill | On failure |
|-------|-------|------------|
| 1 | sync-calendar | Skip -- daily note will lack calendar data |
| 2 | sync-jira | Skip -- Jira tasks won't be fresh (Phase 2 concern, skip in Phase 1) |
| 3 | sync-slack | Skip -- no Slack updates (Phase 2 concern, skip in Phase 1) |
| 4 | sync-granola | Skip -- no meeting note updates |
| 5 | process-inbox | Skip -- inbox stays dirty, user can run manually |
| 6 | today | Must succeed -- this is the core output |

Note: sync-jira and sync-slack are Phase 2 skills. In Phase 1 the orchestrator
calls them if they exist, skips gracefully if they don't.
```

### Interactive Facilitator Pattern (D-06 - Weekly Review, D-12 - Quarterly)

For guided workflows that walk through multiple steps interactively:

```markdown
## Process

### Step flow
Present each step one at a time. Wait for user input/confirmation before proceeding.

| Step | Action | User interaction |
|------|--------|-----------------|
| 1 | Show retrospective data | "How did the week go? Anything to add?" |
| 2 | Show inbox status | "Inbox has X items. Process now? (y/n)" |
| 3 | Show backlog health | "X tasks dragged >2 weeks. Archive or reschedule?" |
| 4 | Show stalled projects | "These projects have no next action. Add one?" |
| 5 | Show someday/maybe | "Review these -- promote any to active?" |
| 6 | Plan next week | "Here are suggested priorities. Adjust?" |

Each step reads fresh data, presents it, and waits for user direction.
```

### New Skills Summary

| Skill | Type | Requirement | Base |
|-------|------|-------------|------|
| /morning | Orchestrator | RITUAL-02 | New -- chains existing skills |
| /close | Ritual | RITUAL-04 | New -- end-of-day reflection |
| /next-actions | View | GTD-03 | New -- reads Backlog, filters by context |
| /projects | View | GTD-04, GTD-06 | New -- reads Projects folder, detects stalled |
| /someday | View | GTD-05 | New -- reads "Algun dia" backlog section |
| /goal | CRUD | GOAL-01, GOAL-05 | New -- manages goals.yaml |
| /quarterly | Facilitator | GOAL-03 | New -- interactive quarterly reflection |
| /migrate-vault | One-time | VAULT-04 | New -- disposable migration skill |
| /weekly-review | Facilitator | RITUAL-03 | Upgrade of /week into interactive facilitator |
| /dump | Capture | GTD-01 | May exist or be alias for /capture |

### Upgraded Skills

| Skill | Changes needed |
|-------|---------------|
| /capture | Update paths to new vault structure, validate frontmatter |
| /process-inbox | Update paths, add GTD context tag suggestions (#home, #office, etc.), add #someday routing |
| /today | Update paths, ensure goals.yaml reading works with new schema, add goal alignment display |
| /week | Update paths, split into /week (generator) and /weekly-review (facilitator) |
| /status | Update paths, add goal dimensions display, add progress history visualization |
| /content | Update paths to new Knowledge/Content folders |
| /sync-calendar | Update cache path to .cache/ |
| /sync-granola | Update processed path to .cache/ |
| /sync-slack | Update if needed |
| /shop | Update paths if menus folder moves |
| /train | Update paths if health folder moves |

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| YAML parsing | Custom YAML parser | Claude's native YAML understanding | Skills are interpreted by Claude at runtime -- Claude reads YAML natively |
| Markdown frontmatter | Regex-based parser | Claude's native markdown understanding | Same -- Claude parses frontmatter natively when reading files |
| Wikilink detection | Custom regex | Pattern matching in skill instructions | Existing skills already describe wikilink patterns; follow the convention |
| Date calculations | Manual date math | Claude's date awareness + ISO formats | Claude handles date math reliably with ISO format strings |
| Fuzzy dedup matching | Levenshtein/edit distance code | Claude's semantic understanding | process-inbox already relies on Claude's AI to detect duplicate intent |
| File globbing | Custom traversal code | Claude's ability to use Bash `ls`, `find` | Skills can instruct Claude to use shell commands for file discovery |

**Key insight:** This project has zero runtime code. Skills are markdown instructions that Claude Code interprets. "Don't hand-roll" means "don't write code that Claude would need to execute" -- instead, write instructions that leverage Claude's native capabilities (reading files, understanding YAML, semantic matching, date awareness).

## Common Pitfalls

### Pitfall 1: Forgetting graceful degradation
**What goes wrong:** New skill assumes a data source exists and fails hard when it doesn't.
**Why it happens:** Developer tests with a complete vault but real vaults have missing pieces.
**How to avoid:** Every data source read must have a "if missing, skip gracefully" instruction. See existing skills' degradation tables.
**Warning signs:** Skill has no "Graceful degradation" section.

### Pitfall 2: Auto-mutating without confirmation
**What goes wrong:** Skill writes files without showing plan first. User loses data or gets unexpected changes.
**Why it happens:** Copying patterns from non-interactive tools.
**How to avoid:** D-15 mandates suggest & confirm. Every mutating skill must present a plan table and ask "Apply these changes?" before writing.
**Warning signs:** Skill has a write step not preceded by a presentation step.

### Pitfall 3: Hardcoded vault paths
**What goes wrong:** Skill breaks when user's vault structure differs from developer's.
**Why it happens:** Using literal paths like "03 Daily" instead of `{config.structure.daily_notes}`.
**How to avoid:** ALL paths must go through `config.yaml` structure section. Migration skill must update config.yaml to match new structure.
**Warning signs:** Any path in a skill that doesn't reference `config.structure.*`.

### Pitfall 4: Breaking wikilinks during migration
**What goes wrong:** Moving files breaks `[[links]]` throughout the vault because Obsidian wikilinks use filenames.
**Why it happens:** Obsidian wikilinks resolve by filename, not path. Moving a file to a new folder does NOT break wikilinks. But RENAMING a file does.
**How to avoid:** Migration should MOVE files (preserving names) not RENAME them. If renaming is needed, search all .md files for `[[old-name]]` and update to `[[new-name]]`.
**Warning signs:** Migration renames files without a wikilink update step.

### Pitfall 5: config.yaml drift from config.example.yaml
**What goes wrong:** New folders/tags added to config.example.yaml but user's config.yaml not updated. Skills break on missing config keys.
**Why it happens:** config.yaml is gitignored; config.example.yaml is in git.
**How to avoid:** Migration skill must update both config.example.yaml (in git) AND the user's config.yaml. New skills should document what config keys they need.
**Warning signs:** Skill references config keys not present in config.example.yaml.

### Pitfall 6: Spanish vs English inconsistency
**What goes wrong:** Some content generated in English when all vault content is in Spanish.
**Why it happens:** Claude defaults to English; skill instructions don't specify language.
**How to avoid:** Every content-generating skill must explicitly state "Generate all content in Spanish" in its instructions.
**Warning signs:** Mixed language in generated notes.

### Pitfall 7: Overwriting user content in daily/weekly notes
**What goes wrong:** Regenerating a daily note overwrites the user's manual entries (Log, Cierre sections).
**Why it happens:** Skill regenerates entire note instead of updating only auto-generated sections.
**How to avoid:** Existing /today skill already handles this -- "If exists: Only update Foco and Agenda sections. Do NOT overwrite Log, Cierre, or any user-written content." Apply same pattern to all updatable notes.
**Warning signs:** Skill creates notes but has no "if exists, update only X sections" logic.

### Pitfall 8: goals.yaml weight validation
**What goes wrong:** Goal weights don't sum to 1.0, making progress calculations meaningless.
**Why it happens:** User adds/removes goals without rebalancing weights.
**How to avoid:** /goal skill should warn when weights don't sum to 1.0 after any CRUD operation. Weekly/quarterly reviews should flag imbalanced weights.
**Warning signs:** No validation step after goal mutations.

## Migration Pattern (VAULT-04)

The one-time `/migrate-vault` skill must:

1. **Git snapshot:** Commit all current vault state with message "backup: pre-migration snapshot"
2. **Read current config.yaml** to find existing paths
3. **Define target structure** (new folder names and paths)
4. **Present migration plan** showing: folders to create, folders to rename/move, files to move, config.yaml changes
5. **Wait for user confirmation**
6. **Execute migration:**
   - Create new folders
   - Move files from old paths to new paths (preserving filenames for wikilink safety)
   - Add missing frontmatter to existing notes that lack it
   - Update config.yaml structure section with new paths
   - Update config.example.yaml to match
7. **Verify:** Read config, check all paths resolve, report any orphaned files
8. **Git commit:** "feat(vault): migrate to PARA-inspired structure"

**Critical:** Obsidian resolves wikilinks by filename, not path. Moving `09 People/John.md` to `05 People/John.md` does NOT break `[[John]]` links. This makes folder reorganization safe.

## Code Examples

### Skill boilerplate (for all new skills)

```markdown
---
name: skill-name
description: Use when [conditions]. Use when user says "[trigger phrases in English and Spanish]".
---

# skill-name

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

[1-2 sentence description]

## Process

### 1. [Read data]
[What to read, with graceful degradation table]

### 2. [Process/Analyze]
[What to do with the data]

### 3. [Present to user]
[Show plan/results, ask for confirmation if mutating]

### 4. [Apply changes] (mutating skills only)
[Only after confirmation, with git safety]

## Graceful Degradation

| Missing data | Behavior |
|-------------|----------|
| [source] | [fallback behavior] |

## Important Rules

- [Constraints specific to this skill]
- Spanish language for all content
```

### goals.yaml example

```yaml
# Goal tracking configuration
# Each goal has measurable metrics, weights, and progress history
# Weights within the same horizon should sum to 1.0

goals:
  - id: revenue-q2
    name: "Aumentar revenue de consultoría un 20%"
    dimension: professional
    horizon: quarterly
    metric: "Revenue mensual"
    target: 12000
    current: 9500
    unit: "EUR"
    weight: 0.35
    deadline: 2026-06-30
    status: in_progress
    history:
      - date: 2026-01-15
        value: 8000
        note: "Baseline Q1"
      - date: 2026-03-22
        value: 9500
        note: "Nuevo cliente cerrado"

  - id: lifeos-v1
    name: "Lanzar Life OS v1"
    dimension: professional
    horizon: quarterly
    metric: "Phases completados"
    target: 3
    current: 0
    unit: "phases"
    weight: 0.25
    deadline: 2026-06-30
    status: in_progress
    history: []

  - id: gym-consistency
    name: "Ir al gym 3x/semana consistentemente"
    dimension: health
    horizon: quarterly
    metric: "Semanas con 3+ sesiones"
    target: 12
    current: 8
    unit: "semanas"
    weight: 0.20
    deadline: 2026-06-30
    status: in_progress
    history:
      - date: 2026-03-22
        value: 8
        note: "Semana 12 completada"
```

### Next actions filter pattern (/next-actions)

```markdown
### 2. Filter tasks

Read `VAULT/{config.structure.backlog}`.
Parse all lines matching `- [ ] ` (unchecked tasks).

If user provided a context filter (e.g., `/next-actions #home`):
- Show only tasks containing that context tag
- Also show tasks with #next tag that match the context

If no filter:
- Group tasks by context tag
- Show count per context: "#home (3), #office (5), #calls (1), #computer (8)"
- Show tasks tagged #next first (regardless of context)

### 3. Present

Show filtered tasks in a clean list:

## Next Actions [#context]

- [ ] Task one #work #next
- [ ] Task two #personal #home
...

Total: X tasks
```

### Stalled project detection pattern (/projects, GTD-06)

```markdown
### 2. Detect stalled projects

For each project in `VAULT/{config.structure.projects}/`:
1. Read frontmatter -- check `status` field
2. If status is `active`:
   a. Search Backlog for tasks tagged with this project's tag
   b. Check if ANY task has #next tag -- if none, project is stalled (no next action)
   c. Check frontmatter `last_updated` -- if > {config.projects.stalled_days} days ago, project is stalled (no progress)
3. Flag stalled projects with reason

### 3. Present

## Projects

| Project | Status | Next Action | Last Updated | Alert |
|---------|--------|-------------|-------------|-------|
| [[Project A]] | active | "Do X" | 3 days ago | |
| [[Project B]] | active | NONE | 15 days ago | STALLED -- no next action |
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Skills use hardcoded paths | Config-driven paths via config.yaml | Already in place | All new skills MUST follow |
| No frontmatter validation | Strict schemas per note type | Phase 1 (D-03) | Skills validate on read/write |
| No GTD contexts | #home, #office, #calls, #computer | Phase 1 (D-16) | Replaces generic #work/#personal for actionability |
| Weekly note = generator only | Weekly review = interactive facilitator | Phase 1 (D-06) | /week generates, /weekly-review facilitates |
| No git safety | Auto-commit before mutations | Phase 1 (D-14) | Every mutating skill does git commit first |

## Open Questions

1. **Backlog structure: flat file vs. multiple files**
   - What we know: Currently a single `Backlog.md` file with sections. GTD-05 needs Someday/Maybe.
   - What's unclear: Should Someday/Maybe be a section in Backlog.md or a separate file? Projects list -- section in Backlog or read from Projects folder?
   - Recommendation: Keep Backlog.md for active tasks. Add "Algun dia / Someday" as a section in Backlog.md (follows existing pattern). Projects folder for project notes; /projects skill reads from folder.

2. **Morning orchestrator: how to chain skills**
   - What we know: D-05 says chain sync-calendar -> ... -> today. But Claude Code skills are invoked one at a time by the user.
   - What's unclear: Does "chaining" mean the /morning SKILL.md contains instructions to run each skill sequentially? Or does it mean the skill instructs Claude to perform the same steps inline?
   - Recommendation: The /morning skill should contain inline instructions that replicate what each sub-skill does, rather than trying to "call" other skills. This avoids tool-calling complexity. Each step references the sub-skill by name for clarity but executes the logic directly.

3. **Dump vs. Capture distinction**
   - What we know: D-13 mentions both /dump and /capture. Current repo has /capture (for knowledge). GTD-01 is about quick inbox capture.
   - What's unclear: Are these the same skill? Different skills?
   - Recommendation: /dump = ultra-fast inbox capture (append raw text to Inbox.md, no classification). /capture = knowledge capture (existing skill, classifies into knowledge notes). Different use cases, different skills.

4. **End-of-day ritual (RITUAL-04) scope**
   - What we know: Requirements say "reflection, energy check, move completed tasks."
   - What's unclear: "Move completed tasks" to where? Archive? Or just mark done in daily note?
   - Recommendation: /close updates the daily note's "Cierre del dia" section with reflection and energy, marks tasks as done in Backlog if user confirms, and logs summary. No file moves needed -- Obsidian Tasks plugin tracks done status.

5. **config.yaml changes needed**
   - What we know: New structure paths, new tags, new goal-related config, stalled project threshold.
   - Recommendation: Add these new config keys: `structure.someday` (or keep in backlog), `structure.archive`, `structure.cache`, `projects.stalled_days` (default 14), update `tags.contexts` to GTD contexts, update `tags.actionability` to include `#someday`.

## Project Constraints (from CLAUDE.md)

- **Package manager:** Detect from lockfile. No lockfile exists yet -- not relevant for Phase 1 (no npm packages).
- **Language:** Match conversation language (Spanish or English). Vault content in Spanish.
- **Conventional Commits:** `<type>(<scope>): <description>` -- applies to git safety commits too.
- **Feature branches:** `feat/<name>`, `fix/<name>`, etc. Never commit directly to `main`.
- **Serena MCP:** Use for source code editing. Skills are markdown files, so standard Read/Write tools are appropriate (non-code files).
- **GSD Workflow:** All edits through GSD workflow commands.
- **DRY:** Don't repeat logic across skills. If multiple skills need the same data reading pattern, document it once and reference it.
- **Security (OWASP):** Not directly relevant for local vault manipulation, but config.yaml is gitignored (contains personal paths). Maintain this pattern.

## Sources

### Primary (HIGH confidence)
- Existing skill files in `/Users/alfonso/Documents/life-os/skills/` -- 13 skills examined for patterns
- `/Users/alfonso/Documents/life-os/config.example.yaml` -- current configuration schema
- `/Users/alfonso/Documents/life-os/.planning/phases/01-gtd-core/01-CONTEXT.md` -- 16 locked decisions

### Secondary (MEDIUM confidence)
- Obsidian wikilink behavior (filename-based resolution, not path-based) -- verified from training data, consistent with Obsidian documentation
- GTD methodology (David Allen's contexts, weekly review steps) -- well-established methodology

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- this phase has no external dependencies; it's all SKILL.md files + config.yaml + vault files
- Architecture: HIGH -- existing 13 skills establish clear patterns; new skills follow same patterns
- Pitfalls: HIGH -- derived from analyzing existing skill patterns and common vault manipulation issues

**Research date:** 2026-03-29
**Valid until:** 2026-06-29 (90 days -- stable domain, no external dependencies that change rapidly)
