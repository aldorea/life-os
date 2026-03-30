# Roadmap: Life OS

## Overview

Three phases that build on each other. Phase 1 ships a complete GTD loop (vault schema + daily rituals + goal tracking) that works with zero external dependencies. Phase 2 connects external sources (Jira, Calendar, Granola, Slack, Reminders) so the vault populates itself. Phase 3 unlocks compound intelligence that only becomes meaningful once data has been accumulating — meeting prep briefs, content generation, and CRM follow-up loops. Each phase is gated by daily use of the prior phase before proceeding.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: GTD Core** - Complete daily productivity loop: vault schema, capture, processing, rituals, and goal tracking — zero external dependencies
- [ ] **Phase 2: External Integrations** - All sync connectors (Jira, Calendar, Granola, Slack, Reminders) flowing into the unified inbox
- [ ] **Phase 3: Compound Intelligence** - CRM and content pipeline features that compound as data accumulates

## Phase Details

### Phase 1: GTD Core
**Goal**: User has a complete, reliable daily productivity loop that works with zero external dependencies
**Depends on**: Nothing (first phase)
**Requirements**: VAULT-01, VAULT-02, VAULT-03, VAULT-04, GTD-01, GTD-02, GTD-03, GTD-04, GTD-05, GTD-06, RITUAL-01, RITUAL-02, RITUAL-03, RITUAL-04, GOAL-01, GOAL-02, GOAL-03, GOAL-04, GOAL-05
**Success Criteria** (what must be TRUE):
  1. User can capture a thought to inbox in under 5 seconds from the CLI with no friction (no form, no classification)
  2. User can process inbox items with AI-powered clarification that assigns GTD context tags, deduplicates, and routes to next actions, projects, or someday/maybe
  3. User can generate a daily note that shows goal-aligned focus, today's agenda, and top 3 next actions — in one command
  4. User can run a weekly review that guides them through retrospective, inbox zero, backlog health, and next week planning as an interactive facilitator
  5. User can define goals with metrics and deadlines, view progress across all dimensions, and run a quarterly reflection workflow
**Plans:** 7 plans

Plans:
- [x] 01-01-PLAN.md — Vault schema definition + config.example.yaml update
- [x] 01-02-PLAN.md — Vault migration skill (/migrate-vault)
- [x] 01-03-PLAN.md — Goal tracking system (/goal + /status upgrade)
- [x] 01-04-PLAN.md — Capture & processing upgrades (/dump, /capture, /process-inbox)
- [x] 01-05-PLAN.md — GTD view skills (/next-actions, /projects, /someday)
- [ ] 01-06-PLAN.md — Daily rituals (/today upgrade, /morning, /close)
- [x] 01-07-PLAN.md — Weekly review + quarterly reflection (/weekly-review, /week upgrade, /quarterly)

### Phase 2: External Integrations
**Goal**: External data (Jira, Calendar, meeting notes, Slack, Reminders) flows into the vault automatically without manual entry
**Depends on**: Phase 1
**Requirements**: SYNC-01, SYNC-02, SYNC-03, SYNC-04, SYNC-05, SYNC-06
**Success Criteria** (what must be TRUE):
  1. User can sync Jira tickets from multiple configured projects into vault with a single command — adding or removing projects requires only a config file change, no code change
  2. User can sync calendar events and Apple Reminders into the vault so the daily note reflects real-world commitments without manual entry
  3. User can sync Granola meeting notes and Slack channel extracts into vault with all synced items flowing through the unified GTD inbox
  4. Each sync connector is independently degradable — a broken Jira sync never blocks daily note generation; stale data is labeled with last-sync timestamp
**Plans:** 3 plans

Plans:
- [ ] 02-01-PLAN.md — Jira sync connector + connectors.example.yaml
- [ ] 02-02-PLAN.md — Apple Reminders sync connector
- [ ] 02-03-PLAN.md — Morning orchestrator wiring + unified inbox flow

### Phase 3: Compound Intelligence
**Goal**: The system compounds — each meeting makes the next more informed, accumulated knowledge becomes publishable content, and stale relationships surface proactively
**Depends on**: Phase 2
**Requirements**: CRM-01, CRM-02, CRM-03, CRM-04, CRM-05, CONT-01, CONT-02, CONT-03, CONT-04, CONT-05, CONT-06
**Success Criteria** (what must be TRUE):
  1. User can get a pre-meeting briefing that aggregates person context, past meeting history, pending action items, and related Jira tickets into a single note
  2. User can capture knowledge from work sessions into structured vault notes with maturity tracking (raw → developing → ready), and get topic suggestions when notes reach "ready" state
  3. User can generate a LinkedIn post or blog article draft from a mature knowledge note in one command, then review and iterate before publishing
  4. User can see stale contacts (no interaction beyond threshold) surfaced proactively with follow-up suggestions
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. GTD Core | 0/7 | Planning complete | - |
| 2. External Integrations | 0/3 | Planning complete | - |
| 3. Compound Intelligence | 0/TBD | Not started | - |
