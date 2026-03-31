# Roadmap: Life OS

## Overview

Redefined 2026-03-31. Phases 1-2 delivered GTD core + sync connectors (26 skills, not fully validated). The project pivots to focus on the user's real pain: planning what to do, breaking it down, and staying on track. Four new phases, each small and validated before moving to the next.

## Phases

**Completed (v1.0):**
- [x] **Phase 1: GTD Core** — Vault schema, capture, processing, rituals, goal tracking (7 plans)
- [x] **Phase 2: External Integrations** — Jira, Calendar, Granola, Slack, Reminders sync (3 plans)

**Active (v1.1):**
- [ ] **Phase 3: Unified Sync** — Single `/sync` command replacing 5 separate sync skills
- [ ] **Phase 4: Weekly Planning** — `/plan-week` to decide what to do each week
- [ ] **Phase 5: Weekly Review** — `/review` to close the week and adjust
- [ ] **Phase 6: Goal Definition** — `/define-goals` to set and break down objectives

## Phase Details

### Phase 1: GTD Core
**Goal**: Complete daily productivity loop with zero external dependencies
**Status**: Complete (2026-03-29)
**Plans:** 7/7 complete

### Phase 2: External Integrations
**Goal**: External data flows into vault automatically
**Status**: Complete (2026-03-30)
**Plans:** 3/3 complete

### Phase 3: Unified Sync
**Goal**: Un solo comando `/sync` que centraliza Jira (Afianza + Previene), Granola, y calendario en el vault — reemplazando los 5 sync individuales
**Depends on**: Phase 2 (sync connectors exist)
**Requirements**: USYNC-01, USYNC-02, USYNC-03
**Success Criteria**:
  1. `/sync` ejecuta todos los conectores configurados en `connectors.yaml` y reporta qué se sincronizó
  2. Si un conector falla, los demás continúan y el usuario ve qué falló
  3. El usuario valida que lo usa diariamente durante 1 semana antes de pasar a Phase 4
**UI hint**: no
**Plans:** 2 plans

Plans:
- [x] 03-01-PLAN.md — Create /sync orchestrator skill + make sync-* skills internal
- [ ] 03-02-PLAN.md — Simplify /morning to delegate sync to /sync

### Phase 4: Weekly Planning
**Goal**: El usuario puede planificar su semana en una conversación guiada con Claude — cruzando goals, backlog, y calendario
**Depends on**: Phase 3 (sync must work so backlog is current)
**Requirements**: PLAN-01, PLAN-02, PLAN-03, PLAN-04, PLAN-05
**Success Criteria**:
  1. `/plan-week` lee backlog + goals + calendario y propone un plan semanal
  2. El usuario ajusta interactivamente y genera la weekly note
  3. Las tareas arrastradas de semanas anteriores se surfacean automáticamente
  4. El usuario valida que lo usa 2 lunes consecutivos antes de pasar a Phase 5
**UI hint**: no
**Plans**: TBD

### Phase 5: Weekly Review
**Goal**: El usuario cierra la semana con una retrospectiva guiada que actualiza goals y detecta estancamientos
**Depends on**: Phase 4 (weekly planning creates the data to review)
**Requirements**: REV-01, REV-02, REV-03
**Success Criteria**:
  1. `/review` guía retrospectiva: completadas, arrastres, reflexión
  2. Goals se actualizan si hubo progreso
  3. Tareas estancadas (>2 semanas) se surfacean
  4. El usuario valida que lo usa 2 viernes consecutivos antes de pasar a Phase 6
**UI hint**: no
**Plans**: TBD

### Phase 6: Goal Definition
**Goal**: El usuario puede definir y desglosar objetivos trimestrales con ayuda de Claude — pasando de "quiero hacer X" a hitos concretos con fechas
**Depends on**: Phase 5 (review reveals which goals need definition)
**Requirements**: GDEF-01, GDEF-02, GDEF-03, GDEF-04
**Success Criteria**:
  1. `/define-goals` facilita definición mediante conversación guiada
  2. Objetivos abstractos se desglosan en hitos con fechas
  3. goals.yaml se actualiza con el resultado
  4. Claude valida coherencia (demasiados goals, fechas irrealistas, conflictos)
**UI hint**: no
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 3 -> 4 -> 5 -> 6
Each phase gated by user validation before proceeding.

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. GTD Core | 7/7 | Complete | 2026-03-29 |
| 2. External Integrations | 3/3 | Complete | 2026-03-30 |
| 3. Unified Sync | 0/2 | Planning complete | - |
| 4. Weekly Planning | 0/TBD | Not started | - |
| 5. Weekly Review | 0/TBD | Not started | - |
| 6. Goal Definition | 0/TBD | Not started | - |
