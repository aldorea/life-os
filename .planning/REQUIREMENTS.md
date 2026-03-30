# Requirements: Life OS

**Defined:** 2026-03-28
**Core Value:** A reliable GTD system that captures everything, centralizes all task sources, and ensures nothing falls through the cracks.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Vault Foundation

- [x] **VAULT-01**: Vault has a consistent folder structure (PARA-inspired) with documented conventions
- [x] **VAULT-02**: All note types have standardized frontmatter schemas (tasks, people, meetings, knowledge, goals)
- [x] **VAULT-03**: Tagging system is defined and documented (contexts, priorities, status, domains)
- [x] **VAULT-04**: Existing vault content is migrated to the new structure without data loss

### GTD Core

- [x] **GTD-01**: User can quick-capture thoughts/tasks to inbox in under 5 seconds from CLI
- [x] **GTD-02**: User can process inbox items with AI-powered clarification (reformulate, tag, deduplicate)
- [x] **GTD-03**: User can view next actions filtered by GTD context (@home, @office, @calls, @computer)
- [x] **GTD-04**: User can manage projects list where every project has at least one next action
- [x] **GTD-05**: User can maintain a Someday/Maybe list reviewed periodically
- [x] **GTD-06**: User can detect stalled projects (no next action or no progress in X days)

### Daily & Weekly Rituals

- [x] **RITUAL-01**: User can generate daily note with agenda, focus, tasks, and goal alignment
- [x] **RITUAL-02**: User can run morning GTD ritual (sync sources, clean backlog, process inbox)
- [x] **RITUAL-03**: User can run weekly review (retrospective, inbox zero, backlog health, plan next week)
- [x] **RITUAL-04**: User can run end-of-day close ritual (reflection, energy check, move completed tasks)

### Goal Tracking

- [x] **GOAL-01**: User can define goals with measurable metrics, weights, deadlines, and status
- [x] **GOAL-02**: User can view goal progress across all dimensions (professional + personal)
- [x] **GOAL-03**: User can run quarterly reflection workflow (retrospective, start/stop/continue)
- [x] **GOAL-04**: Daily note shows focus-of-the-day aligned with current goals
- [x] **GOAL-05**: User can track goal progress over time (history, not just current state)

### Task Centralization

- [ ] **SYNC-01**: User can sync Jira tickets to vault with dynamic project configuration (add/remove without code changes)
- [x] **SYNC-02**: User can sync Apple Reminders to vault inbox
- [ ] **SYNC-03**: User can sync calendar events to vault for daily/weekly planning
- [ ] **SYNC-04**: User can sync Slack messages extracting actions and decisions from key channels
- [ ] **SYNC-05**: User can sync meeting notes from Granola (or any future provider) to vault
- [ ] **SYNC-06**: All synced items flow into a unified inbox for GTD processing

### Mini CRM

- [ ] **CRM-01**: User can view people notes auto-enriched from meetings (role, company, interests)
- [ ] **CRM-02**: User can see meeting history per person (when, what discussed, action items)
- [ ] **CRM-03**: User can get meeting prep briefing (person context, past meetings, pending tasks, related Jira tickets)
- [ ] **CRM-04**: User can detect stale contacts (no interaction beyond threshold) with follow-up alerts
- [ ] **CRM-05**: User can link people to professional goals (relationship map)

### Content Pipeline

- [ ] **CONT-01**: User can capture knowledge from work sessions into structured vault notes with topic tagging
- [ ] **CONT-02**: Knowledge notes track maturity (raw → developing → ready for publishing)
- [ ] **CONT-03**: User can get topic suggestions based on accumulated knowledge and expertise areas
- [ ] **CONT-04**: User can generate content drafts (LinkedIn posts, blog articles) from mature knowledge topics
- [ ] **CONT-05**: User can review and iterate on drafts before publishing
- [ ] **CONT-06**: Content pipeline tracks what's been published and performance (manually logged)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Web Dashboard

- **DASH-01**: User can view read-only web dashboard with goals progress, task overview, and status
- **DASH-02**: Dashboard pulls data directly from vault markdown files
- **DASH-03**: Dashboard shows CRM overview (recent meetings, stale contacts)
- **DASH-04**: Dashboard shows content pipeline status

### Advanced Features

- **ADV-01**: Cross-source action item deduplication (AI-powered matching across Jira + Slack + Granola)
- **ADV-02**: Stale goal detection with proactive nudges
- **ADV-03**: Content calendar / scheduling view
- **ADV-04**: Apple Reminders two-way sync (create reminders from vault)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Mobile app | Obsidian mobile + web dashboard covers mobile access |
| Real-time sync / live updates | Local-first vault; iCloud/Syncthing handles file sync |
| Auto-posting to social media | Platform APIs are fragile; use dedicated tools (Buffer, Typefully) |
| Habit tracking with gamification | Deep product category; use dedicated app or simple daily note checkboxes |
| Email integration | Noisy, privacy-sensitive, low signal-to-noise |
| Time tracking / Pomodoro | Commodity tools exist; integrate Toggl/Clockify if needed |
| Calendar event creation | Read-only sync is the right boundary; create events in native app |
| Multi-user / family sharing | Personal system; each person runs their own instance |
| AI auto-categorization without confirmation | GTD requires conscious processing; AI suggests, user confirms |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| VAULT-01 | Phase 1 | Complete |
| VAULT-02 | Phase 1 | Complete |
| VAULT-03 | Phase 1 | Complete |
| VAULT-04 | Phase 1 | Complete |
| GTD-01 | Phase 1 | Complete |
| GTD-02 | Phase 1 | Complete |
| GTD-03 | Phase 1 | Complete |
| GTD-04 | Phase 1 | Complete |
| GTD-05 | Phase 1 | Complete |
| GTD-06 | Phase 1 | Complete |
| RITUAL-01 | Phase 1 | Complete |
| RITUAL-02 | Phase 1 | Complete |
| RITUAL-03 | Phase 1 | Complete |
| RITUAL-04 | Phase 1 | Complete |
| GOAL-01 | Phase 1 | Complete |
| GOAL-02 | Phase 1 | Complete |
| GOAL-03 | Phase 1 | Complete |
| GOAL-04 | Phase 1 | Complete |
| GOAL-05 | Phase 1 | Complete |
| SYNC-01 | Phase 2 | Pending |
| SYNC-02 | Phase 2 | Complete |
| SYNC-03 | Phase 2 | Pending |
| SYNC-04 | Phase 2 | Pending |
| SYNC-05 | Phase 2 | Pending |
| SYNC-06 | Phase 2 | Pending |
| CRM-01 | Phase 3 | Pending |
| CRM-02 | Phase 3 | Pending |
| CRM-03 | Phase 3 | Pending |
| CRM-04 | Phase 3 | Pending |
| CRM-05 | Phase 3 | Pending |
| CONT-01 | Phase 3 | Pending |
| CONT-02 | Phase 3 | Pending |
| CONT-03 | Phase 3 | Pending |
| CONT-04 | Phase 3 | Pending |
| CONT-05 | Phase 3 | Pending |
| CONT-06 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 36 total
- Mapped to phases: 36
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-28*
*Last updated: 2026-03-28 after roadmap creation*
