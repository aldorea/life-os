---
name: sync-training
---

# sync-training

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that reads a CSV export from a fitness app and writes a structured training log in markdown format to the Obsidian vault.

## Process

### 1. Find and read the CSV

Check these locations in order:
1. `VAULT/{config.structure.training_log}` directory for CSV files
2. `{config.training.csv_search_path}` for files matching `{config.training.csv_pattern}` (most recent)
3. Ask the user

### 2. Parse the CSV

Typical CSV contains: date, exercise name, sets, reps, weight, notes.

Parse each row and group by:
- **Date** — each training session
- **Exercise** — within each session

### 3. Generate training log

Write to `VAULT/{config.structure.training_log}`:

```markdown
---
last_sync: YYYY-MM-DD HH:MM
source: CSV export
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
| Semana actual | X sesiones (objetivo: {config.training.weekly_goal}) |

## PRs (Records Personales)

| Ejercicio | Peso máx | Fecha | Reps |
|-----------|----------|-------|------|
| [exercise] | [weight] | [date] | [reps] |

## Sesiones

### YYYY-MM-DD (DíaSemana)

| Ejercicio | Series | Reps | Peso (kg) | Notas |
|-----------|--------|------|-----------|-------|
| [exercise] | [sets] | [reps] | [weight] | [notes] |
```

### 4. Graceful degradation

- If CSV not found: warn user, provide instructions for exporting
- If CSV format unexpected: parse what you can, warn about unparseable rows
- If training log already exists: merge new data, don't duplicate existing sessions

## Output

Single file: `VAULT/{config.structure.training_log}`
