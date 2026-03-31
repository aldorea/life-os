# Phase 3: Unified Sync - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-31
**Phase:** 03-unified-sync
**Areas discussed:** /sync vs /morning relationship, Connector selection, Output & reporting format, Existing skills fate

---

## /sync vs /morning Relationship

| Option | Description | Selected |
|--------|-------------|----------|
| Extract sync from morning | /sync becomes standalone engine. /morning calls /sync internally, then does inbox + daily note. | ✓ |
| /sync replaces morning's sync section | /sync is new skill, /morning rewritten to call /sync as step 1. | |
| /sync is just morning without ritual | /sync runs what morning does but stops after syncing. | |

**User's choice:** Extract sync from morning
**Notes:** Clean separation: sync = data in, morning = sync + ritual.

### Follow-up: Skill shape

| Option | Description | Selected |
|--------|-------------|----------|
| New skill: skills/sync/SKILL.md | Standalone skill file. User invokes /sync directly. | ✓ |
| Inline in morning, callable standalone | Keep sync logic in morning but add flag for sync-only mode. | |

**User's choice:** New skill: skills/sync/SKILL.md

### Follow-up: Morning edit scope

| Option | Description | Selected |
|--------|-------------|----------|
| Simplify morning | Rewrite morning/SKILL.md to 3 steps: /sync, process-inbox, daily note. | ✓ |
| Minimal change to morning | Replace Steps 2-6 with single /sync invocation, keep rest. | |

**User's choice:** Simplify morning

---

## Connector Selection

| Option | Description | Selected |
|--------|-------------|----------|
| All connectors always | /sync runs every connector in connectors.yaml. No flags. | ✓ |
| Optional filter: /sync [connector...] | /sync alone = all, /sync jira = only jira. | |
| All + exclude flag | Default = all, can exclude with --skip. | |

**User's choice:** All connectors always

### Follow-up: Parallelism

| Option | Description | Selected |
|--------|-------------|----------|
| Sequential | One connector at a time. Simpler, predictable. | ✓ |
| Parallel where possible | Run independent connectors simultaneously. | |
| You decide | Claude picks best approach. | |

**User's choice:** Sequential

### Follow-up: Which connectors

| Option | Description | Selected |
|--------|-------------|----------|
| All 6 configured connectors | Calendar, Jira, Slack, Reminders, Granola, Training. | ✓ |
| Core 5 only (exclude training) | Training is different pattern. | |
| Only USYNC-01 scope | Jira + Granola + Calendar only. | |

**User's choice:** All 6 configured connectors

---

## Output & Reporting Format

| Option | Description | Selected |
|--------|-------------|----------|
| Status line per connector | One line per connector: icon + name + result. Clean, scannable. | ✓ |
| Summary table | Markdown table with columns: Connector, Status, Items, Notes. | |
| Minimal: only errors | Show nothing if all succeeds. Only report failures. | |

**User's choice:** Status line per connector

### Follow-up: Nested output in morning

| Option | Description | Selected |
|--------|-------------|----------|
| Morning shows /sync report inline | /morning includes sync status lines as part of its summary. | ✓ |
| Morning rewrites summary | /morning absorbs sync results and presents own format. | |
| You decide | Claude picks cleanest approach. | |

**User's choice:** Morning shows /sync report inline

---

## Existing Skills Fate

| Option | Description | Selected |
|--------|-------------|----------|
| Keep as implementation, remove as user skills | Remove description from frontmatter. Internal docs only. | ✓ |
| Keep all callable | Individual sync-* remain user-facing for debugging. | |
| Delete individual skills | Consolidate all logic into sync/SKILL.md. | |

**User's choice:** Keep as implementation, remove as user skills

### Follow-up: How to make internal

| Option | Description | Selected |
|--------|-------------|----------|
| Remove description field | Keep files in same location. Remove description from frontmatter. | ✓ |
| Move to sync/_internal/ | Move sync-* folders under skills/sync/_internal/. | |
| You decide | Claude picks cleanest approach. | |

**User's choice:** Remove description field

### Follow-up: Training inclusion

| Option | Description | Selected |
|--------|-------------|----------|
| Include in /sync | All sync connectors under /sync for consistency. | ✓ |
| Keep sync-training separate | Training is manual CSV import, different pattern. | |
| You decide | Claude evaluates fit. | |

**User's choice:** Include in /sync

---

## Claude's Discretion

- Connector execution order (sequential, deterministic)
- Error message formatting per connector failure mode
- How /sync internally references sync-* SKILL.md files
- Whether to track a global last_sync timestamp

## Deferred Ideas

None — discussion stayed within phase scope.
