# Architecture Research

**Domain:** Personal Life OS / Productivity System (CLI-first, Obsidian-backed, GTD)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     INTERACTION LAYER                                │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────────────┐     │
│  │  Claude Code  │  │  Obsidian Vault   │  │   Web Dashboard    │     │
│  │  (CLI Skills) │  │  (View/Navigate)  │  │   (Read-only)      │     │
│  └──────┬───────┘  └────────┬─────────┘  └─────────┬──────────┘     │
│         │                   │                      │                │
├─────────┴───────────────────┴──────────────────────┴────────────────┤
│                     ORCHESTRATION LAYER                              │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────────────┐     │
│  │   Skills      │  │  Config Engine    │  │  Sync Scheduler    │     │
│  │  (13 skills)  │  │  (config.yaml)   │  │  (on-demand)       │     │
│  └──────┬───────┘  └────────┬─────────┘  └─────────┬──────────┘     │
│         │                   │                      │                │
├─────────┴───────────────────┴──────────────────────┴────────────────┤
│                     DOMAIN LOGIC LAYER                               │
│  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌────────┐  │
│  │  GTD     │  │  Goals  │  │  CRM     │  │ Content │  │ Health │  │
│  │  Engine  │  │ Tracker │  │  (People)│  │ Pipeline│  │Tracker │  │
│  └────┬────┘  └────┬────┘  └────┬─────┘  └────┬────┘  └───┬────┘  │
│       │            │            │             │            │        │
├───────┴────────────┴────────────┴─────────────┴────────────┴────────┤
│                     CONNECTOR LAYER                                  │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐       │
│  │  Jira  │  │Calendar│  │Granola │  │ Slack  │  │ Apple  │       │
│  │  MCP   │  │iBuddy  │  │  MCP   │  │  MCP   │  │Remind. │       │
│  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘       │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                     PERSISTENCE LAYER                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Obsidian Vault (Markdown Files)                 │    │
│  │  Daily/ Weekly/ Backlog Inbox Goals People Meetings Knowledge│    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| **Claude Code Skills** | Primary action interface — user runs skills to trigger all workflows | Markdown SKILL.md files dispatched by Claude's LLM reasoning |
| **Obsidian Vault** | Persistent storage, visualization, navigation, Dataview/Tasks queries | Folder of markdown files with frontmatter, community plugins |
| **Web Dashboard** | Read-only status view (goals, tasks, calendar, contacts) | Lightweight web app reading vault files or an API layer |
| **Config Engine** | Centralized configuration for all skills (paths, tags, sections) | Single `config.yaml` read by every skill at Step 0 |
| **GTD Engine** | Inbox capture, clarification, backlog management, weekly review | `capture`, `process-inbox`, `today`, `week` skills |
| **Goals Tracker** | Quarterly/annual goal setting, progress tracking, alignment | `goals.yaml` + `status` skill + weekly/daily alignment logic |
| **CRM (People)** | Contact management, meeting history, staleness tracking | People notes with frontmatter, enriched by `sync-granola` |
| **Content Pipeline** | Knowledge maturity tracking, draft generation, publishing workflow | `capture` + `content` skills, knowledge/ and content-drafts/ folders |
| **Connectors** | Pull external data into vault-native markdown format | `sync-*` skills wrapping MCP servers or CLI tools |

## Recommended Project Structure

```
life-os/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── skills/                      # All Claude Code skills
│   ├── today/SKILL.md           # Daily note generator
│   ├── week/SKILL.md            # Weekly note + retrospective
│   ├── capture/SKILL.md         # Knowledge capture to vault
│   ├── process-inbox/SKILL.md   # GTD: inbox → backlog
│   ├── status/SKILL.md          # Read-only dashboard
│   ├── prep/SKILL.md            # Meeting preparation briefing
│   ├── content/SKILL.md         # Content generation from knowledge
│   ├── sync-jira/SKILL.md       # Jira → vault connector
│   ├── sync-granola/SKILL.md    # Granola → vault + people enrichment
│   ├── sync-calendar/SKILL.md   # Calendar → vault connector
│   ├── sync-slack/SKILL.md      # Slack → vault connector
│   ├── shop/SKILL.md            # Shopping list generator
│   └── train/SKILL.md           # Training log management
├── config.yaml                  # User configuration (gitignored)
├── config.example.yaml          # Configuration template
├── dashboard/                   # Web dashboard (to be built)
│   ├── src/
│   │   ├── api/                 # Vault reader API
│   │   ├── components/          # UI components
│   │   └── pages/               # Dashboard pages
│   └── package.json
└── docs/                        # Documentation
```

### Vault Structure (in Obsidian)

```
Vault/
├── 01 Backlog.md                # GTD: all actionable tasks
├── 02 Inbox.md                  # GTD: unclarified captures
├── 03 Daily/                    # Daily notes (YYYY-MM-DD.md)
├── 04 Weekly/                   # Weekly notes (YYYY-W##.md)
├── 05 Projects/                 # Project overview notes
├── 06 Jira/                     # Synced Jira tickets as notes
├── 08 Resources/
│   ├── calendar-cache.md        # Synced calendar events
│   ├── meetings/                # Individual meeting notes
│   ├── knowledge/               # Topic-based knowledge notes
│   │   └── content-ideas.md     # Content pipeline ideas
│   ├── content-drafts/          # Generated content drafts
│   ├── granola-processed.md     # Dedup registry for Granola
│   └── claude-memory/           # AI context persistence
├── 09 People/                   # Mini CRM: person notes
├── 10 Health/                   # Training logs, menus
├── 11 Nutrition/                # Shopping lists
└── config/
    ├── goals.yaml               # Goal definitions + weights
    ├── constraints.yaml          # Work hours, hard stops
    ├── connectors.yaml           # Dynamic connector config
    └── voice.md                  # Content voice/style guide
```

### Structure Rationale

- **Numbered folders (01-11):** Forces sort order in Obsidian sidebar, groups by domain. Well-established pattern in Obsidian community.
- **Skills as SKILL.md:** Each skill is a self-contained markdown instruction set. Claude's skill dispatcher reads the frontmatter `name` and `description` to route invocations. No code — pure prompt engineering.
- **Config in vault:** `goals.yaml`, `constraints.yaml`, `connectors.yaml` live inside the vault so the user can edit them with Obsidian. The `config.yaml` at repo root holds system paths and tool settings.
- **Separation of concerns:** Connectors write raw data to vault. Domain logic skills read vault data and produce outputs. No skill directly calls an external API except sync-* skills.

## Architectural Patterns

### Pattern 1: Vault as Database (File-based Persistence)

**What:** All persistent state lives as markdown files with YAML frontmatter. No database. Obsidian plugins (Dataview, Tasks) query these files in real-time.
**When to use:** Always — this is the foundational pattern. Every piece of data must have a canonical markdown representation.
**Trade-offs:**
- (+) No vendor lock-in, human-readable, git-versionable, works offline
- (+) Obsidian ecosystem (Dataview, Tasks, Graph View) for free
- (-) No relational queries, no transactions, no concurrent writes
- (-) Performance degrades at ~10K+ files for some Dataview queries

**Example:**
```yaml
# People note frontmatter acts as a "row" in the CRM "table"
---
name: Pedro López
company: Afianza
role: Tech Lead
context: work
last_interaction: 2026-03-25
---
```

### Pattern 2: Connector-Consumer Separation

**What:** Connector skills (`sync-*`) are the only components that talk to external APIs. They write normalized markdown to the vault. Consumer skills (`today`, `week`, `prep`, `status`) only read vault files — they never call external APIs directly.
**When to use:** Always. This decouples external dependencies from core logic.
**Trade-offs:**
- (+) Consumer skills work even if APIs are down (graceful degradation)
- (+) Easy to swap connectors (e.g., replace Granola with another tool)
- (+) Testable — mock vault files, test consumer skills
- (-) Data freshness depends on sync frequency
- (-) Requires a "sync before read" discipline (skills already do this, e.g., `today` calls `sync-calendar` first)

```
External API → [sync-* skill] → Vault Markdown → [consumer skill] → User Output
```

### Pattern 3: Configuration-Driven Skills

**What:** Every skill reads `config.yaml` at Step 0 and uses configuration values for all paths, tags, sections, and behavior. No hardcoded values in skills.
**When to use:** Always. Enables personalization without code changes.
**Trade-offs:**
- (+) Single source of truth for system behavior
- (+) User can customize without touching skills
- (+) Dynamic Jira projects via `connectors.yaml`
- (-) Config file becomes a single point of failure
- (-) Schema validation is implicit (skills fail at runtime if config is wrong)

### Pattern 4: Skill Composition (Skill-calls-Skill)

**What:** Skills can invoke other skills as sub-steps. For example, `today` calls `sync-calendar` before generating the daily note. `week` also calls `sync-calendar`.
**When to use:** When a skill needs fresh data that another skill produces.
**Trade-offs:**
- (+) DRY — sync logic lives in one place
- (+) Composable workflows
- (-) Implicit dependencies (not declared, just documented in SKILL.md)
- (-) Cascading failures if a sub-skill breaks

### Pattern 5: Frontmatter as Structured Metadata

**What:** Every vault note uses YAML frontmatter for machine-readable metadata (dates, tags, status, counts). The markdown body is human-readable content. Dataview and Tasks plugins query frontmatter.
**When to use:** Any note that needs to be queried, filtered, or aggregated.
**Trade-offs:**
- (+) Queryable by Dataview without parsing markdown body
- (+) Standard Obsidian pattern, well-supported
- (-) Frontmatter must be kept in sync (e.g., `last_interaction` in People notes)
- (-) No schema enforcement — drift is possible

## Data Flow

### Primary Data Flows

```
┌──────────────────────────────────────────────────────────────────┐
│                     INBOUND (External → Vault)                    │
│                                                                   │
│  Jira ──[sync-jira]──→ 06 Jira/*.md                              │
│  Granola ──[sync-granola]──→ meetings/*.md + 09 People/*.md       │
│  Calendar ──[sync-calendar]──→ calendar-cache.md                  │
│  Slack ──[sync-slack]──→ (inbox items or context)                 │
│  User voice ──[capture]──→ 02 Inbox.md or knowledge/*.md          │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                     PROCESSING (Vault → Vault)                    │
│                                                                   │
│  02 Inbox.md ──[process-inbox]──→ 01 Backlog.md                   │
│  goals.yaml + Backlog ──[today]──→ 03 Daily/*.md                  │
│  Daily notes + Backlog ──[week]──→ 04 Weekly/*.md                 │
│  knowledge/*.md ──[content]──→ content-drafts/*.md                │
│  People + meetings ──[prep]──→ (conversational output)            │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                     OUTBOUND (Vault → User)                       │
│                                                                   │
│  Vault ──[status]──→ CLI dashboard output                         │
│  Vault ──[Obsidian]──→ Visual navigation, graph, search           │
│  Vault ──[Web Dashboard]──→ Browser-based status overview         │
│  content-drafts ──[manual]──→ LinkedIn, blog (user publishes)     │
└──────────────────────────────────────────────────────────────────┘
```

### Key Data Flows

1. **GTD Capture → Process → Execute:** User captures thought via `capture` skill → lands in Inbox or knowledge → `process-inbox` clarifies and moves to Backlog with tags → `today`/`week` surfaces tasks for execution. This is the core GTD loop.

2. **Meeting Intelligence Loop:** `sync-granola` pulls meetings → creates meeting notes + enriches People notes + extracts action items to Backlog → `prep` reads People + meeting history to prepare for next meeting. This creates a compounding knowledge effect where each meeting makes the next one more informed.

3. **Knowledge → Content Pipeline:** `capture` accumulates entries in topic notes → maturity grows (seed → growing → ready) → `content` scans ready topics and generates drafts → user reviews and publishes. Knowledge compounds into publishable output.

4. **Goal Alignment Thread:** `goals.yaml` defines priorities with weights → `today` suggests daily focus aligned to goals → `week` suggests weekly priorities from goals → `status` shows goal progress → everything is goal-aware.

### State Management

The system has no runtime state. All state is persisted as files in the vault:

| State | Where Stored | Updated By |
|-------|-------------|------------|
| Tasks (pending/done) | `01 Backlog.md` | `process-inbox`, manual, `sync-granola` |
| Today's plan | `03 Daily/{date}.md` | `today` |
| Week's plan | `04 Weekly/{week}.md` | `week` |
| Calendar events | `calendar-cache.md` | `sync-calendar` |
| Jira tickets | `06 Jira/*.md` | `sync-jira` |
| People context | `09 People/*.md` | `sync-granola`, manual |
| Meeting history | `meetings/*.md` | `sync-granola` |
| Knowledge base | `knowledge/*.md` | `capture` |
| Processed meeting IDs | `granola-processed.md` | `sync-granola` |
| Goals & weights | `config/goals.yaml` | Manual |
| System config | `config.yaml` | Manual |

## Scaling Considerations

This is a single-user personal system. Traditional scaling (users, requests/sec) does not apply. The relevant scaling dimensions are:

| Scale Dimension | Current | At 1 Year | At 3 Years | Mitigation |
|----------------|---------|-----------|------------|------------|
| Vault notes | ~100 | ~1,000 | ~5,000 | Obsidian handles 10K+ notes fine. Dataview queries may slow — use targeted `FROM` clauses |
| Daily notes | ~30 | ~365 | ~1,095 | Archive old years to subfolder. Tasks plugin queries already filter by date |
| People notes | ~20 | ~100 | ~300 | No issue. Frontmatter queries are fast |
| Meeting notes | ~50 | ~500 | ~1,500 | Most impactful at scale. Use date-prefixed filenames for natural archiving |
| Knowledge entries | ~20 | ~200 | ~500 | Topic-level notes keep individual files manageable |
| Jira tickets | ~30 | ~200 | ~500 | Archive done tickets periodically. Only sync active ones |
| Config.yaml complexity | Simple | Moderate | Complex | Consider splitting into multiple config files if > 200 lines |

### Scaling Priorities

1. **First bottleneck: Dataview query speed.** At ~2K+ notes, broad `FROM ""` queries slow down. Fix: always scope queries with specific folder paths (already done in current skills).
2. **Second bottleneck: Sync-granola backlog.** If Granola accumulates many unprocessed meetings, a single sync becomes slow. Fix: the processed-registry pattern already handles incremental sync.
3. **Third bottleneck: Backlog.md as a single file.** At 200+ tasks, a single backlog file becomes unwieldy. Fix: split into `Backlog-work.md` and `Backlog-personal.md`, or one file per backlog section.

## Anti-Patterns

### Anti-Pattern 1: Vault as API Cache

**What people do:** Store raw API responses (full Jira JSON, calendar ICS) in the vault.
**Why it's wrong:** Bloats vault, breaks Obsidian readability, makes Dataview queries useless (can't query raw JSON frontmatter).
**Do this instead:** Connectors must transform API data into human-readable markdown with clean frontmatter. The vault is a knowledge base, not a cache.

### Anti-Pattern 2: Skills That Write AND Read External APIs

**What people do:** Build a skill that fetches Jira data AND generates a daily note in one step.
**Why it's wrong:** Couples external dependency to core logic. If Jira is down, daily note generation fails entirely.
**Do this instead:** Maintain connector-consumer separation. `sync-jira` writes to vault. `today` reads from vault. Each works independently.

### Anti-Pattern 3: Hardcoded Paths in Skills

**What people do:** Write `VAULT/03 Daily/` directly in skill logic.
**Why it's wrong:** User can't customize folder structure. Breaking change if vault is reorganized.
**Do this instead:** Always reference `{config.structure.daily_notes}` from config.yaml. Already followed in current skills.

### Anti-Pattern 4: Real-Time Dashboard Polling Vault Files

**What people do:** Web dashboard watches vault files with `fs.watch` and re-renders on every change.
**Why it's wrong:** File watching is unreliable across OSes, causes unnecessary CPU usage, and Obsidian itself watches files causing conflicts.
**Do this instead:** Dashboard reads vault on page load or on explicit refresh. Personal system does not need real-time updates. A 30-second or manual refresh is sufficient.

### Anti-Pattern 5: Mega-Skill (One Skill Does Everything)

**What people do:** Build a single "morning routine" skill that syncs all connectors, processes inbox, generates daily note, and shows status.
**Why it's wrong:** Impossible to debug, can't run partial steps, one failure kills everything.
**Do this instead:** Compose small skills. `today` already calls `sync-calendar` as a sub-step. A "morning" workflow can be a thin orchestrator that calls `sync-granola` → `process-inbox` → `today` → `status` in sequence.

## Integration Points

### External Services

| Service | Integration Pattern | Connector | Notes |
|---------|---------------------|-----------|-------|
| Jira (Afianza + Previene) | MCP server → `sync-jira` skill | `sync-jira` | Dynamic project config via `connectors.yaml`. Multiple instances supported |
| Granola | MCP server → `sync-granola` skill | `sync-granola` | Source-agnostic meeting note format. Granola is replaceable |
| Apple Calendar | `icalBuddy` CLI → `sync-calendar` skill | `sync-calendar` | Local tool, no API key needed |
| Slack | MCP server → `sync-slack` skill | `sync-slack` | Context/mention extraction |
| Apple Reminders | To be implemented | Future connector | Potential via `reminders-cli` or Shortcuts |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Skills ↔ Vault | Direct file read/write | Skills use absolute paths from config. No intermediary |
| Skills ↔ Config | Read `config.yaml` at Step 0 | Every skill starts with config load. Fail-fast if missing |
| Skills ↔ Skills | Skill composition (one skill invokes another) | `today` → `sync-calendar`. Implicit, documented in SKILL.md |
| Skills ↔ MCP | MCP tool calls within skill execution | Claude runtime handles MCP protocol. Skills just reference tool names |
| Web Dashboard ↔ Vault | Read-only file access (API or direct) | Dashboard must NEVER write to vault. Read-only boundary is critical |
| Obsidian ↔ Vault | Native Obsidian file watching | Obsidian owns the UI. Skills own the writes. Potential conflict if both write simultaneously |

### Conflict Prevention: Obsidian vs Skills

Since both Obsidian (manual edits) and Claude Code skills can write to vault files, a simple protocol prevents conflicts:

1. **Skills announce before writing** — always show the user what will change and ask for confirmation (already implemented in `process-inbox`).
2. **Skills preserve user content** — never overwrite sections marked as user-written (Log, Cierre, Reflexion sections in daily/weekly notes).
3. **Append-only connectors** — sync skills add new entries, never modify existing ones (except frontmatter like `last_interaction`).
4. **Dedup registries** — `granola-processed.md` tracks what has been synced to prevent re-processing.

## Build Order (Dependency-Driven)

Based on component dependencies, the recommended build order is:

```
Phase 1: Foundation
  config.yaml schema ← everything depends on this
  Vault structure    ← all skills read/write here
  Core GTD skills    ← capture, process-inbox, today (the daily loop)

Phase 2: Intelligence
  sync-* connectors  ← bring external data in
  week + goals       ← requires daily notes + goals to exist
  prep               ← requires People + meetings from connectors

Phase 3: Compound Value
  CRM enrichment     ← builds on People notes from Phase 2
  Content pipeline   ← builds on knowledge notes from Phase 1 capture
  status dashboard   ← reads everything, build last

Phase 4: Web Dashboard
  Vault reader API   ← needs stable vault structure from Phases 1-3
  Dashboard UI       ← consumes API, read-only view
```

**Rationale:** Each phase produces value independently. Phase 1 is usable alone (GTD works without connectors). Phase 2 adds external data. Phase 3 exploits accumulated data. Phase 4 is a presentation layer that requires everything else to be stable.

## Sources

- [Obsidian + Claude Code PKM starter kit](https://github.com/ballred/obsidian-claude-pkm) — reference architecture for Claude-powered Obsidian systems
- [PKM at Scale: 8,000 Notes](https://www.dsebastien.net/personal-knowledge-management-at-scale-analyzing-8-000-notes-and-64-000-links/) — vault scaling analysis
- [GTD Obsidian Vault Template](https://deepwiki.com/adiehl96/obsidian-vault-template-for-gtd) — GTD-specific vault architecture
- [Obsidian x Claude Code Workflows](https://www.axtonliu.ai/newsletters/ai-2/posts/obsidian-claude-code-workflows) — context layer + skills + automation pattern
- [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/) — meta-tool architecture explanation
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) — official Obsidian agent skills reference
- [AI Operating System for Personal Productivity](https://motyl.dev/news/ai-operating-system-personal-productivity-2026) — AI-first productivity system patterns
- [Super Productivity](https://super-productivity.com/) — reference for Jira/Calendar integration patterns

---
*Architecture research for: Personal Life OS (CLI-first, Obsidian-backed, GTD)*
*Researched: 2026-03-28*
