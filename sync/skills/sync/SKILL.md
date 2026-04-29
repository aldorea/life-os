---
name: sync
description: Run all enabled connectors (calendar, granola, slack, jira, training, telegram) sequentially and return a single status report. Use when user says "sync", "sincroniza", "refresca datos", or is invoked by /morning.
---

# sync

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

Read `VAULT/{config.structure.connectors_config}` to determine which connectors are enabled (`connectors.<name>.enabled: true`). If a connector is not enabled or the key is missing, skip it and report "skipped".

## Overview

Aggregator. Runs each enabled connector skill in sequence. Each connector is independent -- if one fails, the rest continue. Returns a single status block (check / X / dash per connector) that other skills (like `/morning`) can include verbatim.

## Process

For each connector in this order -- calendar, granola, slack, jira, training, telegram -- check its `enabled` flag in `connectors.yaml`:

| Connector | If enabled | If disabled |
|-----------|-----------|-------------|
| calendar  | Execute `sync-calendar` skill | `-` skipped (disabled) |
| granola   | Execute `sync-granola` skill | `-` skipped (disabled) |
| slack     | Execute `sync-slack` skill | `-` skipped (disabled) |
| jira      | Execute `sync-jira` skill | `-` skipped (disabled) |
| training  | Execute `sync-training` skill **only if `run_on != manual`** | `-` skipped (manual / disabled) |
| telegram  | Execute `sync-telegram` skill **only if `run_on == session_start`** (skip if `run_on == close`) | `-` skipped (runs at close / disabled) |

Capture per-connector result: `ok` (with one-line detail) | `error` (with error message) | `skipped` (with reason).

**Important:** if any sub-skill fails, catch the error and continue. Sync as a whole should never throw.

## Status report

Return exactly this block (no extra wrapping text). Consumers like `/morning` will include it verbatim under a `[Sync]` section:

```
Sync completado:
- [✓] Calendar — {one-line detail or "sin cambios"}
- [✓] Granola — {X meetings procesadas / "sin nuevas"}
- [✓] Slack — {X items al inbox / "sin menciones"}
- [—] Jira — skipped (disabled in connectors.yaml)
- [—] Training — skipped (run_on: manual)
- [—] Telegram — skipped (runs at close)
```

Legend:
- `[✓]` ok
- `[✗]` error (include short error message)
- `[—]` skipped (include reason)

## Graceful Degradation

| Scenario | Behavior |
|----------|----------|
| `connectors.yaml` missing | Warn user, default to running calendar + granola only |
| A connector skill not found | Mark as `[✗] not installed` and continue |
| Vault path missing | Abort — can't sync without vault |

## Important Rules

- **Read-only aggregator** — does not modify data directly; only dispatches to connector skills.
- **Sequential, not parallel** — avoids file conflicts on shared outputs (inbox, backlog).
- **Single status block** — the whole point is one cohesive report; do not print per-connector narration before the block.
- Spanish for user-facing output.
