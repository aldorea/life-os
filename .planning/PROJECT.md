# Life OS

## What This Is

Un sistema personal de productividad que ayuda a centralizar tareas de múltiples fuentes, planificar la semana, y mantener el foco en los objetivos. Funciona como skills de Claude Code sobre un vault de Obsidian existente.

## Core Value

Ayudar a Alfonso a decidir qué hacer cada semana, desglosarlo, organizarlo y ejecutarlo — resolviendo la fricción entre "tengo muchas cosas" y "no sé por dónde empezar".

## Requirements

### Validated (Phase 1-3, archived)

- [x] Vault schema, GTD capture, processing, daily/weekly rituals, goal tracking (Phase 1)
- [x] Sync connectors: Jira, Calendar, Granola, Slack, Reminders (Phase 2)
- [x] Unified sync: /sync centraliza todos los conectores, /morning simplificado (Phase 3)

### Active

- [x] Sync unificado: un solo comando centraliza Jira (Afianza + Previene), Granola, y calendario en el vault — Validated in Phase 3: Unified Sync
- [ ] Planificación semanal: cruzar goals + backlog + calendario → plan semanal con prioridades y capacidad
- [ ] Cierre semanal: retrospectiva, arrastres, reflexión, actualización de goals
- [ ] Definición de objetivos: facilitar la definición y desglose de goals trimestrales/anuales

### Out of Scope

- Web dashboard — no aporta valor ahora, el vault es la interfaz
- CRM / people tracking — futuro, no es el dolor actual
- Content generation pipeline — futuro
- Mobile app — Obsidian mobile es suficiente
- Auto-posting social media
- Skills especulativas que no resuelven un dolor real

## Context

- Vault de Obsidian en iCloud con estructura funcional: `01 Backlog.md`, `03 Daily/`, `04 Weekly/`, `config/goals.yaml`, `config/connectors.yaml`
- MCP servers configurados: Jira (Afianza + Previene), Slack, Granola
- 3 calendarios: Apple Calendar (personal), Gmail (Orbitant), Outlook (Afianza) — bloqueo de tiempo en Outlook
- Captura en papel → pasa a Obsidian manualmente
- 26 skills existentes de fases anteriores — la mayoría sin validar por el usuario
- El usuario ya hace GTD en Obsidian pero le falta: planificación semanal, desglose de tareas, definición de objetivos

## Constraints

- **Vault**: Obsidian-first — todo vive como markdown en el vault existente
- **Skills**: Pocas, validadas, que resuelvan dolor real — no construir especulativamente
- **Config**: Reutilizar `config/connectors.yaml` y `config/goals.yaml` existentes
- **Calendarios**: Solo lectura. Bloqueo de tiempo es manual en Outlook.
- **Jira**: Dos instancias (Afianza vía jira-afianza MCP, Previene vía jira-previene MCP)

## Key Decisions

| Decision | Rationale | Date |
|----------|-----------|------|
| Redefinir proyecto: de 26 skills a 4 core | 26 skills sin validar abrumaron al usuario. Foco en el dolor real: planificar y organizar | 2026-03-31 |
| Vault existente se mantiene tal cual | La estructura actual (backlog, daily, weekly, goals.yaml) funciona. No hay que migrar nada | 2026-03-31 |
| No web dashboard en v1 | No aporta valor vs la fricción de planificación semanal | 2026-03-31 |
| Usage gate: validar cada skill antes de construir la siguiente | Lección aprendida de fases 1-2 donde se construyó sin validar | 2026-03-31 |

## Evolution

- Phase 1-2 artifacts archived in `.planning/phases/01-gtd-core/` and `.planning/phases/02-external-integrations/`
- Phase 3 (original Compound Intelligence) replaced by focused v1.1 milestone

---
*Last updated: 2026-03-31 — project redefined with user to focus on planning/organizing pain point*
