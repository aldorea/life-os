# Feature Research

**Domain:** Personal Productivity / Life OS (GTD + Knowledge Management + CRM + Content Pipeline)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Feature Landscape

### Table Stakes (Users Expect These)

Features that any serious GTD-based Life OS must have. Missing these means the system cannot serve as a trusted external brain.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Quick capture to inbox** | GTD foundation -- brain dump must be frictionless, zero-friction or users revert to sticky notes | LOW | Already built (`capture` skill). Must work from CLI in under 5 seconds. |
| **Inbox processing / clarify** | Raw captures are useless without clarification into actionable tasks | MEDIUM | Already built (`process-inbox` skill). Key: reformulate vague items, deduplicate, tag. |
| **Next actions list** | Core GTD -- users need a single "what do I do now?" view filtered by context | LOW | Partially exists via Backlog + tags. Needs a clean context-filtered view. |
| **Projects list with next actions** | GTD requires every project to have at least one next action or it stalls | MEDIUM | Backlog sections exist but no explicit project-level tracking with stall detection. |
| **Someday/Maybe list** | GTD bucket for ideas not committed to -- prevents backlog bloat | LOW | Exists as last Backlog section. Needs periodic review prompts. |
| **Weekly review** | GTD's most critical habit -- without it the system decays within 2 weeks | MEDIUM | Already built (`week` skill). Includes retrospective + planning. |
| **Daily note generation** | Surfaces today's agenda, focus, and tasks in one view | MEDIUM | Already built (`today` skill). Calendar + goals + tasks synthesized. |
| **Calendar integration** | Users need to see meetings alongside tasks to plan realistically | LOW | Already built (`sync-calendar` skill via icalBuddy). |
| **External task centralization (Jira)** | Professional users manage work across Jira instances -- must see everything in one place | HIGH | Partially built (Jira MCP configured, sync-jira skill exists). Dynamic project config needed. |
| **Meeting notes capture** | Meetings generate decisions and action items -- losing them is unacceptable | MEDIUM | Already built (`sync-granola` skill). Source-agnostic design is correct. |
| **People notes / Mini CRM** | Knowing context about who you meet (role, last interaction, personality) makes meetings effective | MEDIUM | Already built via `sync-granola` People enrichment + `prep` skill. |
| **Search across vault** | Users must find anything they've captured -- notes, tasks, people, meetings | LOW | Obsidian handles this natively (full-text search + Dataview). No custom build needed. |
| **Goal setting with tracking** | Without goals, tasks are busy-work -- need quarterly objectives with measurable progress | MEDIUM | Goals YAML exists, referenced by `status` and `today` skills. Needs structured review workflow. |
| **Status dashboard (CLI)** | Quick "how am I doing?" across all dimensions | LOW | Already built (`status` skill). Read-only aggregation. |
| **Vault structure / organization** | Markdown files must be organized predictably or Obsidian becomes a junk drawer | MEDIUM | Config-driven structure exists. Vault redesign is an active requirement. |

### Differentiators (Competitive Advantage)

Features that set this system apart from Notion Life OS templates, generic GTD apps, and off-the-shelf tools.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **AI-powered inbox clarification** | LLM reformulates vague captures into actionable tasks with smart tagging -- no other GTD tool does this natively | LOW | Already works via `process-inbox`. This is the killer feature vs. manual GTD apps. |
| **Meeting prep briefing** | Aggregates people context, past meetings, pending tasks, and Jira tickets into a pre-meeting brief -- saves 10-15 min per meeting | MEDIUM | Already built (`prep` skill). Competitors like Clay/Clarify do CRM but not meeting-prep-from-vault. |
| **Knowledge-to-content pipeline** | Captured knowledge auto-tracks maturity and suggests when topics are ready for publishing | MEDIUM | Already built (`capture` + `content` skills). Unique: knowledge notes accumulate entries and trigger content generation at maturity threshold. |
| **Source-agnostic meeting sync** | Abstract meeting ingestion away from any provider (Granola today, Otter tomorrow) -- prevents vendor lock-in | LOW | Architecture decision already made. Most systems are tightly coupled to one transcription tool. |
| **Cross-source action item extraction** | Slack, Granola, Calendar, Jira all feed into one Inbox with deduplication | HIGH | Partially built (Granola + Slack sync exist). Unique: AI deduplicates and groups related items across sources. |
| **CLI-first with AI agent** | No context-switching to a web UI for actions. Conversational interface via Claude Code skills. Power users prefer this. | LOW | Core interaction model. Notion/ClickUp require GUI. This is the Vim-vs-VSCode differentiator. |
| **Goal-aligned daily focus** | Daily note suggests "focus of the day" by cross-referencing goals, calendar, and backlog -- not just listing tasks | LOW | Already built in `today` skill. Most tools show tasks; this shows what matters. |
| **People enrichment from meetings** | Every meeting auto-creates/updates contact notes with role, company, last interaction -- CRM builds itself | MEDIUM | Already built in `sync-granola`. Personal CRMs (Clay, Folk) require manual entry or email integration. |
| **Quarterly/Annual reflection workflow** | Structured goal review with retrospective, not just goal-setting -- closes the feedback loop | MEDIUM | Not yet built. OKR best practice: quarterly reviews with "what to start/stop/continue". |
| **Web dashboard for read-only overview** | Quick visual status of goals, tasks, habits without opening terminal or Obsidian | HIGH | Not yet built. Requirements specify this. Should be read-only, pulling from vault markdown. |
| **Dynamic Jira project configuration** | Add/remove Jira projects without code changes -- config-driven | MEDIUM | Not yet built. Critical for multi-project professionals. |
| **Contextual task filtering** | GTD contexts (@home, @office, @calls, @computer) surface relevant tasks based on where you are | LOW | Tags exist but no dedicated context-switching skill. Easy to add. |
| **Stale contact detection** | Alerts when important contacts have gone without interaction beyond a threshold | LOW | Already in `status` skill. Personal CRMs charge for this. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create maintenance burden, complexity, or misaligned incentives.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| **Mobile app** | "I need to capture on the go" | Native app development is a massive scope expansion for a personal system. Maintenance cost is disproportionate to value. | Obsidian mobile for vault access + Apple Reminders as quick capture inbox (sync to vault via skill). Already in Out of Scope. |
| **Real-time sync / live updates** | "I want changes reflected instantly everywhere" | Obsidian vault is local-first markdown. Real-time sync adds conflict resolution, networking complexity, and fragility. | iCloud/Syncthing for vault sync (already works). Web dashboard can poll on refresh. |
| **Auto-posting to social media** | "Generate and publish in one flow" | Platform APIs change constantly, auth tokens expire, scheduling logic is complex. Specialized tools (Buffer, Typefully) do this better. | Generate draft content, copy-paste to platform or use dedicated scheduling tool. Already in Out of Scope. |
| **Habit tracking with streaks** | "I want to track daily habits like meditation, reading" | Habit tracking is a deep product category (Habitica, Streaks, etc.). Building a mediocre version wastes effort. | Track gym via `train`/`sync-training` (already built). For other habits, use dedicated app or simple vault checkbox in daily notes. |
| **AI auto-categorization without confirmation** | "Just put things in the right place automatically" | Silent automation erodes trust. GTD requires conscious processing -- the clarify step is intentional. | AI suggests, user confirms. Current `process-inbox` skill does this correctly. Never auto-save. |
| **Email integration** | "Scan my email for tasks and action items" | Email is noisy, privacy-sensitive, and difficult to parse reliably. High effort, low signal-to-noise ratio. | Manually forward important emails to inbox capture. Or use Apple Mail rules to flag, then process. |
| **Time tracking / Pomodoro** | "I want to track how long tasks take" | Time tracking adds friction to every task. Pomodoro timers are commodity tools. | If needed, integrate Toggl/Clockify as a data source (like Jira). Don't build. |
| **Gamification (points, badges, levels)** | "Make productivity fun!" | Gamification shifts motivation from intrinsic (getting things done) to extrinsic (points). Short-term engagement, long-term burnout. | Use energy tracking in daily notes (already exists: red/yellow/green). Celebrate wins in weekly review. |
| **Multi-user / family sharing** | "My partner could use this too" | Exponential complexity: permissions, shared vs. private data, conflict resolution. This is a personal system. | Each person runs their own instance. Already in Out of Scope (no real-time collaboration). |
| **Calendar event creation** | "Create meetings from the CLI" | Calendar APIs are complex (recurring events, invites, timezone handling). Risk of creating duplicate/wrong events. | Read-only calendar sync is the right boundary. Create events in native calendar app. |
| **Natural language date parsing** | "Remind me next Tuesday at 3pm" | Building a reliable date parser is surprisingly hard. Edge cases with relative dates, timezones, locale-specific formats. | Use explicit ISO dates or week tags (#YYYY-W##). The weekly/daily structure handles scheduling. |

## Feature Dependencies

```
[Vault Structure / Organization]
    |
    +--required-by--> [Daily Note Generation]
    |                      |
    |                      +--requires--> [Calendar Integration]
    |
    +--required-by--> [Weekly Review]
    |                      |
    |                      +--requires--> [Daily Note Generation]
    |                      +--requires--> [Goal Setting]
    |
    +--required-by--> [Inbox Processing]
    |                      |
    |                      +--requires--> [Quick Capture]
    |
    +--required-by--> [Status Dashboard (CLI)]
                           |
                           +--requires--> [Goal Setting]
                           +--requires--> [People Notes / CRM]
                           +--requires--> [Task Centralization]

[Meeting Notes Capture]
    |
    +--enables--> [People Enrichment / CRM]
    |                 |
    |                 +--enables--> [Meeting Prep Briefing]
    |
    +--enables--> [Cross-Source Action Item Extraction]
                      |
                      +--feeds--> [Inbox Processing]

[Knowledge Capture]
    |
    +--enables--> [Content Generation Pipeline]
                      |
                      +--requires--> [Knowledge maturity tracking]

[Goal Setting]
    |
    +--enables--> [Goal-Aligned Daily Focus]
    +--enables--> [Quarterly Reflection Workflow]
    +--enables--> [Web Dashboard]
                      |
                      +--requires--> [Status Dashboard logic]
                      +--requires--> [Vault Structure]

[External Task Centralization (Jira)]
    |
    +--requires--> [Dynamic Jira Project Config]
    +--feeds--> [Daily Note] (via task queries)
    +--feeds--> [Meeting Prep] (via ticket context)
```

### Dependency Notes

- **Vault Structure is foundational:** Every other feature reads from the vault. Getting the folder structure, frontmatter conventions, and tagging scheme right is a prerequisite for everything.
- **Quick Capture requires Inbox Processing:** Capture without clarification creates a growing junk pile. These two must ship together.
- **Weekly Review requires Daily Notes:** The retrospective analyzes daily note data (completed tasks, energy, reflections). Without dailies, the review is hollow.
- **Meeting Prep requires People Notes:** The briefing aggregates from People notes. Without CRM data, prep is just a calendar entry.
- **Content Pipeline requires Knowledge Capture:** Content generation draws from accumulated knowledge entries. Without mature topics, content is generic.
- **Web Dashboard requires Status logic:** The dashboard is a visual layer on top of the same vault-reading logic that `status` CLI skill uses.

## MVP Definition

### Launch With (v1)

The GTD core that makes this a trusted external brain.

- [ ] **Vault structure redesign** -- Foundation for everything else. PARA-inspired organization with consistent frontmatter.
- [ ] **Quick capture + inbox processing** -- The GTD entry point. Already built, needs polish and vault integration.
- [ ] **Daily note generation** -- The daily driver. Already built, needs vault structure alignment.
- [ ] **Calendar sync** -- Already built. Essential for daily/weekly planning.
- [ ] **Weekly review** -- Already built. GTD's most important habit.
- [ ] **Backlog management with next actions** -- Core task list with context tags, project grouping, and stall detection.
- [ ] **Goal setting framework** -- Goals YAML with quarterly objectives, weight, status, and deadline.
- [ ] **Meeting notes sync (Granola)** -- Already built. Feeds CRM and action items.
- [ ] **People notes / Mini CRM** -- Already built via Granola enrichment. Foundation for prep and relationship tracking.
- [ ] **Status dashboard (CLI)** -- Already built. Quick overview of all dimensions.

### Add After Validation (v1.x)

Features to layer on once the GTD core is solid and daily usage is habitual.

- [ ] **Meeting prep briefing** -- Add when user consistently has People notes populated (after 2-4 weeks of Granola syncing).
- [ ] **Knowledge capture with maturity tracking** -- Already built. Add structured workflow once daily capture habit exists.
- [ ] **Content generation pipeline** -- Add when knowledge notes reach "ready" maturity. Already built, needs integration.
- [ ] **Jira centralization (dynamic config)** -- Add when vault structure supports multi-project task views.
- [ ] **Slack sync** -- Add when inbox processing is smooth. Avoids overwhelming the inbox early on.
- [ ] **Quarterly reflection workflow** -- Add after first full quarter of goal tracking data exists.
- [ ] **Contextual task filtering** -- Add when backlog has enough tasks tagged by context to make filtering useful.

### Future Consideration (v2+)

Features that need the full system running to justify.

- [ ] **Web dashboard** -- Defer until vault data is rich enough to visualize. High implementation cost (separate web app). Build once CLI workflow is validated.
- [ ] **Cross-source deduplication** -- Advanced AI matching across Jira + Slack + Granola + manual tasks. Defer until all sources are syncing.
- [ ] **Stale goal detection with nudges** -- "You haven't progressed on X in 3 weeks." Needs historical data to be meaningful.
- [ ] **Content calendar / scheduling view** -- Only valuable with a consistent content cadence. Defer until pipeline is proven.

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Vault structure redesign | HIGH | MEDIUM | P1 |
| Quick capture + inbox processing | HIGH | LOW (exists) | P1 |
| Daily note generation | HIGH | LOW (exists) | P1 |
| Calendar sync | HIGH | LOW (exists) | P1 |
| Weekly review | HIGH | LOW (exists) | P1 |
| Backlog / next actions management | HIGH | MEDIUM | P1 |
| Goal setting framework | HIGH | MEDIUM | P1 |
| Meeting notes sync (Granola) | HIGH | LOW (exists) | P1 |
| People notes / Mini CRM | MEDIUM | LOW (exists) | P1 |
| Status dashboard (CLI) | MEDIUM | LOW (exists) | P1 |
| Meeting prep briefing | MEDIUM | LOW (exists) | P2 |
| Knowledge capture | MEDIUM | LOW (exists) | P2 |
| Content generation pipeline | MEDIUM | LOW (exists) | P2 |
| Jira centralization (dynamic) | HIGH | MEDIUM | P2 |
| Slack sync | MEDIUM | LOW (exists) | P2 |
| Quarterly reflection | MEDIUM | MEDIUM | P2 |
| Contextual task filtering | LOW | LOW | P2 |
| Web dashboard | MEDIUM | HIGH | P3 |
| Cross-source deduplication | LOW | HIGH | P3 |
| Content calendar | LOW | MEDIUM | P3 |

**Priority key:**
- P1: Must have for launch -- the GTD core
- P2: Should have, add when GTD habits are established
- P3: Nice to have, future consideration after system is validated

## Competitor Feature Analysis

| Feature | Notion Life OS Templates | Obsibrain (Obsidian) | Todoist + GTD | This System (Life OS) |
|---------|--------------------------|----------------------|---------------|------------------------|
| Task management | Databases with views, filters, Kanban | Obsidian Tasks plugin + Dataview queries | Native task lists with priorities, labels, filters | Markdown backlog + Obsidian Tasks plugin + AI processing |
| GTD workflow | Manual template setup, no enforcement | P.A.R.A + GTD structure, manual processing | Good GTD support (inbox, projects, labels as contexts) | Full GTD with AI-powered inbox clarification |
| Goal tracking | Database with progress bars, linked to tasks | SMART projects with progress tracking | Basic via projects/sections | YAML-based goals with weight, quarterly review, daily alignment |
| CRM / People | Contact database, manually maintained | CRM via custom templates | None | Auto-enriched from meetings, with relationship context |
| Meeting notes | Manual entry or Notion AI summary | Manual entry | None | Source-agnostic sync (Granola today) with auto-enrichment |
| Knowledge base | Wiki-style pages, backlinks | Zettelkasten with backlinks | None | Thematic notes with maturity tracking, content pipeline |
| Content generation | None (separate tool) | None | None | Knowledge-to-content pipeline with topic maturity |
| Calendar integration | Notion Calendar (separate app) | Community plugins | Google Calendar 2-way sync | Read-only via icalBuddy, feeds daily/weekly notes |
| Dashboard | Built-in views and rollups | Command Center dashboard | Productivity view, Karma stats | CLI status + future web dashboard |
| AI assistance | Notion AI (summarize, write) | None native (plugin-dependent) | Todoist AI (task suggestions) | Deep LLM integration via Claude Code skills for every workflow |
| Data ownership | Notion cloud (proprietary) | Local markdown files | Todoist cloud | Local markdown files (Obsidian vault) |
| Extensibility | Notion API, integrations | Community plugins (1000+) | Todoist API, integrations | Claude Code skills (fully customizable) |
| Price | Free tier + $10/mo Pro + template cost ($20-50) | Free (Obsidian) + plugin ecosystem | Free tier + $5/mo Pro | Free (Obsidian + Claude Code) + API costs |

**Key competitive insight:** No existing product combines GTD + CRM + Knowledge Management + Content Pipeline with AI-powered processing. Notion Life OS templates are the closest in breadth but lack AI agent capabilities and are cloud-dependent. Obsibrain is the closest in architecture (Obsidian-based) but lacks the AI processing layer. This system's unique value is the conversational AI agent that actively processes, connects, and surfaces information -- it is not just a passive template.

## Sources

- [Notion Life OS Templates (notion.com)](https://www.notion.com/templates/life-os-all-in-one-productivity-system)
- [Obsibrain - Productivity System for Obsidian](https://www.obsibrain.com/)
- [Best GTD Apps Compared (Lovable)](https://lovable.dev/guides/best-gtd-application)
- [GTD Methodology (todoist.com)](https://www.todoist.com/productivity-methods/getting-things-done)
- [Personal CRM Software Comparison (monday.com)](https://monday.com/blog/crm-and-sales/personal-crm-software/)
- [Top Personal CRM Systems 2025 (Clarify)](https://www.clarify.ai/blog/top-personal-crm-systems-to-boost-your-productivity-in-2025)
- [Personal Productivity Systems 2026 (Sentari)](https://withsentari.com/personal-productivity-systems-2026/)
- [Life OS Dashboard](https://lifeosdashboard.com/)
- [Obsidian GTD Setup (Face Dragons)](https://facedragons.com/productivity/set-up-gtd-in-obsidian/)
- [OKR Guide (Todoist)](https://www.todoist.com/productivity-methods/okrs-objectives-key-results)
- [Obsidian + Claude PKM (GitHub)](https://github.com/ballred/obsidian-claude-pkm)
- [Second Brain GTD with Claude (GitHub)](https://github.com/sean-esk/second-brain-gtd)
- [Obsidian Forum: All-in-one GTD Template](https://forum.obsidian.md/t/i-created-an-all-in-one-productivity-template-for-obsidian-task-management-gtd-para-goal-tracking-reviews-and-more/85792)

---
*Feature research for: Personal Productivity / Life OS*
*Researched: 2026-03-28*
