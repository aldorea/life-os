# Pitfalls Research

**Domain:** Personal Productivity Life OS (GTD + Obsidian + CLI Skills + Integrations)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Critical Pitfalls

### Pitfall 1: System-as-Project Syndrome (Building the Productivity System Instead of Being Productive)

**What goes wrong:**
The Life OS becomes a perpetual engineering project. You spend weeks designing the perfect vault structure, refining skill prompts, adding integrations, and tweaking templates -- but never settle into a steady state where the system fades into the background and actual work gets done. The ~15 existing skills are already at risk of this: they're partially working, meaning the system is in a half-built state that demands constant attention.

**Why it happens:**
Engineers love building tools. The dopamine hit of "improving the system" feels productive but isn't. Every new integration (Granola, Slack, Jira, Calendar, Apple Reminders) adds complexity that needs maintenance. The system grows faster than the habits that make it useful.

**How to avoid:**
- Define a hard "system freeze" after each phase. No structural changes for 2+ weeks while you actually use it.
- Track a ratio: time spent ON the system vs. time spent USING the system. If ON > 20%, stop building.
- Ship the GTD core (capture, process, review) first and use it daily for 2 weeks before adding any integration.
- Each skill should solve a problem you have TODAY, not a problem you might have later.

**Warning signs:**
- You have more open tasks about the system than about actual work.
- Skills exist but you don't run them daily.
- You keep redesigning vault structure without having used the current one for a full week.
- Multiple integrations are "partially working" (which is the current state).

**Phase to address:**
Phase 1 (GTD Core). Ship the minimum GTD loop (capture -> inbox -> process -> backlog -> daily/weekly review) and freeze. All integrations come later.

---

### Pitfall 2: Skipping the Weekly Review (The GTD Death Spiral)

**What goes wrong:**
The weekly review is the single most critical habit in GTD. Without it, your inbox grows stale, your backlog becomes a graveyard of outdated tasks, contexts lose meaning, and you stop trusting the system. Once trust breaks, you fall back to keeping things in your head, and the entire Life OS becomes shelfware. This is the #1 reason GTD implementations fail -- documented across every GTD community.

**Why it happens:**
The weekly review feels like overhead. It's not urgent (no meeting, no deadline), so it gets skipped. The `week` skill generates a nice note, but generating the note is not the same as doing the review. The review requires human judgment: renegotiating commitments, moving items to someday/maybe, pruning dead projects.

**How to avoid:**
- The `week` skill must be a review FACILITATOR, not just a note generator. It should walk the user through each GTD review step interactively (not just dump a summary).
- Build a "review health" metric: days since last completed review. Surface this prominently in the daily note when > 7 days.
- Make the review output lightweight -- the barrier to completion must be low. A 10-minute review done is better than a 45-minute review skipped.
- The weekly note should flag: stale inbox items (> 3 days), backlog tasks with no activity (> 2 weeks), projects with no next action defined.

**Warning signs:**
- The `week` skill is run but the "Reflexion" section stays empty.
- Inbox item count grows week over week.
- Backlog has tasks older than 4 weeks with no updates.
- Daily notes stop being generated (the system is being abandoned).

**Phase to address:**
Phase 1 (GTD Core). The weekly review loop must be built and tested as part of the core GTD implementation, not bolted on later.

---

### Pitfall 3: Integration Fragility (External APIs as Single Points of Failure)

**What goes wrong:**
The system depends on 5+ external services (Jira x2, Granola, Slack, Apple Calendar, Apple Reminders). Each integration is a fragile coupling. Jira Cloud enforces points-based rate limiting, per-second limits, and per-issue write limits. Granola or Slack MCPs can be unavailable. Apple Reminders has no web API -- it requires native EventKit access. When any integration breaks, the entire morning routine (`today` skill) degrades. Worse: if sync fails silently, you miss tasks and lose trust in the system.

**Why it happens:**
The design currently chains skills: `today` calls `sync-calendar`, which calls `icalBuddy`. If `icalBuddy` hangs (a known issue with CLI subprocesses), the entire daily note generation stalls. Jira MCP across two instances doubles the failure surface. Each integration is built by a different MCP server with different reliability characteristics.

**How to avoid:**
- Every integration MUST have a circuit breaker: timeout after N seconds, use cached data, continue with degradation notice. The existing skills have "graceful degradation" tables -- good. Enforce this rigorously.
- Separate "sync" from "generate". Run syncs as independent background operations that write to cache files. Generators (today, week) only read cache files, never call external APIs directly. This is partially the current design -- make it absolute.
- Store a `last_sync` timestamp per integration in a single status file. The daily note should show sync freshness: "Jira: synced 2h ago", "Calendar: synced 5min ago", "Granola: STALE (3 days)".
- Never block the user's workflow on a sync. A daily note with stale data is infinitely better than no daily note.

**Warning signs:**
- Morning routine takes > 60 seconds to complete.
- You find yourself re-running skills because "something didn't sync."
- A service outage (Jira maintenance, Granola downtime) makes the system unusable.
- Sync errors are swallowed silently -- you don't know data is stale.

**Phase to address:**
Phase 2 (Integrations). Build each integration with explicit timeout, caching, and freshness tracking. Test failure modes (disconnect network, kill MCP server) before considering an integration "done."

---

### Pitfall 4: Capture Friction Kills Adoption

**What goes wrong:**
GTD lives or dies on ubiquitous capture. If capturing a thought requires opening a terminal, running a Claude Code skill, waiting for LLM processing, and confirming the save -- that's 30+ seconds of friction. By then, the thought is gone or you've decided "I'll remember it." The current `capture` skill fetches URLs, classifies topics, checks duplicates, updates frontmatter -- all valuable but all adding latency to what should be a 2-second action.

**Why it happens:**
The capture skill is over-engineered for the capture moment. Classification, deduplication, and knowledge-base enrichment are processing activities (GTD's "clarify" step), not capture activities. GTD explicitly separates capture from processing for this reason.

**How to avoid:**
- Split capture into two modes: (1) Quick capture: append raw text to inbox, zero processing, < 3 seconds. (2) Rich capture: the current skill with classification, for when you have time. Quick capture should be the default.
- The inbox is the buffer. Everything goes there raw. Processing happens later (via `process-inbox`).
- Consider non-CLI capture paths: a simple hotkey that appends to inbox.md, Obsidian's built-in quick-add, or Apple Shortcuts integration.
- Measure: if you're capturing < 3 items/day, the friction is too high.

**Warning signs:**
- You capture things mentally ("I'll add it later") instead of into the system.
- The inbox is mostly empty -- not because you processed everything, but because you're not capturing.
- Capture only happens during dedicated "system time," never in the flow of work.

**Phase to address:**
Phase 1 (GTD Core). Quick capture must be frictionless from day one. Rich capture can come in Phase 2.

---

### Pitfall 5: Vault Structure Over-Engineering and Premature Restructuring

**What goes wrong:**
The project explicitly includes "vault redesign: reorganize existing daily notes, meeting notes, and knowledge base." This is a trap. Restructuring the vault before the workflows are stable means you'll restructure again when you discover the structure doesn't match how you actually use the system. Obsidian's community is full of people who spent weeks on folder hierarchies, templates, and frontmatter schemas only to abandon them when their workflows evolved.

**Why it happens:**
A clean vault feels productive. Engineers especially love clean architecture. But vault structure should emerge from usage patterns, not precede them. The config.yaml already defines `config.structure.*` paths -- changing these breaks every skill simultaneously.

**How to avoid:**
- Freeze the vault structure for Phase 1. Use the existing structure, warts and all. Only restructure after 4+ weeks of active daily use with the new GTD workflows.
- When restructuring (Phase 3+), migrate incrementally: move one folder at a time, update config.yaml, verify all skills still work, then move the next.
- Prefer flat structures over deep nesting. `meetings/2026-03-28-standup.md` is better than `meetings/2026/03/28/standup.md`.
- Wikilinks make folder structure less important -- Obsidian finds notes by name, not path. Don't over-invest in folder organization.

**Warning signs:**
- You've reorganized the vault more than once without having used the new structure for a full month.
- Skills break after vault changes because paths in config.yaml are stale.
- You spend time on folder naming conventions instead of on your actual work.

**Phase to address:**
Phase 3 or later (Vault Optimization). Only after GTD core and integrations are stable and actively used.

---

### Pitfall 6: Partial GTD Implementation (The Worst of Both Worlds)

**What goes wrong:**
You implement capture and daily notes but skip processing, contexts, and weekly review. Or you build all the skills but only use `today` and `capture`. Partial GTD is worse than no GTD: your inbox fills up but never gets processed, creating anxiety. You have a backlog but no next actions defined, so you still decide what to do from your head. The system becomes a guilt-generating machine instead of a stress-reducing one.

**Why it happens:**
GTD has 5 interconnected steps (capture, clarify, organize, reflect, engage). The tempting approach is to build skills for each step incrementally across phases. But if you ship capture in Phase 1 and processing in Phase 3, you have 2 phases of a broken system.

**How to avoid:**
- Phase 1 must include the COMPLETE GTD loop, even if primitive. A text-based inbox + manual processing + simple backlog + weekly review checklist is better than an automated capture + beautiful daily note + no processing.
- The minimum viable GTD is: (1) capture to inbox, (2) process inbox to backlog with contexts/next actions, (3) daily note showing next actions, (4) weekly review that audits the whole system. All four must ship together.
- Each subsequent phase enhances the loop (better sync, richer templates, more integrations) but never breaks it.

**Warning signs:**
- Inbox grows but `process-inbox` is never run.
- Backlog exists but has no `#next` tags -- everything is equally prioritized (meaning nothing is).
- You have "projects" in GTD terms (multi-step outcomes) with no defined next action.
- "Someday/maybe" list doesn't exist -- everything is either active or forgotten.

**Phase to address:**
Phase 1 (GTD Core). Non-negotiable: the full loop ships as a unit.

---

### Pitfall 7: LLM Nondeterminism in Vault Mutations

**What goes wrong:**
Every skill runs through Claude Code, meaning an LLM interprets the SKILL.md instructions and generates file mutations. LLMs are nondeterministic: the same skill invoked twice may produce different frontmatter schemas, different wikilink formats, different section headings, or different tag conventions. Over time, the vault accumulates inconsistencies that break Dataview queries, Tasks plugin queries, and cross-referencing.

**Why it happens:**
The skills are prompt-based, not code-based. The SKILL.md files describe intent ("create a meeting note with this format") but the LLM has latitude in interpretation. Frontmatter fields might be `last_interaction` in one note and `lastInteraction` in another. Tags might be `#work` or `#trabajo`. Dates might be ISO or localized.

**How to avoid:**
- Define a strict vault schema document: exact frontmatter fields per note type, exact tag vocabulary, exact date formats. Reference it from every skill.
- Use validation: after any skill writes to the vault, spot-check the output against the schema. Consider a `lint-vault` skill that checks consistency.
- Pin critical formats in SKILL.md with explicit examples and "NEVER deviate from this format" instructions.
- For critical writes (backlog tasks, frontmatter updates), prefer deterministic operations (find-and-replace) over generative ones (rewrite the section).

**Warning signs:**
- Dataview queries return incomplete results (some notes don't match because of format variations).
- The Tasks plugin shows duplicate or missing tasks.
- Wikilinks are broken because person names are formatted differently across notes.
- You find yourself manually fixing notes after skills run.

**Phase to address:**
Phase 1 (GTD Core). Establish the schema before any skill writes to the vault at scale. Enforce it in every subsequent phase.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoding Jira project keys in skills | Quick to build | Every new project requires code change | Never -- config.yaml already supports dynamic projects, enforce it |
| Storing all tasks in a single backlog.md | Simple to query | File grows unwieldy (> 500 tasks), merge conflicts with concurrent writes | MVP only -- split by context/project by Phase 3 |
| Using `icalBuddy` CLI for calendar | Works on macOS, no auth needed | macOS-only, subprocess hangs, no Linux/CI support | Acceptable for personal use if timeout is enforced |
| Embedding meeting summaries inline in People notes | Quick enrichment | People notes grow huge, hard to navigate | Never -- current Dataview approach (link, don't embed) is correct |
| Spanish-only vault content | Natural for user | Limits content reuse for English blog/LinkedIn | Acceptable -- content skill already handles language switching |
| No backup/versioning beyond git | Simple | Accidental vault corruption from LLM writes has no rollback | Phase 1 -- git commit before each skill run that mutates vault |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Jira (x2 instances) | Polling both instances on every `today` invocation, hitting rate limits | Sync to cache files on a schedule or on-demand; `today` reads cache only. Use field filtering to request only needed data. |
| Granola MCP | Assuming meeting IDs are stable across sessions | Use the `granola-processed.md` dedup registry (already designed). Verify ID stability. |
| Apple Calendar (icalBuddy) | No timeout on CLI subprocess -- hangs indefinitely | Wrap with `timeout 10s icalBuddy ...`. If timeout, use cached calendar data. |
| Slack MCP | Scanning all channels, extracting too much noise | Strict channel allowlist in config. Aggressive filtering (decisions + action items only). The current design is good here. |
| Apple Reminders | Assuming web API exists | There is no web API. EventKit requires native macOS access. AppleScript or Swift CLI is the only path. Plan for macOS-only limitation. |
| Obsidian Dataview | Writing queries that scan entire vault | Scope queries with `FROM "folder"` and use frontmatter filters. Avoid `TASK FROM ""` (scans everything). |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Dataview queries across entire vault | Daily note takes 3+ seconds to render in Obsidian | Scope all Dataview/Tasks queries to specific folders. Use frontmatter-based filtering. | ~1500+ notes, especially on mobile |
| Single monolithic backlog.md | `process-inbox` and `today` both read/write the same file; LLM reads entire file on every invocation | Split backlog by context (work/personal) or archive completed tasks monthly | ~300+ tasks in single file |
| LLM reading too many files per skill | Morning routine reads all Jira notes, all People notes, all daily notes | Each skill should read the MINIMUM needed. Use frontmatter indexes or summary files instead of scanning directories. | ~50+ files in any scanned directory |
| Sync-all-on-startup pattern | `today` triggers `sync-calendar` which is fine, but adding `sync-jira`, `sync-slack`, `sync-granola` to startup creates a 2+ minute boot | Syncs should be independent, cached, and stale-tolerant. Morning routine reads caches, does not trigger all syncs. | 3+ integrations active |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Storing API tokens in config.yaml inside the vault | Tokens synced to git, exposed in Obsidian | config.yaml should reference environment variables or a separate `.env` file outside the vault. `.gitignore` it. |
| Jira MCP server with write access | Accidental ticket modifications from LLM hallucination | Use read-only Jira tokens where possible. Gate write operations behind explicit user confirmation. |
| Slack MCP reading DMs or private channels | Private conversations captured in plaintext vault | Config should whitelist ONLY specific public channels. Never scan DMs. |
| Vault on cloud sync (iCloud/Dropbox) without encryption | Personal CRM data, meeting notes, task details exposed to cloud provider | Use Obsidian Sync (encrypted) or self-hosted sync. Or accept the risk for a personal system. |
| LLM sending vault content to API | Knowledge base and CRM data sent to Claude API on every skill invocation | Accept this for Claude Code (required for functionality). Be aware of what's in the vault -- no passwords, no secrets in notes. |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Skills require exact syntax (`/today`, `/capture url`) | User forgets commands, falls back to manual note-taking | Skills already use flexible trigger descriptions (good). Ensure the `description` field in SKILL.md covers all natural language variants. |
| Skills produce verbose output | User skims or ignores output, misses important alerts | Keep summaries to 3-5 lines. Use a consistent format: action taken + alerts only. Save detail for when asked. |
| No "undo" for vault mutations | LLM writes incorrect data, user must manually fix | Git commit before each mutation. Or at minimum, show a diff before applying changes. `process-inbox` already asks for confirmation (good pattern). |
| Daily note is too long | User stops reading it, defeats the purpose | Daily note should fit on one screen. Move reference data (full task list, all Jira tickets) to linked notes. The daily note shows only: focus, agenda, top 3 tasks, alerts. |
| Mixed language (Spanish content, English code/tags) | Cognitive overhead switching between languages | Pick one language for each layer: Spanish for content, English for tags/frontmatter/code. Be consistent -- current design already does this well. |

## "Looks Done But Isn't" Checklist

- [ ] **GTD Capture:** Often missing "capture from non-CLI contexts" -- verify you can capture a thought in < 5 seconds from any context (meeting, browser, phone)
- [ ] **Weekly Review:** Often missing the "audit projects for next actions" step -- verify every project in the backlog has at least one `#next` task after review
- [ ] **Jira Sync:** Often missing "ticket status change detection" -- verify that a ticket moved to Done in Jira disappears from the daily note within one sync cycle
- [ ] **People Notes:** Often missing "deduplication across name formats" -- verify "Pedro Lopez", "Pedro Lopez Garcia", and "pedro.lopez@company.com" all resolve to the same person note
- [ ] **Calendar Sync:** Often missing "recurring event handling" -- verify that recurring meetings don't create duplicate entries in the cache
- [ ] **Content Pipeline:** Often missing "connection back to source knowledge" -- verify that a published post links back to the knowledge entries that generated it
- [ ] **Backlog Processing:** Often missing "someday/maybe list" -- verify that non-actionable items have a place that isn't the backlog and isn't deletion
- [ ] **Dashboard (web):** Often missing "offline/stale data indicators" -- verify the dashboard shows when data was last synced, not just the data itself

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| System-as-Project Syndrome | LOW | Declare a system freeze. Use what exists for 2 weeks. Log what's actually broken vs. what's merely imperfect. Only fix what's broken. |
| Skipped Weekly Reviews | LOW | Do one review now. It will be messy. That's fine. Set a recurring calendar event. The skill can help but the habit is on you. |
| Integration Breakage | MEDIUM | Fall back to cache files. Fix the integration offline. The system should never be unusable because one integration is down. |
| Capture Friction | LOW | Add a one-line alias: `echo "- $*" >> ~/vault/inbox.md`. Use it for a week. Fancy capture can wait. |
| Vault Structure Mess | HIGH | This is why you don't restructure prematurely. If already messy: freeze, document current structure, write a migration script, migrate in one atomic operation, update config.yaml. |
| Partial GTD Loop | MEDIUM | Identify which GTD step is missing. Build the simplest version of it (even a manual checklist). Integrate into the weekly review. |
| LLM Format Drift | MEDIUM | Run a vault audit: scan all notes of a type, compare frontmatter fields. Write a normalization script. Then add schema enforcement to skills. |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| System-as-Project Syndrome | Phase 1 | System freeze enforced; user runs GTD loop daily for 2+ weeks before Phase 2 starts |
| Skipping Weekly Review | Phase 1 | `week` skill includes interactive review facilitation; review completion is tracked |
| Integration Fragility | Phase 2 | Each integration has documented timeout, cache, and failure mode. Tested with network disconnected. |
| Capture Friction | Phase 1 | Quick capture path exists that takes < 5 seconds. Capture rate is 3+ items/day. |
| Vault Over-Engineering | Phase 3+ | Vault restructure only after 4+ weeks of active use. Migration is scripted, not manual. |
| Partial GTD | Phase 1 | All 5 GTD steps have a corresponding skill or manual process. Full loop is exercised weekly. |
| LLM Nondeterminism | Phase 1 | Vault schema document exists. Spot-check after each skill run. `lint-vault` skill validates consistency. |
| Jira Rate Limiting | Phase 2 | Sync uses field filtering, caching, and respects rate limit headers. Tested with 2 instances simultaneously. |
| Apple Reminders Lock-in | Phase 2 | Documented as macOS-only. AppleScript or Swift CLI approach chosen. Fallback plan if user changes platform. |
| Dataview Performance | Phase 3+ | All queries scoped to specific folders. Tested with 1000+ note vault. Mobile rendering < 2 seconds. |

## Sources

- [10 Reasons Why GTD Might Be Failing](https://facilethings.com/blog/en/why-gtd-fails) -- GTD-specific failure modes
- [19 Common Mistakes in GTD](https://medium.com/@rubengp/19-common-mistakes-in-gtd-7-the-wrap-a1c873390a99) -- Comprehensive GTD mistake catalog
- [7 Reasons for Failing in Implementing GTD](https://medium.com/alon-sabi/7-reasons-for-failing-in-implementing-gtd-and-tips-to-overcome-them-7a95b82ab901) -- Implementation failure patterns
- [Your Second Brain Is Broken](https://medium.com/@ann_p/your-second-brain-is-broken-why-most-pkm-tools-waste-your-time-76e41dfc6747) -- PKM system abandonment patterns
- [I Wish I Knew These Before Creating My Obsidian Vault](https://www.makeuseof.com/i-wish-i-knew-these-before-creating-my-obsidian-vault/) -- Obsidian vault design mistakes
- [10 Problems with Obsidian](https://medium.com/@theo-james/10-problems-with-obsidian-youll-realize-when-it-s-too-late-17e903886847) -- Obsidian-specific pitfalls
- [What I Learned Building AI Agents on Top of the Obsidian CLI](https://www.marknagelberg.com/what-i-learned-building-ai-agents-on-top-of-the-obsidian-cli/) -- LLM + Obsidian integration challenges
- [Jira Cloud Rate Limiting](https://developer.atlassian.com/cloud/jira/platform/rate-limiting/) -- Official Jira rate limit documentation
- [Dataview Performance Issues](https://forum.obsidian.md/t/dataview-very-slow-performance/52592) -- Dataview scaling problems
- [Apple EventKit Documentation](https://developer.apple.com/documentation/eventkit) -- Apple Reminders API limitations

---
*Pitfalls research for: Personal Productivity Life OS (GTD + Obsidian + CLI + Integrations)*
*Researched: 2026-03-28*
