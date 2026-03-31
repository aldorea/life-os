# Requirements: Life OS

**Defined:** 2026-03-28
**Redefined:** 2026-03-31
**Core Value:** Ayudar a decidir qué hacer cada semana, desglosarlo, organizarlo y ejecutarlo.

## v1.0 Requirements (Completed)

Phases 1-2 delivered. Skills exist but not fully validated by user.

### Vault Foundation (Phase 1)
- [x] VAULT-01 through VAULT-04: Vault structure, schemas, tagging, migration

### GTD Core (Phase 1)
- [x] GTD-01 through GTD-06: Capture, processing, views, projects, someday

### Rituals (Phase 1)
- [x] RITUAL-01 through RITUAL-04: Daily note, morning, weekly review, close

### Goal Tracking (Phase 1)
- [x] GOAL-01 through GOAL-05: Goal CRUD, progress, quarterly, alignment

### Sync (Phase 2)
- [x] SYNC-01 through SYNC-06: Jira, Reminders, Calendar, Slack, Granola, unified inbox

## v1.1 Requirements (Active)

Focused on the user's real pain: planning, organizing, and goal definition.

### Unified Sync

- [x] **USYNC-01**: Un solo comando `/sync` centraliza Jira (Afianza + Previene), Granola, y calendario en el vault
- [x] **USYNC-02**: Cada conector falla independientemente — si Jira falla, calendario sigue
- [x] **USYNC-03**: Output claro de qué se sincronizó y qué falló, sin ruido

### Weekly Planning

- [ ] **PLAN-01**: `/plan-week` lee backlog, goals.yaml, y calendario para proponer un plan semanal
- [ ] **PLAN-02**: Flujo interactivo: Claude pregunta "qué quieres conseguir esta semana", propone, usuario ajusta
- [ ] **PLAN-03**: Genera weekly note con prioridades, capacidad estimada, y tareas asignadas a la semana
- [ ] **PLAN-04**: Cruza tareas del backlog con goals activos para sugerir qué priorizar
- [ ] **PLAN-05**: Identifica arrastres de semanas anteriores y los surfacea

### Weekly Review

- [ ] **REV-01**: `/review` guía el cierre semanal: qué se completó, qué se arrastra, reflexión
- [ ] **REV-02**: Actualiza goals.yaml si hubo progreso en algún objetivo
- [ ] **REV-03**: Detecta tareas estancadas (>2 semanas sin moverse) y las surfacea

### Goal Definition

- [ ] **GDEF-01**: `/define-goals` facilita la definición de objetivos trimestrales mediante conversación guiada
- [ ] **GDEF-02**: Ayuda a desglosar un objetivo abstracto en hitos concretos con fechas
- [ ] **GDEF-03**: Genera/actualiza goals.yaml con el resultado
- [ ] **GDEF-04**: Valida coherencia: ¿hay demasiados goals? ¿las fechas son realistas? ¿hay conflictos de tiempo?

## Future (Not Now)

| Feature | When |
|---------|------|
| CRM / people tracking | Cuando el sistema base esté validado |
| Content generation | Cuando haya knowledge capture real |
| Web dashboard | Cuando haya necesidad de vista rápida fuera de Obsidian |
| Meeting prep briefings | Cuando CRM exista |

## Out of Scope

| Feature | Reason |
|---------|--------|
| Mobile app | Obsidian mobile es suficiente |
| Calendar event creation | Solo lectura. Bloqueo es manual en Outlook |
| Auto-posting social media | Usar herramientas dedicadas |
| Email integration | Ruido, privacidad, bajo valor |
| Skills especulativas | Solo construir lo que resuelve un dolor validado |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| USYNC-01..03 | Phase 3 (Sync) | Pending |
| PLAN-01..05 | Phase 4 (Planning) | Pending |
| REV-01..03 | Phase 5 (Review) | Pending |
| GDEF-01..04 | Phase 6 (Goals) | Pending |

---
*Last updated: 2026-03-31 — redefined with user input*
