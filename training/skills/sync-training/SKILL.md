---
name: sync-training
description: DEPRECATED 2026-05-07. Heavy app no longer used. Training data is now captured via Telegram bot — sessions go through `sync-telegram` to `10 Health/Training/{date}.md` automatically. DO NOT invoke this skill. Kept as archive only.
---

# sync-training (DEPRECATED)

> **Deprecated 2026-05-07**: Heavy app no longer used. Training capture now flows through the Telegram bot → `sync-telegram` → `10 Health/Training/{DD-MM-YYYY}.md`. Free-text format with `#training-raw` tag. The `train` skill reads from there.
>
> Do not invoke this skill. The CSV import path is no longer maintained. Kept for archive / git history.

## Overview (legacy)

Connector that reads a CSV export from the Heavy app and writes a structured training log in markdown format to the Obsidian vault.

## Prerequisites

- User must export CSV from Heavy and place it somewhere accessible
- Default location: `10 Health/heavy-export.csv`
- If file not found, ask the user where the CSV is

## Process

### 1. Find and read the CSV

Check these locations in order:
1. `10 Health/heavy-export.csv`
2. `~/Downloads/` for any file matching `*heavy*` or `*Heavy*` (most recent)
3. Ask the user

### 2. Parse the CSV

Heavy CSV typically contains: date, exercise name, sets, reps, weight, notes.

Parse each row and group by:
- **Date** — each training session
- **Exercise** — within each session

### 3. Generate training log

Write to `10 Health/training-log.md`:

```markdown
---
last_sync: YYYY-MM-DD HH:MM
source: Heavy (CSV export)
total_sessions: N
date_range: YYYY-MM-DD — YYYY-MM-DD
---

# Training Log

## Resumen

| Métrica | Valor |
|---------|-------|
| Sesiones totales | N |
| Última sesión | YYYY-MM-DD |
| Ejercicios únicos | N |
| Semana actual | X sesiones (objetivo: 3) |

## Sesiones

### YYYY-MM-DD (DíaSemana)

| Ejercicio | Series | Reps | Peso (kg) | Notas |
|-----------|--------|------|-----------|-------|
| Peso muerto | 4 | 8, 8, 6, 5 | 100 | PR |
| Press banca | 3 | 10, 10, 8 | 60 | — |

### YYYY-MM-DD (DíaSemana)
...
```

### 4. Generate PRs summary

At the top of the file, include a personal records section:

```markdown
## PRs (Records Personales)

| Ejercicio | Peso máx | Fecha | Reps |
|-----------|----------|-------|------|
| Peso muerto | 120kg | YYYY-MM-DD | 1 |
| Sentadilla | 100kg | YYYY-MM-DD | 3 |
```

### 5. Graceful degradation

- If CSV not found: warn user, provide instructions for exporting from Heavy
- If CSV format unexpected: parse what you can, warn about unparseable rows
- If training-log.md already exists: merge new data, don't duplicate existing sessions

## Output

Single file: `10 Health/training-log.md`

## Vault Path

The Obsidian vault is at: `/Users/alfonso/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault`
