---
name: train
description: Use when the user wants to check training history, personal records, progress, or says "train", "entrenamiento", "training", "gimnasio", "PRs", "cuánto levanto", "progreso gym", "heavy".
---

# train

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Query skill that reads the training log and answers questions about exercise history, personal records, volume trends, and consistency. Does not modify files.

## Process

### 1. Read training data

Read `VAULT/{config.structure.training_log}`. If it doesn't exist, suggest running `sync-training` first.

### 2. Answer the query

| Query type | Example | How to answer |
|-----------|---------|---------------|
| **PRs** | "¿Cuál es mi PR de peso muerto?" | Find max weight for that exercise |
| **Progress** | "¿Cómo va mi press banca en 3 meses?" | Show weight trend over time |
| **Consistency** | "¿Cuántas veces fui al gym este mes?" | Count sessions per week/month |
| **Last session** | "¿Qué hice la última vez?" | Show most recent session |
| **Volume** | "¿Cuánto volumen hice de pierna?" | Sum sets x reps x weight |
| **Comparison** | "Compara mis últimas 4 semanas" | Weekly summary table |
| **Goal check** | "¿Estoy yendo X veces por semana?" | Check against `{config.training.weekly_goal}` from goals |

### 3. Format response

For PRs and progress, use tables:

```
### PR — Peso muerto
| Fecha | Peso | Reps | Estimado 1RM |
|-------|------|------|-------------|
| 2026-03-01 | 120kg | 3 | ~127kg |
```

For consistency:
```
### Consistencia — Marzo 2026
| Semana | Sesiones | Objetivo |
|--------|----------|----------|
| W10 | 2 | {config.training.weekly_goal} ❌ |
| W11 | 3 | {config.training.weekly_goal} ✅ |
```

### 4. Graceful degradation

- No training log: suggest running `sync-training`
- Exercise not found: ask if it has another name
- Partial data: work with what's available, note gaps

## Important Rules

- Read-only — NEVER modify training-log.md
- Use 1RM estimation only if user asks: `{config.training.one_rm_formula}`
- Match exercise names fuzzy (e.g., "peso muerto" = "deadlift")
- Reference weekly goal from config when showing consistency
