---
date: 2026-05-09
topic: open-ideation
focus: ""
mode: repo-grounded
---

# Ideation: Life OS — Open-Ended (Surprise-Me)

## Grounding Context

**Project shape:** Claude Code plugin marketplace + Obsidian vault (no database). 5 plugin bundles (rituals, sync, wiki, training, content), 26+ skills. TypeScript/Node.js 22, markdown-first. All vault I/O via `obsidian` CLI.

**Architecture:** Connector-consumer separation — sync-* skills write to vault, consumers read only. Config-driven via `config.yaml`. Three-tier wiki: raw notes → drafts → pages. CRITICAL_FACTS.md as operational identity layer.

**Current gaps:** No weekly planning workflow (Phase 4 blocked); wiki graduation criteria unclear; goal alignment detection not built; close ritual underdeveloped; no web dashboard.

**Strategic goal:** AI maintains the system — gathers info, detects contradictions, confronts user when acts diverge from goals, acts as thinking partner.

**Key institutional learnings:** System-as-Project Syndrome (primary failure mode); Connector-Consumer Separation; LLM Nondeterminism in vault mutations; Capture Friction kills adoption; Config-driven skills.

**External context:** No tool tightly couples CLI capture + typed entities + LLM synthesis + Obsidian vault (genuine gap). GitOps analogy: vault = desired state; morning ritual = reconciliation loop (unbuilt). AI best as routing/retrieval, not autonomous generation.

---

## Ranked Ideas

### 1. Goal Covenant Engine
**Description:** Convert `CRITICAL_FACTS.md` into a machine-readable YAML schema with explicit, measurable covenants per goal — including numeric thresholds and pre-authored breach protocols. The close ritual runs a dead-reckoning check: "Based on your last checkpoint and committed pace, you should have logged 3 training sessions — you've done 1." Breach protocols are authored at goal-setting time (when motivation is high), not improvised at failure time.

**Warrant:** `direct:` "Goal alignment detection not built (key metric: % daily focus aligned to monthly goals)" — named unbuilt strategic gap. `external:` Bond covenant monitoring: measurable thresholds with pre-agreed consequences that activate without discretionary judgment — structurally identical to maintaining commitment alignment.

**Rationale:** CRITICAL_FACTS.md is currently a prose document nobody diffs against reality. Covenants close the loop: the system has teeth for the first time. The breach response is designed when motivation is high, not improvised when it's low.

**Downsides:** Requires CRITICAL_FACTS schema redesign and close ritual extension. Covenant thresholds need tuning. Risk of gaming covenants.

**Confidence:** 85%
**Complexity:** Medium
**Status:** Unexplored

---

### 2. Morning Ritual as Commitment Audit First
**Description:** Reorder the morning ritual: open with a diff of yesterday's declared commitments (from close ritual) vs actual completions — before sync, before inbox, before new information. The close ritual is extended to record commitments (a `## Commitments` section with checkboxes). The morning becomes an accountability partner first, an information feed second.

**Warrant:** `direct:` "Close ritual is underdeveloped — no automated reflection or goal-drift detection." The close/morning gap is the same gap seen from both ends of the day.

**Rationale:** The morning ritual currently adds new information on top of unprocessed carry-over. Reordering forces confrontation before context-loading — when it's hardest to rationalize away. Closes the tightest behavioral loop in the system.

**Downsides:** Close ritual must run for morning to have something to diff against (graceful degradation: skip audit if no commitments found). Risk of high-friction mornings after hard days. Commitment format must be regex-parseable (user-authored, not LLM-generated) to avoid nondeterminism.

**Implementation sketch:** (1) Extend close ritual: prompt for 1-3 commitments, write as checkboxes under `## Commitments` in daily note (~30 min). (2) Extend morning ritual: Phase 0 reads yesterday's commitments, diffs against completion, prompts carry-forward decision (~45 min). No new infrastructure. No LLM call for the diff.

**Connection to other ideas:** Prerequisite/accelerant for Goal Covenant Engine (#1) — covenant tracking needs the close/morning loop as its raw data feed. Enriches Goal-Drift Detection (#6) and LLM Autoproposes Wiki (#3).

**Confidence:** 90%
**Complexity:** Low
**Status:** Explored (deep dive completed)

---

### 3. LLM Autoproposes Wiki Pages from Daily Notes
**Description:** Invert wiki ownership: after each close ritual (or on a schedule), an LLM pass reads the day's daily notes, meeting notes, and close log, and autonomously proposes new wiki pages or edits to existing ones — placing them in `.drafts/` for human approval. Humans approve, not author. The wiki grows at the speed of lived experience rather than the speed of motivation-to-capture.

**Warrant:** `external:` "Most systems fail at manage (curation, pruning, contradiction resolution), not write or read" — Anthropic LLM memory architecture research. Life OS has strong write and read layers; manage is entirely human-activated.

**Rationale:** If pages only get created when the user has activation energy to initiate `wiki:ingest`, the knowledge base grows unevenly. Auto-proposal makes the wiki a continuous reflection of what's actually happening.

**Downsides:** LLM cost per close ritual. Risk of draft backlog replacing editorial backlog. Schema enforcement required to prevent nondeterminism in proposed frontmatter.

**Confidence:** 78%
**Complexity:** Medium
**Status:** Unexplored

---

### 4. Self-Aware Adaptive Ritual
**Description:** Three-layered restructuring: (a) **Push trigger** — morning ritual runs on schedule (cron), pre-computes synthesis, sends a Telegram diff; user replies "ok" or edits one line to confirm; (b) **Skill ledger** — every skill invocation appends a structured entry to `skill-ledger.md` (inputs, outputs, decisions, duration); skills read own history before executing; (c) **No-LLM fallback** — each skill has a zero-cost path (templates, diffs, string matching) that runs by default; LLM path triggers only on anomaly detection.

**Warrant:** `reasoned:` Morning ritual only runs when the user has activation energy — highest when life is going well, lowest when it matters most. Push-trigger removes the activation barrier. `direct:` Graceful degradation is an existing architectural principle. `external:` Anthropic compaction pattern: skills should be self-aware of their own track record.

**Rationale:** Push makes the ritual reliable; the ledger makes skills improve over time; the fallback makes the system resilient to API outages and cost spikes. Each layer is independently useful; combined they shift the system from passive infrastructure to active agent.

**Downsides:** Push-trigger requires scheduler/cron setup. Skill ledger adds per-invocation write cost. Confirmation step may become a rubber stamp. Highest implementation complexity in this list.

**Confidence:** 75%
**Complexity:** High
**Status:** Unexplored

---

### 5. Wiki Self-Organization Layer
**Description:** The wiki re-ranks and prunes itself based on actual retrieval behavior. Pages cited in future `wiki:query` sessions accumulate a signal counter; unreferenced pages decay toward draft status after a configurable interval. A stratigraphy pass in `wiki:digest` detects gaps (concepts frequent in recent daily notes with no wiki page) and dormant knowledge (wiki pages not mentioned in any recent daily note). The digest surfaces both as a ranked action queue.

**Warrant:** `external:` Immune system clonal selection: diversity generated, signal amplifies survivors, unreferenced material decays — structurally identical to the problem of maintaining a relevant knowledge base. `direct:` "Wiki graduation criteria unclear — no automated detection"; "The graph is the value."

**Rationale:** Adds a third dynamic to the wiki (alongside grow and prune): self-organization. The stratigraphy pass is especially powerful — it connects the knowledge layer to what's actively on your mind without any explicit linking.

**Downsides:** Requires persisting query-reference counts (schema addition). Decay threshold is sensitive — too aggressive and pages demote prematurely. Stratigraphy requires reading all recent daily notes on every digest run.

**Confidence:** 80%
**Complexity:** Medium
**Status:** Unexplored

---

### 6. Invert Goal-Drift Detection — Flag Absent Goals
**Description:** Instead of semantic similarity between daily activities and goals (expensive, nondeterministic), invert: scan daily notes and close logs for presence/absence of each tracked goal ID or keyword. A goal unmentioned for N consecutive days triggers a flag — deterministic, zero-LLM cost, zero false negatives. Close ritual runs this check automatically.

**Warrant:** `reasoned:` Absence detection requires only string matching; semantic similarity requires LLM with nondeterminism risk (institutional learning). A goal you never mention is unambiguously neglected — no model needed.

**Rationale:** Solves "goal alignment detection not built" with the simplest possible mechanism. Absence is more signal-clear than divergence — you can debate whether an activity "aligns" with a goal; you cannot debate a goal was never mentioned.

**Downsides:** Requires goals expressed as identifiable keywords/IDs in CRITICAL_FACTS (overlaps with #1). Doesn't distinguish "deliberately deprioritized" from "forgotten."

**Confidence:** 92%
**Complexity:** Low
**Status:** Unexplored

---

### 7. Auto-Backlink Injection on Every Wiki Write
**Description:** When any skill writes or updates a wiki page, a post-write hook scans all existing pages for mentions of the new page's title and known aliases, and injects `[[wikilinks]]` retroactively. Runs in background, appends one log entry. Zero human action required.

**Warrant:** `direct:` "Cross-link aggressively with [[wikilink]] syntax — the graph is the value" (CLAUDE.md). Linking is currently a manual step skipped under time pressure — this automates the highest-leverage graph maintenance operation.

**Rationale:** The wiki's retrieval value compounds with graph density. Each new page retroactively enriches all prior knowledge that mentions it.

**Downsides:** May inject spurious backlinks when a page title is a common word. Requires alias management to be robust. Scan cost grows with vault size.

**Confidence:** 88%
**Complexity:** Low
**Status:** Unexplored

---

## Rejection Summary

| # | Idea | Reason Rejected |
|---|------|-----------------|
| 1 | Capture Debt Dashboard | Tactical; superseded by Goal Covenant Engine |
| 2 | Frontmatter Schema Enforcer | Tactical detail; existing lint targets this |
| 3 | Offline-Aware Morning Ritual | Tactical infrastructure fix; below ambition floor |
| 4 | Quick-Capture Zero-Classification | Partially exists via Telegram; superseded by broader ritual ideas |
| 5 | Connector Health Check | Utility feature; below ambition floor |
| 6 | /process Inbox Skill | GTD detail; superseded by Commitment Audit reframe |
| 7 | Capability-Probe Registry | Duplicates offline ritual; below ambition floor |
| 8 | Vault as a Patient | Evocative framing; too vague as a discrete actionable idea |
| 9 | Rituals as State Machines | High System-as-Project risk; significant infrastructure for uncertain payoff |
| 10 | Config as Behavior-Derived | Too expensive; System-as-Project risk; long-term vision |
| 11 | Infinite Memory / Flat Corpus | Requires embedding infrastructure not in system |
| 12 | Epidemiological Commitment Tracing | Too expensive; System-as-Project risk |
| 13 | Connector Fan-Out Registry | Infrastructure-first; System-as-Project risk at current phase |
| 14 | Phase Constraint Inverted | Development strategy, not a product feature |
| 15 | Wiki Contradiction-First KM | Good; cut in final trim to reach 7 |
| 16 | Performance Analytics Layer | Good; cut in final trim |
| 17 | Universal Capture Bus | Good; cut in final trim |
| 18 | Three-Tier Wiki Too Many | Superseded by Wiki Self-Organization Layer |
| 19 | Pull Model Wrong Architecture | Included in Self-Aware Adaptive Ritual |
| 20 | Automate Daily-to-Weekly Compaction | Superseded by LLM Autoproposes |
| 21 | Invert Wiki Graduation Gate | Superseded by Wiki Self-Organization Layer |
