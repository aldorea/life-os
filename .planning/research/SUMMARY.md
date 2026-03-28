# Project Research Summary

**Project:** Life OS — Personal Productivity System
**Domain:** GTD + Knowledge Management + CRM + Content Pipeline (CLI-first, Obsidian-backed)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Executive Summary

Life OS is a CLI-first personal productivity system where an AI agent (Claude Code) operates directly on an Obsidian markdown vault. The architecture has three distinct layers: a CLI layer (Claude Code skills — already functional), a vault layer (Obsidian markdown as the single source of truth — already exists), and a web dashboard layer (read-only status view — to be built). The critical architectural insight from research is that the CLI and vault layers require zero new dependencies — all stack decisions concern only the web dashboard and thin shared utilities. No database is needed; the vault IS the database, and adding one would create sync drift with no benefit for a single-user personal system.

The recommended approach builds a complete GTD loop first (capture → inbox → process → backlog → daily note → weekly review) and freezes it before adding any integration. Research is unambiguous that partial GTD implementations are worse than no GTD — they generate anxiety without relieving it. The 13 existing skills partially cover this loop but are in a "half-built" state that needs hardening before layering external integrations (Jira, Granola, Slack, Calendar). For the web dashboard (a v2+ concern), Next.js 15 with Server Components is the clear choice, reading vault files directly via `fs` without a database layer.

The primary risk is System-as-Project Syndrome: an engineer's tendency to keep refining the productivity system instead of using it. Research identifies this as the #1 failure mode for GTD + Obsidian implementations, and the current project already shows warning signs (15 partially-working skills, pending vault restructure, multiple partially-integrated services). The mitigation is non-negotiable: ship the complete GTD loop, use it daily for two weeks, then proceed. Every subsequent phase must be gated by active daily usage of the prior phase.

## Key Findings

### Recommended Stack

The stack is deliberately minimal because the vault is the data store. No database, no ORM, no standalone backend server. The only non-trivial stack decision is the web dashboard, where Next.js 15 + React 19 + Server Components is the clear winner — it can read vault files on the server without a client bundle, has the largest component ecosystem for dashboards (shadcn/ui, Tremor), and deploys to Vercel with zero config.

For vault manipulation in any future automation scripts, gray-matter + yaml + globby cover frontmatter parsing, YAML config files, and file discovery. Biome replaces ESLint + Prettier as a single binary. Node.js 22 LTS supports TypeScript natively via `--strip-types`.

**Core technologies:**
- **Next.js 15.5 + React 19:** Web dashboard framework — Server Components read vault files server-side; largest dashboard component ecosystem
- **TypeScript 5.5+:** Non-negotiable given multiple data shapes (tasks, goals, contacts, calendar events)
- **Tailwind CSS 4.2 + shadcn/ui:** Styling and UI components — CSS-native config, pairs with shadcn copy-paste components
- **Node.js 22 LTS:** Runtime with native TypeScript strip-types support
- **gray-matter + yaml + globby:** Vault manipulation — frontmatter parsing, YAML config, file discovery
- **TanStack Query 5.95:** Client-side data fetching for dashboard — smart caching, pairs with Server Components
- **Zod 4.3:** Schema validation for config.yaml and API responses
- **Biome 2.3:** Lint + format in one binary — 50x faster than ESLint + Prettier
- **Vitest 4.1:** Testing for dashboard components and utilities
- **Vercel:** Zero-config Next.js deployment for personal dashboard

**What to avoid:** Prisma/Drizzle (no database), Moment.js (deprecated), ts-node (use tsx or native Node.js), Redux/Zustand (TanStack Query covers server state), styled-components (Tailwind is compile-time), Electron/Tauri (over-engineered for a personal dashboard).

### Expected Features

Research confirms the system covers a unique combination of GTD + CRM + Knowledge Management + Content Pipeline that no single existing product provides. The competitive differentiation is the AI agent layer — not just a passive template, but a conversational interface that actively processes, connects, and surfaces information.

**Must have — table stakes (GTD core):**
- Quick capture to inbox — frictionless, under 5 seconds, any context
- Inbox processing / clarify — AI-reformulated actionable tasks with tagging
- Next actions list with context filtering — core GTD "what do I do now?" view
- Projects list with next action tracking and stall detection
- Someday/Maybe list for ideas not currently committed to
- Weekly review — GTD's most critical habit; without it the system decays in 2 weeks
- Daily note generation — agenda, focus, top tasks in one view
- Calendar integration — see meetings alongside tasks for realistic planning
- Goal setting with quarterly tracking and daily alignment
- Status dashboard (CLI) — quick "how am I doing?" across all dimensions

**Should have — differentiators:**
- AI-powered inbox clarification (existing `process-inbox`) — the killer feature vs. manual GTD apps
- Meeting prep briefing aggregating people context, meeting history, and pending tickets
- Knowledge-to-content pipeline with maturity tracking (seed → growing → ready → draft)
- Source-agnostic meeting sync (abstract from Granola; replaceable connector pattern)
- Dynamic Jira project configuration via `connectors.yaml`
- Quarterly reflection workflow closing the OKR feedback loop
- People enrichment from meetings — CRM that builds itself

**Defer to v2+:**
- Web dashboard — high implementation cost; build once CLI workflow is validated and vault data is rich
- Cross-source deduplication (Jira + Slack + Granola + manual) — needs all sources active first
- Content calendar / scheduling view — only valuable with consistent content cadence

**Anti-features to avoid:** Mobile app (scope explosion), real-time sync (conflict resolution complexity), auto-posting to social media (fragile APIs), habit tracking (deep product category — use dedicated app), AI auto-categorization without user confirmation (erodes GTD trust).

### Architecture Approach

The system is a five-layer architecture: Interaction (Claude Code skills, Obsidian, Web Dashboard) → Orchestration (skills, config engine, sync scheduler) → Domain Logic (GTD engine, goals tracker, CRM, content pipeline, health tracker) → Connector (Jira MCP, icalBuddy, Granola MCP, Slack MCP) → Persistence (Obsidian vault markdown files). All state is file-based — no runtime state, no database. Every piece of data has a canonical markdown representation with YAML frontmatter.

**Major components:**
1. **Claude Code Skills (13 existing)** — primary action interface; markdown prompt files dispatched by LLM reasoning; zero runtime code
2. **Obsidian Vault** — persistent storage, visualization, navigation, Dataview/Tasks queries; the database
3. **Config Engine (config.yaml)** — single source of truth for all skill behavior; every skill reads it at Step 0
4. **Connector Skills (sync-*)** — the ONLY components that talk to external APIs; write normalized markdown to vault; consumer skills never call external APIs directly
5. **Web Dashboard (to build)** — read-only Next.js app reading vault files via Server Components; NEVER writes to vault

**Key patterns:** Vault as Database, Connector-Consumer Separation, Configuration-Driven Skills, Skill Composition (skill-calls-skill), Frontmatter as Structured Metadata.

**Critical boundaries:** Web dashboard is read-only — never writes to vault. Consumer skills (`today`, `week`, `prep`) never call external APIs. Connector skills (`sync-*`) write to vault; consumers read from vault. Config path is the single point of customization.

### Critical Pitfalls

1. **System-as-Project Syndrome** — avoid by enforcing a system freeze after each phase; the ratio of time spent ON the system vs. USING it must stay below 20%; ship the GTD core and use it daily for 2 weeks before any additions.

2. **Skipping the Weekly Review (GTD Death Spiral)** — avoid by making the `week` skill an interactive review FACILITATOR (not just a note generator); add review health metrics; surface "last review was N days ago" prominently in daily note when >7 days.

3. **Integration Fragility** — avoid by treating every integration as independently degradable; sync skills write to cache files with `last_sync` timestamps; consumer skills read cache only, never block on live API calls; always timeout after N seconds and continue with stale data notice.

4. **Capture Friction Kills Adoption** — avoid by splitting capture into two modes: quick capture (append raw text to inbox, under 3 seconds, default) vs. rich capture (current `capture` skill with classification, for when there's time); the inbox is the buffer, not a processing step.

5. **LLM Nondeterminism in Vault Mutations** — avoid by creating a strict vault schema document (exact frontmatter fields per note type, exact tag vocabulary, ISO date formats); pin formats in SKILL.md with explicit examples; add a `lint-vault` skill for consistency checks.

6. **Partial GTD Implementation** — avoid by shipping the complete GTD loop in Phase 1 as a unit: capture → process → backlog with contexts → daily note → weekly review. A primitive complete loop beats a sophisticated partial one.

## Implications for Roadmap

Research is aligned: four phases, each independently valuable, each gated by active daily use of the prior phase.

### Phase 1: GTD Core (Complete Loop)

**Rationale:** Every other feature depends on a functioning GTD loop. Research is unambiguous that partial GTD is worse than no GTD. The 13 existing skills are partially working — Phase 1 hardens them into a complete, reliable daily workflow before any integrations are added. This directly addresses the top-ranked pitfall (System-as-Project Syndrome) by forcing a "use it" gate before Phase 2.

**Delivers:** A fully functional daily productivity loop with zero external dependencies. User can capture thoughts, process them into actionable tasks with context tags, generate a goal-aligned daily note, and complete a weekly review with retrospective.

**Addresses:**
- Quick capture with two modes (quick < 3s, rich with classification)
- Inbox processing with AI clarification, project assignment, context tagging
- Backlog management: next actions, projects with stall detection, someday/maybe list
- Daily note generation (goal-aligned focus, agenda, top 3 tasks, alerts)
- Weekly review as an interactive facilitator (not just a note generator) with review health tracking
- Goal setting framework (goals.yaml with quarterly objectives, weight, deadline, status)
- Vault schema document establishing exact frontmatter fields, tag vocabulary, date formats
- Config.yaml hardening — all skills use config paths, zero hardcoded values

**Avoids:** Partial GTD (ship the full 5-step loop), LLM nondeterminism (vault schema first), capture friction (quick capture path), system-as-project syndrome (2-week usage gate before Phase 2).

**Research flag:** Standard patterns — no phase research needed. GTD loop is well-documented.

### Phase 2: External Integrations (Connectors)

**Rationale:** Connectors bring external data into a now-stable vault structure. Adding them before Phase 1 is stable would create integration failures on an unstable foundation. Each connector must be independently deployable and independently degradable — a broken Jira sync must never block daily note generation.

**Delivers:** External data flowing into the vault without manual entry. Jira tickets, meeting notes, calendar events, and Slack context all arrive as vault-native markdown with clean frontmatter.

**Addresses:**
- Calendar sync (icalBuddy with timeout wrapper — stale cache is acceptable)
- Granola meeting sync with People enrichment (dedup registry pattern)
- Jira sync with dynamic project config via `connectors.yaml` and caching (avoid rate limits across two instances)
- Slack sync with channel allowlist (decisions and action items only, no DMs)
- Sync freshness tracking: `last_sync` per integration surfaced in daily note

**Avoids:** Integration fragility (explicit timeout + cache + failure mode per connector), Jira rate limiting (field filtering + cache reads in consumer skills), sync-all-on-startup (syncs are independent background operations).

**Research flag:** Needs phase research for Jira multi-instance rate limiting and icalBuddy timeout behavior. Apple Reminders has no web API — requires AppleScript or Swift CLI; document as macOS-only.

### Phase 3: Compound Intelligence

**Rationale:** These features require accumulated data from Phase 2 to be meaningful. Meeting prep needs People notes populated (2-4 weeks of Granola syncing). Content generation needs knowledge notes at "ready" maturity. Quarterly reflection needs a full quarter of goal tracking. Building these in Phase 3 means the data is there when the features launch.

**Delivers:** The system starts compounding: each meeting makes the next one more informed, accumulated knowledge becomes publishable content, goals have historical data for meaningful retrospectives.

**Addresses:**
- Meeting prep briefing (People + meeting history + Jira tickets → pre-meeting brief)
- Knowledge capture with maturity tracking (seed → growing → ready) and content generation
- Quarterly reflection workflow (OKR retrospective: start/stop/continue)
- Contextual task filtering (@home, @office, @calls, @computer)
- Stale contact detection surfaced in `status` skill
- CRM enrichment — verify deduplication across name format variants
- Backlog split if single file exceeds 200+ tasks (work vs. personal)

**Avoids:** Vault over-engineering (restructure only after 4+ weeks of active use with Phase 2 structure; migrate incrementally with scripted migration, not manual), Dataview performance (all queries scoped to specific folders).

**Research flag:** Standard patterns for most features. Content pipeline maturity model may need design research.

### Phase 4: Web Dashboard

**Rationale:** Dashboard is a presentation layer that requires everything else to be stable. It reads vault structure established in Phase 1, external data populated in Phase 2, and compound data from Phase 3. Building it earlier would require rebuilding as the vault structure evolves. The high implementation cost (separate Next.js app) is justified only when vault data is rich enough to visualize.

**Delivers:** A browser-based read-only overview of goals, tasks, calendar, contacts, and sync freshness — accessible without opening the terminal or Obsidian.

**Addresses:**
- Next.js 15 + Server Components reading vault files via `fs` + gray-matter
- shadcn/ui for tables, cards, progress bars; Tremor for goal progress charts
- TanStack Query for client-side refresh (page-load or manual refresh — no real-time polling)
- Zod validation for vault data shapes read by the dashboard
- Sync freshness indicators per integration (never show stale data without labeling it)
- Vercel deployment with authentication (NextAuth.js or Clerk) if remote access needed

**Avoids:** Real-time vault polling (file watching is unreliable; manual refresh is sufficient for personal system), dashboard writing to vault (read-only boundary is absolute).

**Research flag:** Needs phase research for Tremor compatibility with Tailwind CSS v4 (migration status unclear), authentication approach if remote access required.

### Phase Ordering Rationale

- **Foundation before integrations:** Vault schema and config.yaml must be stable before connectors write to the vault; changing frontmatter conventions after connectors run breaks all historical data.
- **GTD loop complete before enhancements:** Research is explicit — partial GTD creates anxiety rather than reducing it. The loop must ship as a unit.
- **Connectors before compound features:** Meeting prep requires People notes populated by Granola sync. Content generation requires knowledge notes captured via the `capture` skill. These features are empty without prior phases.
- **Dashboard deferred:** High implementation cost + presentation-only value; justified only with rich vault data. Matches FEATURES.md placement as v2+.
- **Usage gate between phases:** After each phase, 2+ weeks of daily use before starting the next. Enforced by the pitfall research — system-as-project syndrome is the primary failure mode.

### Research Flags

**Phases needing deeper research during planning:**
- **Phase 2 (Integrations):** Jira multi-instance rate limiting behavior; icalBuddy subprocess timeout patterns; Granola MCP ID stability guarantee; Apple Reminders native access approach (AppleScript vs. Swift CLI).
- **Phase 4 (Dashboard):** Tremor + Tailwind CSS v4 compatibility status; authentication approach for remote vault access (NextAuth.js vs. Clerk for a personal system).

**Phases with standard patterns (can skip research-phase):**
- **Phase 1 (GTD Core):** GTD methodology and Obsidian vault patterns are extensively documented. Claude Code skill architecture is established.
- **Phase 3 (Compound Intelligence):** Features build on established vault patterns from Phase 1 and connector outputs from Phase 2. Content maturity model is the only design decision without a clear prior art.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All technologies verified against official release notes and docs. Version compatibility table is explicit. No speculative recommendations. |
| Features | HIGH | Competitor analysis is thorough (Notion Life OS, Obsibrain, Todoist GTD). Feature boundaries and dependency graph are well-reasoned. MVP definition is clear. |
| Architecture | HIGH | Patterns are validated against multiple reference implementations (obsidian-claude-pkm, GTD vault templates, Claude Skills deep dive). Anti-patterns are explicitly documented. |
| Pitfalls | HIGH | Sources are primary (GTD failure research, Jira rate limit docs, Dataview performance forums, Apple EventKit docs). Pitfalls are specific to this exact system type, not generic. |

**Overall confidence:** HIGH

### Gaps to Address

- **Tremor + Tailwind CSS v4 compatibility:** STACK.md notes "check Tremor docs for Tailwind v4 migration status" with MEDIUM confidence. Before Phase 4 planning, verify Tremor is production-ready with Tailwind v4. Fallback: use Recharts directly with shadcn/ui chart components.

- **Apple Reminders integration path:** No web API exists; requires AppleScript or Swift CLI for EventKit access. Document as macOS-only and plan accordingly in Phase 2. No blocking issue — just needs implementation choice confirmed.

- **Vault structure freeze decision:** PITFALLS.md recommends freezing the current vault structure and only restructuring after 4+ weeks of active use. The existing vault has a partially-designed structure. Phase 1 must decide: use current structure as-is (faster to ship, messier) or do the minimal required restructuring before any skill hardening (slower, cleaner foundation). This is a planning decision, not a research gap.

- **Quick capture path beyond CLI:** Research identifies that capture must work in under 5 seconds from any context (meeting, browser, phone). The CLI path is covered. Non-CLI paths (Obsidian quick-add, Apple Shortcuts, hotkey → inbox.md append) need design in Phase 1 planning.

- **Git commit-before-mutate pattern:** PITFALLS.md recommends committing the vault before each skill run that mutates it, as the only rollback mechanism. This is an architectural decision (does the system automate this commit, or rely on the user?) that needs resolution in Phase 1.

## Sources

### Primary (HIGH confidence)

- [Next.js 15.5 release](https://nextjs.org/blog/next-15-5) — version and Server Components features
- [Tailwind CSS v4.2 release](https://tailwindcss.com/blog/tailwindcss-v4) — CSS-native config confirmed
- [Zod v4 release notes](https://zod.dev/v4) — performance improvements verified
- [Node.js TypeScript docs](https://nodejs.org/en/learn/typescript/run-natively) — native --strip-types support
- [Jira Cloud Rate Limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/) — rate limit behavior for multi-instance sync
- [Apple EventKit Documentation](https://developer.apple.com/documentation/eventkit) — Apple Reminders API limitations confirmed
- [Biome migration guide](https://biomejs.dev/guides/migrate-eslint-prettier/) — rule coverage and speed claims
- [Vitest 4.0 announcement](https://vitest.dev/blog/vitest-4) — Browser Mode stable
- [obsidian-claude-pkm (GitHub)](https://github.com/ballred/obsidian-claude-pkm) — reference architecture for Claude-powered Obsidian systems
- [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/) — meta-tool architecture pattern

### Secondary (MEDIUM confidence)

- [Obsibrain](https://www.obsibrain.com/) — closest Obsidian-based competitor; confirms PARA + GTD viability
- [GTD failure patterns (FacileThings)](https://facilethings.com/blog/en/why-gtd-fails) — weekly review skip as primary failure mode
- [PKM at Scale: 8,000 Notes](https://www.dsebastien.net/personal-knowledge-management-at-scale-analyzing-8-000-notes-and-64-000-links/) — Dataview performance at scale
- [Dataview Performance Issues (Obsidian Forum)](https://forum.obsidian.md/t/dataview-very-slow-performance/52592) — folder-scoped query recommendation
- [Tremor dashboard components](https://www.tremor.so/) — Tailwind v4 compatibility needs verification
- [Obsidian GTD Vault Template (DeepWiki)](https://deepwiki.com/adiehl96/obsidian-vault-template-for-gtd) — numbered folder pattern validated

### Tertiary (LOW confidence)

- [Temporal API Stage 4](https://www.wearedevelopers.com/en/magazine/544/the-temporal-api-how-javascript-dates-might-actually-be-getting-fixed-544) — noted as future option when Node.js support catches up; use date-fns for now
- [AI Operating System for Personal Productivity (motyl.dev)](https://motyl.dev/news/ai-operating-system-personal-productivity-2026) — confirms AI-first productivity system patterns but source is newer and less established

---
*Research completed: 2026-03-28*
*Ready for roadmap: yes*
