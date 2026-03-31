# Phase 3: Unified Sync - Context

**Gathered:** 2026-03-31
**Status:** Ready for planning

<domain>
## Phase Boundary

Single `/sync` command that runs all configured connectors (calendar, jira, slack, reminders, granola, training) sequentially, reports results clearly, and handles failures independently. Replaces the 5+ individual sync skill invocations with one unified entry point. Also simplifies `/morning` to call `/sync` internally instead of inlining all sync logic.

This phase does NOT cover: inbox processing (stays in /morning), daily note generation (stays in /morning), new connectors, or changes to connector logic itself.

</domain>

<decisions>
## Implementation Decisions

### /sync vs /morning Architecture
- **D-01:** `/sync` is a new standalone skill at `skills/sync/SKILL.md`. It is the unified sync engine that runs all configured connectors.
- **D-02:** `/morning` calls `/sync` as its first step. Morning's SKILL.md is simplified to: Step 1 = /sync, Step 2 = process-inbox, Step 3 = generate daily note. All inline sync logic is removed from morning.
- **D-03:** `/morning` shows the /sync report inline as part of its summary — one unified output, not two separate reports.

### Connector Selection
- **D-04:** `/sync` always runs ALL connectors configured in `connectors.yaml`. No selective filtering, no flags. If a connector isn't configured, it's skipped automatically.
- **D-05:** Connectors run sequentially, one at a time, in a fixed order. No parallelism.
- **D-06:** All 6 connectors included: calendar, jira, slack, reminders, granola, training. Training (Heavy CSV import) is treated like any other connector.

### Output & Reporting
- **D-07:** Output is one status line per connector with icon + name + result. Format:
  ```
  Sync completado:

  ✓ Calendar   — 12 eventos sincronizados
  ✓ Jira       — 8 tickets (Afianza: 5, Previene: 3)
  ✓ Slack      — 3 items de 2 canales
  ✗ Reminders  — permiso denegado
  ✓ Granola    — 2 reuniones
  ─ Training   — no configurado

  1 error. Revisa Reminders.
  ```
- **D-08:** Icons: ✓ = success, ✗ = error, ─ = skipped/not configured.
- **D-09:** Error summary line at the end only if there were failures.

### Existing Skills Fate
- **D-10:** Individual sync-* skills (sync-calendar, sync-jira, sync-slack, sync-reminders, sync-granola, sync-training) become internal implementation docs. Remove the `description:` field from their SKILL.md frontmatter so Claude Code doesn't register them as user-invocable skills.
- **D-11:** Keep sync-* files in their current locations (`skills/sync-*/SKILL.md`). /sync reads them by path. No folder restructuring.
- **D-12:** User-facing skills after this phase: `/sync` (sync all) and `/morning` (sync + ritual). Individual sync-* are no longer directly invocable by the user.

### Claude's Discretion
- Connector execution order (as long as it's sequential and deterministic)
- Error message formatting for each connector's specific failure modes
- How /sync internally references sync-* SKILL.md files (direct read vs import pattern)
- Whether to add a `last_sync` timestamp file or rely on individual connector timestamps

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Phase Context (carry-forward)
- `.planning/phases/01-gtd-core/01-CONTEXT.md` — D-05 (morning chains syncs), D-14 (git auto-commit before mutations)
- `.planning/phases/02-external-integrations/02-CONTEXT.md` — D-01 (Jira config in connectors.yaml), D-09 (existing sync skills untouched in Phase 2), D-10 (all connectors write to inbox), D-11 (wikilink dedup)

### Existing Skills to Modify
- `skills/morning/SKILL.md` — Current morning orchestrator with inline sync logic. Must be simplified to call /sync.
- `skills/sync-jira/SKILL.md` — Jira connector implementation (remove description field)
- `skills/sync-calendar/SKILL.md` — Calendar connector implementation (remove description field)
- `skills/sync-slack/SKILL.md` — Slack connector implementation (remove description field)
- `skills/sync-reminders/SKILL.md` — Reminders connector implementation (remove description field)
- `skills/sync-granola/SKILL.md` — Granola connector implementation (remove description field)
- `skills/sync-training/SKILL.md` — Training connector implementation (remove description field)

### Configuration
- `config.example.yaml` — Vault paths, connector config location, daily settings

### Requirements
- `.planning/REQUIREMENTS.md` — USYNC-01 (single /sync command), USYNC-02 (independent failure), USYNC-03 (clear output)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **morning/SKILL.md**: Contains all sync orchestration logic (Steps 2-6). This is the primary source to extract from.
- **sync-*/SKILL.md**: Each has complete connector implementation with graceful degradation. These stay as-is for implementation reference.
- **config.yaml loading pattern**: Every skill starts with Step 0 loading config.yaml. /sync follows same pattern.
- **connectors.yaml**: Already has sections for jira, slack, reminders. Calendar and Granola config may be in config.yaml directly.

### Established Patterns
- **Config-driven**: All vault paths via config.yaml. Connectors via connectors.yaml.
- **Graceful degradation**: Each connector handles missing tools/MCP/data without aborting. /sync inherits this.
- **Sequential execution**: Morning already runs connectors sequentially. /sync follows same pattern.
- **Spanish output**: All user-facing text in Spanish.

### Integration Points
- **skills/sync/SKILL.md**: New file to create — the unified sync engine
- **skills/morning/SKILL.md**: Must be rewritten to call /sync instead of inline sync steps
- **skills/sync-*/SKILL.md**: Remove `description:` from frontmatter to make internal
- **connectors.yaml**: Already the configuration source for all connectors

</code_context>

<specifics>
## Specific Ideas

- The output format with ✓/✗/─ icons was explicitly approved by user
- Morning summary groups output into [Sync] and [Ritual] sections for clarity
- Training (Heavy CSV import) is included despite being a different sync pattern — user wants consistency
- "No noise" per USYNC-03 means: only show the status lines + error summary. No verbose per-ticket logging unless debugging.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 03-unified-sync*
*Context gathered: 2026-03-31*
