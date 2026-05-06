---
name: review
description: Use when doing a GTD weekly review, planning the next week, or user says "review", "weekly", "revisión semanal", "planifica semana", "semana", "GTD review".
---

# review

## Overview

Full GTD Weekly Review. The most important ritual — where the system stays trustworthy. Combines brain dump, inbox processing, backlog health review, weekly planning, and weekly note generation in one guided session (~15 min).

## Pipeline

```
Brain dump (capture loose thoughts)
       │
       ▼
Process Inbox (clarify + organize)
       │
       ▼
Review Backlog (stale, vague, tags, overload, goals)
       │
       ▼
Plan next week (priorities, capacity, tag tasks)
       │
       ▼
Generate weekly note (retrospective + planning)
```

## Process

### Step 1: Brain dump (~2 min)

> **Antes de empezar la revisión: ¿hay algo dándote vueltas que no esté capturado?**
> (Suelta todo — ya lo procesamos en el siguiente paso)

Capture everything to `02 Inbox.md` as raw items. Zero friction, zero judgment.
If user says "nada" / "no" → skip.

### Step 2: Process Inbox (~3 min)

Run `obsidian vault="Obsidian Vault" read path="02 Inbox.md"`. If items exist:

1. **Reformulate** vague items → concrete actions (verb + object + context)
2. **Apply GTD decision tree:**
   - Not actionable → trash / reference / someday
   - Actionable <2 min → suggest doing now (2-min rule)
   - Actionable >2 min, multi-step → project with subtasks
   - Actionable >2 min, single step → next action
3. **Detect duplicates** against `01 Backlog.md`
4. **Suggest tags** (context, project, priority per `08 Resources/claude-memory/feedback_priority_criteria.md`)
5. **Present table** → wait for user confirmation
6. **Apply**: add to Backlog sections, clear Inbox

If Inbox empty → skip silently.

### Step 3: Review Backlog (~5 min)

List all task notes: `obsidian vault="Obsidian Vault" files folder="01 Tasks"`. Read frontmatter of each note to check status/priority/week. Also read `obsidian vault="Obsidian Vault" read path="config/goals.yaml"`. Run these checks:

#### 3a. Stale tasks
Notes with `week` property < current week and `status != "done"`.
> **Tareas de semanas pasadas:**
> - [[IDP-confirmar-JWT-TTL]] (week: 2026-W17) — 2 semanas
> Para cada una: ¿mover a esta semana / algún día / eliminar?
Apply: `obsidian vault="Obsidian Vault" property:set name=week value="YYYY-W##" path="01 Tasks/{file}"`

#### 3b. Vague tasks
Notes with title that lacks a concrete verb or is too abstract.
> **Tareas poco concretas:**
> - [[tema-api]] → ¿Qué acción?
Suggest reformulations, update title if confirmed.

#### 3c. Property consistency
- Notes with `status: todo` but `week` filled in → should be `in-progress`
- Notes with `status: waiting` without context in body → add note
- Notes with empty `project` → suggest one

#### 3d. Week overload
Count notes where `week == next-week`. If >10:
> **Semana sobrecargada:** X tareas. ¿Mover algunas?

#### 3e. Goal alignment
Compare notes by `project` vs `config/goals.yaml` in_progress goals:
- Any `in_progress` goal with no tasks for this/next week?
- Any `due` date approaching (<2 weeks)?
Flag only if clearly relevant.

Apply all confirmed changes with `obsidian vault="Obsidian Vault" property:set name={prop} value={val} path="01 Tasks/{file}"`.

### Step 4: Plan next week (~3 min)

1. **Calculate capacity** — read with `obsidian vault="Obsidian Vault" read path="config/constraints.yaml"` and `obsidian vault="Obsidian Vault" read path="08 Resources/calendar-cache.md"`:
   - Work hours, meeting hours, focus time available
   - Flag heavy days, free days

2. **Suggest priorities** (top 3 work, top 2 personal) based on:
   - Sprint tickets In Progress → always first
   - `#urgente` + `#importante` tasks
   - Goal deadlines approaching
   - User preferences from `08 Resources/claude-memory/preferences.md`

3. **Ask user to assign tasks** to next week
   - Present suggestions, user confirms
   - Never auto-assign
   - Apply: `obsidian vault="Obsidian Vault" property:set name=week value="YYYY-W##" path="01 Tasks/{file}"`

### Step 5: Generate weekly note (~2 min)

#### Retrospective (from previous week's dailies)
List files with `obsidian vault="Obsidian Vault" files folder="03 Daily"`, then read each daily note for the ending week with `obsidian vault="Obsidian Vault" read path="03 Daily/{date}.md"`:
- Tasks completed (count work vs personal)
- Energy trend (from Cierre entries)
- Key wins mentioned
- Tasks dragged (pending >2 weeks)

#### Template: `04 Weekly/YYYY-W##.md`

```markdown
---
tags: weekly
week: YYYY-W##
date_start: YYYY-MM-DD
date_end: YYYY-MM-DD
---

# Semana ## · YYYY
`DD MMM` → `DD MMM`

---

## Retrospectiva semana anterior

**Completadas:** X tareas (Y trabajo, Z personal)
**Energía media:** 🟢/🟡/🔴
**Qué fue bien:** [from dailies]
**Qué mejorar:** [from dailies]
**Arrastradas:** [tasks pending >2 weeks]

---

## Prioridades de la semana

### Trabajo
1. [Confirmed by user]
2. ...

### Personal
1. [Confirmed by user]
2. ...

---

## Capacidad

| Métrica | Valor |
|---------|-------|
| Horas laborables | Xh |
| Reuniones programadas | Xh (Y reuniones) |
| Foco disponible | ~Xh |
| Día más libre | [day] |
| Día más cargado | [day] |

---

## Tareas de esta semana
```tasks
not done
(path includes 01 Backlog) OR (path includes 06 Jira)
(description includes #YYYY-W##) OR (description includes #next)
```

---

## Completadas esta semana
```tasks
done
done after YYYY-MM-DD
done before YYYY-MM-DD
```

---

## Reflexión
**¿Qué fue bien esta semana?**

**¿Qué mejorar?**

**¿Algo que deba eliminar o delegar?**
```

### 6. Summary

```
Revisión semanal completada:

📥 Brain dump: X items capturados
📋 Inbox: X procesados → Backlog
🔍 Backlog: X revisados (Y corregidos, Z movidos)
📅 Semana W##: X tareas planificadas, Xh foco disponible
📊 Retro W##: X completadas, energía media 🟡
⚠️ [alertas: goals en riesgo, tareas arrastradas >3 semanas]
```

## Important Rules

- Each step is INTERACTIVE — wait for user input before proceeding
- User can stop at any step ("ya está", "suficiente", "skip")
- Skip empty steps automatically (empty Inbox, clean Backlog)
- Never auto-assign week tags — always confirm with user
- Never delete tasks without confirmation
- Priorities are SUGGESTIONS — user decides
- Sprint tickets are untouchable — they come from Jira planning, not the weekly review
- Keep `tasks` code blocks as Obsidian plugin queries, not static lists
- Retrospective only if previous week dailies exist

## Vault I/O

Use `obsidian vault="Obsidian Vault"` CLI for all vault reads and writes — never use the Read tool directly on vault files.
