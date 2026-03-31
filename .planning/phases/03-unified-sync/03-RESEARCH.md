# Phase 3: Unified Sync - Research

**Researched:** 2026-03-31
**Domain:** Claude Code skill orchestration (markdown SKILL.md files), YAML configuration
**Confidence:** HIGH

## Summary

Phase 3 is a pure skill-layer refactor with no new external dependencies. The work consists of: (1) creating a new `skills/sync/SKILL.md` that reads `connectors.yaml` and sequentially invokes each sync-* connector's logic, collecting results and formatting a summary report; (2) simplifying `skills/morning/SKILL.md` to delegate its sync steps to `/sync`; and (3) removing the `description:` frontmatter field from each `sync-*` SKILL.md so they are no longer user-invocable.

All six connectors already exist and work independently with graceful degradation. The core engineering challenge is the orchestration pattern: how `/sync` references each connector's instructions, how it aggregates success/failure/skip results, and how it formats the compact status report per D-07/D-08/D-09.

**Primary recommendation:** Build `/sync` as a self-contained SKILL.md that embeds the orchestration loop directly (read connectors.yaml, iterate connectors in fixed order, execute each connector's logic by reading its SKILL.md, collect results, format report). No TypeScript code, no runtime scripts -- pure skill instructions that Claude Code follows.

<user_constraints>

## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** `/sync` is a new standalone skill at `skills/sync/SKILL.md`. It is the unified sync engine that runs all configured connectors.
- **D-02:** `/morning` calls `/sync` as its first step. Morning's SKILL.md is simplified to: Step 1 = /sync, Step 2 = process-inbox, Step 3 = generate daily note. All inline sync logic is removed from morning.
- **D-03:** `/morning` shows the /sync report inline as part of its summary -- one unified output, not two separate reports.
- **D-04:** `/sync` always runs ALL connectors configured in `connectors.yaml`. No selective filtering, no flags.
- **D-05:** Connectors run sequentially, one at a time, in a fixed order. No parallelism.
- **D-06:** All 6 connectors included: calendar, jira, slack, reminders, granola, training.
- **D-07:** Output is one status line per connector with icon + name + result (see CONTEXT.md for exact format).
- **D-08:** Icons: check = success, X = error, dash = skipped/not configured.
- **D-09:** Error summary line at the end only if there were failures.
- **D-10:** Individual sync-* skills become internal implementation docs. Remove `description:` field from their SKILL.md frontmatter.
- **D-11:** Keep sync-* files in current locations. No folder restructuring.
- **D-12:** User-facing skills after this phase: `/sync` and `/morning`. Individual sync-* are no longer directly invocable.

### Claude's Discretion
- Connector execution order (as long as it's sequential and deterministic)
- Error message formatting for each connector's specific failure modes
- How /sync internally references sync-* SKILL.md files (direct read vs import pattern)
- Whether to add a `last_sync` timestamp file or rely on individual connector timestamps

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope.

</user_constraints>

<phase_requirements>

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| USYNC-01 | Un solo comando `/sync` centraliza Jira (Afianza + Previene), Granola, y calendario en el vault | D-01 defines /sync as standalone skill; D-04/D-05/D-06 define execution model; architecture pattern below shows how to structure the SKILL.md |
| USYNC-02 | Cada conector falla independientemente -- si Jira falla, calendario sigue | Existing graceful degradation pattern in all sync-* skills; /sync wraps each in independent try-continue; result collection pattern below |
| USYNC-03 | Output claro de que se sincronizo y que fallo, sin ruido | D-07/D-08/D-09 define exact output format with icons; report aggregation pattern below |

</phase_requirements>

## Standard Stack

No new libraries or packages are needed. This phase operates entirely within the existing Claude Code skill system.

### Core
| Component | Location | Purpose | Why |
|-----------|----------|---------|-----|
| SKILL.md files | `skills/*/SKILL.md` | Skill definition format | Claude Code reads these as instructions; this is the project's entire runtime |
| config.yaml | `${CLAUDE_PLUGIN_ROOT}/config.yaml` | Vault paths, daily settings | Every skill loads this at Step 0 |
| connectors.yaml | `VAULT/{config.structure.connectors_config}` | Connector-specific config (Jira projects, Slack channels, Reminders lists) | /sync reads this to determine which connectors are configured |
| YAML frontmatter | In each SKILL.md | Skill metadata (name, description) | `description:` field controls whether Claude Code registers the skill as user-invocable |

### No Additional Dependencies
This is a markdown-only phase. No npm packages, no scripts, no build tools. The "code" is SKILL.md instructions that Claude Code interprets at runtime.

## Architecture Patterns

### Project Structure (changes only)
```
skills/
  sync/
    SKILL.md              # NEW -- unified sync orchestrator
  morning/
    SKILL.md              # MODIFIED -- simplified to call /sync
  sync-calendar/
    SKILL.md              # MODIFIED -- remove description: field
  sync-jira/
    SKILL.md              # MODIFIED -- remove description: field
  sync-slack/
    SKILL.md              # MODIFIED -- remove description: field
  sync-reminders/
    SKILL.md              # MODIFIED -- remove description: field
  sync-granola/
    SKILL.md              # MODIFIED -- remove description: field
  sync-training/
    SKILL.md              # MODIFIED -- remove description: field
```

### Pattern 1: Orchestrator Skill with Inline Connector Logic

**What:** `/sync` SKILL.md contains the full orchestration loop: load config, iterate connectors in fixed order, execute each connector's sync logic (referencing the sync-* SKILL.md for implementation details), collect results, format report.

**When to use:** This is the only pattern for this phase.

**Key design decision -- how /sync references sync-* skills:**

The recommended approach is that `/sync` SKILL.md contains a section per connector that says "Execute the sync logic defined in `skills/sync-{name}/SKILL.md`" with a brief summary of what each does. Claude Code reads the referenced SKILL.md when it reaches that step.

This avoids duplicating all connector logic in `/sync` while keeping the orchestrator self-contained. The sync-* SKILL.md files remain the source of truth for each connector's implementation.

**Structure of skills/sync/SKILL.md:**

```markdown
---
name: sync
description: Runs all configured sync connectors and reports results. Use when user says "sync", "sincroniza", "sincronizar todo", "sync all".
---

# sync

## Step 0 -- Load configuration
[Standard config.yaml load pattern]

## Overview
[Brief description of what /sync does]

## Process

### 1. Read connector configuration
Read connectors.yaml. Determine which connectors have configuration present.

### 2. Execute connectors (sequential, fixed order)

For each connector in order: calendar, jira, slack, reminders, granola, training.

#### 2.1 Calendar
- Check: config.calendar.tool != "none"
- Execute: Follow sync logic from skills/sync-calendar/SKILL.md
- Collect result: {status, message}

#### 2.2 Jira
- Check: connectors.yaml has jira.projects with entries
- Execute: Follow sync logic from skills/sync-jira/SKILL.md
- Collect result: {status, message}

[... same for each connector ...]

### 3. Format and display report
[D-07/D-08/D-09 format]

## Graceful Degradation
[Table per connector]
```

### Pattern 2: Simplified Morning Orchestrator

**What:** `/morning` is rewritten to three high-level steps: (1) call /sync, (2) process inbox, (3) generate daily note.

**Key insight:** Morning currently has Steps 2-6 doing individual syncs (calendar, jira, slack, reminders, granola). These 5 steps collapse into one: "Execute the /sync skill and capture its report."

**Structure after simplification:**

```markdown
## Process

### 1. Announce start
Buenos dias! Iniciando rutina matutina...

### 2. Sync all data sources
Execute /sync skill. Capture the full report output.
[sync] section of summary = /sync report verbatim

### 3. Process inbox
[Existing Step 7 logic -- unchanged]

### 4. Generate daily note
[Existing Step 8 logic -- unchanged]

### 5. Morning summary
[Ritual] section: inbox processing + daily note results
[Sync] section: /sync report inline (per D-03)
```

### Pattern 3: Frontmatter Description Removal

**What:** Remove the `description:` field from sync-* SKILL.md frontmatter to make them internal (not user-invocable).

**How Claude Code skill discovery works:** Claude Code scans SKILL.md files and registers those with a `description:` field as invocable commands. Removing `description:` while keeping `name:` means the file still exists as documentation but is not presented to the user as a command.

**Example -- before:**
```yaml
---
name: sync-jira
description: Syncs Jira tickets from configured projects...
---
```

**Example -- after:**
```yaml
---
name: sync-jira
---
```

### Anti-Patterns to Avoid
- **Duplicating connector logic in /sync:** Do not copy-paste each connector's full implementation into /sync. Reference the sync-* SKILL.md files instead. Single source of truth.
- **Adding conditional flags to /sync:** D-04 says no selective filtering. Do not add `--only calendar` or similar flags.
- **Removing sync-* SKILL.md files:** D-11 says keep them in place. They are the implementation reference.
- **Breaking morning's interactive inbox step:** The inbox processing in morning is the only interactive step (user confirms). This must remain intact when morning is simplified.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Connector configuration | Custom config format | Existing connectors.yaml | Already has jira, slack, reminders sections; calendar/granola/training config is in config.yaml |
| Skill invocation | Inter-process calls or scripts | Claude Code reading SKILL.md files directly | This is how the skill system works -- Claude reads instructions, not code |
| Result collection | Shared state file between skills | Inline result tracking within /sync's execution | /sync is one continuous Claude Code execution; results are tracked in working memory |

## Common Pitfalls

### Pitfall 1: Connector Config Scattered Across Two Files
**What goes wrong:** Calendar and training config live in `config.yaml`, while jira, slack, and reminders live in `connectors.yaml`. Granola checks for MCP availability. /sync must check all three sources.
**Why it happens:** Historical -- connectors.yaml was introduced in Phase 2 for new connectors, while calendar and training were configured in config.yaml from Phase 1.
**How to avoid:** /sync Step 1 must explicitly document which config source to check for each connector:
- Calendar: `config.yaml > calendar.tool`
- Jira: `connectors.yaml > jira.projects`
- Slack: `connectors.yaml > slack.channels`
- Reminders: `connectors.yaml > reminders.lists`
- Granola: MCP availability (`query_granola_meetings`)
- Training: `config.yaml > training.import_source`
**Warning signs:** Connector shows as "not configured" when it actually is, because /sync checked the wrong config file.

### Pitfall 2: Morning Summary Duplication
**What goes wrong:** After simplification, morning shows both the /sync report AND its own summary section with sync results, duplicating information.
**Why it happens:** If morning's summary template is not updated to remove the per-connector lines (Step 9 in current morning SKILL.md), both the /sync report and the old summary format appear.
**How to avoid:** Morning's summary must be restructured into two sections: [Sync] (verbatim from /sync) and [Ritual] (inbox + daily note only). Remove all individual connector lines from the morning summary template.
**Warning signs:** User sees sync results twice in morning output.

### Pitfall 3: Granola Has No connectors.yaml Section
**What goes wrong:** /sync tries to check connectors.yaml for granola config but there is no `granola:` section defined. The connector relies on MCP availability, not config.
**Why it happens:** Granola was designed as an MCP-dependent connector with a processed registry file, not a config-driven one.
**How to avoid:** For granola, the "is configured" check is: can Claude access the `query_granola_meetings` MCP tool? Not a YAML config check. Document this exception clearly in /sync.
**Warning signs:** Granola always shows as "not configured" even when the MCP server is available.

### Pitfall 4: Training Connector Has Different Sync Pattern
**What goes wrong:** Training sync requires finding a CSV file (from Downloads or configured path), which is different from all other connectors that pull from APIs/MCP/system tools.
**Why it happens:** Training imports from a file export, not a live service.
**How to avoid:** /sync must handle training's "find CSV" pattern. If no CSV is found, result is "skip" with message "no hay CSV nuevo". Do NOT prompt the user for a file path during /sync -- that would break the non-interactive flow.
**Warning signs:** /sync hangs waiting for user input because training skill asks "where is the CSV?"

### Pitfall 5: Forgetting Git Safety (D-14 from Phase 1)
**What goes wrong:** /sync runs multiple connectors that write to the vault but does not do git commits between mutations.
**Why it happens:** Individual sync-* skills each handle their own git safety. When orchestrated by /sync, it is unclear whether each connector still does its own commits or /sync handles it centrally.
**How to avoid:** Each connector's SKILL.md already includes git safety steps. Since /sync delegates to their logic, the git safety is inherited. Do NOT add a separate git layer in /sync that conflicts. Verify this in testing.
**Warning signs:** Git history shows no commits during sync, or double-commits for the same files.

## Code Examples

### Connector Result Collection Pattern

Within /sync execution, Claude tracks results as a mental model (not a file). After each connector:

```
Connector results (tracked during execution):
- calendar: {status: "success", message: "12 eventos sincronizados"}
- jira: {status: "success", message: "8 tickets (Afianza: 5, Previene: 3)"}
- slack: {status: "success", message: "3 items de 2 canales"}
- reminders: {status: "error", message: "permiso denegado"}
- granola: {status: "success", message: "2 reuniones"}
- training: {status: "skipped", message: "no configurado"}
```

### Report Formatting (D-07 exact format)

```
Sync completado:

[check] Calendar   -- 12 eventos sincronizados
[check] Jira       -- 8 tickets (Afianza: 5, Previene: 3)
[check] Slack      -- 3 items de 2 canales
[X]     Reminders  -- permiso denegado
[check] Granola    -- 2 reuniones
[dash]  Training   -- no configurado

1 error. Revisa Reminders.
```

(Using actual Unicode characters from D-07: checkmark, X, dash)

### SKILL.md Frontmatter Before/After

**sync-jira before:**
```yaml
---
name: sync-jira
description: Syncs Jira tickets from configured projects to the vault...
---
```

**sync-jira after:**
```yaml
---
name: sync-jira
---
```

### Morning SKILL.md Simplified Structure

**Before:** Steps 2-6 are inline sync implementations (calendar, jira, slack, reminders, granola) -- ~200 lines.

**After:** Step 2 is a single delegation: "Execute the /sync skill." -- ~5 lines replacing ~200.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| 5+ individual `/sync-*` commands | Single `/sync` command | Phase 3 (now) | User runs one command instead of five |
| Morning inlines all sync logic | Morning delegates to /sync | Phase 3 (now) | Morning SKILL.md shrinks from ~280 to ~100 lines |
| All sync skills user-invocable | Only /sync and /morning visible | Phase 3 (now) | Cleaner skill list, less cognitive load |

## Open Questions

1. **last_sync timestamp file**
   - What we know: Each connector currently tracks its own timestamps (calendar_cache has last_sync, granola has processed registry, jira overwrites notes). There is no unified "last sync" timestamp.
   - What's unclear: Whether a unified `.cache/last-sync.md` file would be useful for /morning or other skills to check "when was the last full sync?"
   - Recommendation: Claude's discretion per CONTEXT.md. Recommend adding a simple `.cache/last-sync.yaml` with per-connector timestamps. Low effort, useful for debugging and future features. But not blocking.

2. **Training connector interactivity**
   - What we know: sync-training's SKILL.md has a fallback "Ask the user" step if no CSV is found.
   - What's unclear: Should /sync suppress this interactive prompt and just skip?
   - Recommendation: Yes -- /sync must be non-interactive. If training can't find a CSV automatically, it reports "skip" with reason. The user can always run the training sync logic manually if needed.

## Environment Availability

Step 2.6: SKIPPED (no external dependencies identified). This phase modifies only SKILL.md markdown files within the existing project. All external tool dependencies (icalBuddy, osascript, Jira MCP, Slack MCP, Granola MCP) are already managed by the individual sync-* connectors from Phase 2.

## Sources

### Primary (HIGH confidence)
- `skills/morning/SKILL.md` -- current morning orchestrator with inline sync logic (280 lines)
- `skills/sync-*/SKILL.md` -- all six connector implementations, frontmatter structure
- `config.example.yaml` -- vault path structure, calendar/training config locations
- `connectors.example.yaml` -- jira, slack, reminders config structure
- `.planning/phases/03-unified-sync/03-CONTEXT.md` -- all 12 locked decisions
- `.planning/phases/01-gtd-core/01-CONTEXT.md` -- D-05 (morning chains syncs), D-14 (git auto-commit)
- `.planning/phases/02-external-integrations/02-CONTEXT.md` -- D-01 (Jira config), D-09 (existing skills untouched), D-10 (inbox flow), D-11 (wikilink dedup)

### Secondary (MEDIUM confidence)
- Claude Code skill discovery mechanism (description field controls invocability) -- based on observed behavior in this project

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no new libraries, entirely within existing skill system
- Architecture: HIGH -- patterns derived directly from existing morning/sync-* skills and locked decisions
- Pitfalls: HIGH -- identified from reading actual config files and skill implementations

**Research date:** 2026-03-31
**Valid until:** Indefinite (no external version dependencies)
