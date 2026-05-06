# Wiki Plugin — Design Spec

**Date:** 2026-05-06
**Status:** Draft

## Problem

Knowledge accumulates across sessions but doesn't compound. Articles get read and forgotten, insights from Claude sessions disappear, PDFs pile up unprocessed. The existing `08 Resources/knowledge/` folder has good intent but no structure or operations to maintain it.

## Solution

An LLM-maintained wiki following the pattern described by Andrej Karpathy: instead of re-processing raw sources on every query, the LLM incrementally builds and maintains a structured wiki of synthesized markdown pages. Knowledge compounds over time through cross-references, comparisons, and syntheses.

## Three-Layer Architecture

```
sources/     ← Raw, immutable originals. Never edited after ingest.
pages/       ← LLM-generated wiki. Claude owns this layer entirely.
WIKI.md      ← Schema: categories, conventions, workflows.
```

The separation is strict: `sources/` is read-only after ingestion. `pages/` is written and maintained exclusively by the wiki skills.

## Vault Structure

```
08 Resources/wiki/
├── WIKI.md            ← Schema and conventions (the "constitution")
├── index.md           ← Content catalog organized by category, one line per page
├── log.md             ← Append-only chronological record of all operations
├── sources/
│   ├── pdfs/          ← PDF files (downloaded at ingest time)
│   └── transcripts/   ← YouTube transcripts, pasted text
└── pages/
    └── *.md           ← Wiki pages (flat, no subfolders)
```

URLs are referenced only in the log — they are already accessible online. PDFs and transcripts are stored locally because they may disappear.

## Plugin Structure (life-os repo)

```
wiki/
├── .claude-plugin/
└── skills/
    ├── ingest.md
    ├── query.md
    ├── lint.md
    └── digest.md
```

Registered in `marketplace.json` as:
```json
{ "name": "wiki", "source": "./wiki", "version": "0.1.0", "description": "LLM Wiki: ingest, query, lint, and digest of personal knowledge." }
```

## Page Types

All pages live flat in `pages/`. No subfolders — `index.md` handles navigation.

| Type | Description | Examples |
|------|-------------|---------|
| **Concept** | Abstract idea, pattern, methodology | `event-driven-architecture.md`, `saga-pattern.md`, `gtd-methodology.md` |
| **Entity** | Concrete tool, technology, library | `mikro-orm.md`, `next-js-15.md`, `kubernetes.md`, `claude-code.md` |
| **Comparison** | A vs B with tradeoffs | `rest-vs-graphql.md`, `eda-choreography-vs-orchestration.md` |
| **Project synthesis** | Accumulated knowledge about a personal domain | `orbitant-architecture.md`, `previene-stack.md`, `life-os-design.md` |

### Page Format

```markdown
---
type: concept|entity|comparison|synthesis
updated: YYYY-MM-DD
sources:
  - https://example.com/article
  - pdfs/paper.pdf
tags: [technical, architecture]
---

# Page Title

## Summary
2-4 sentence overview.

## Key Concepts / How It Works
Main content. Structured with headers as needed.

## Tradeoffs / When to Use
Honest tradeoffs. Not everything is good for everything.

## See Also
- [[related-page]]
- [[another-page]]
```

One page = one concept. If a source contains material for two distinct concepts, the skill creates two pages.

## index.md Format

```markdown
# Wiki Index

## Technical
- [event-driven-architecture](pages/event-driven-architecture.md) — EDA patterns, brokers, choreography vs orchestration
- [saga-pattern](pages/saga-pattern.md) — distributed transactions, compensating transactions

## Tools
- [mikro-orm](pages/mikro-orm.md) — Unit of Work, batch processing, entity manager
- [kubernetes](pages/kubernetes.md) — contexts, namespaces, Docker Desktop local

## Projects
- [orbitant-architecture](pages/orbitant-architecture.md) — IDP, microservices, token exchange

## Ideas
- [gtd-methodology](pages/gtd-methodology.md) — capture, clarify, organize, review
```

## log.md Format

Append-only. Each entry has a parseable timestamp prefix and operation type.

```markdown
## 2026-05-06T10:23 | ingest | https://martinfowler.com/articles/saga.html
Pages created: saga-pattern.md
Pages updated: event-driven-architecture.md
Summary: Added saga pattern — choreography vs orchestration tradeoff explained

## 2026-05-06T14:00 | lint
Orphans: kubernetes.md (not in index) — fixed
Broken refs: 1 in orbitant-architecture.md — fixed
Contradictions: 0

## 2026-05-06T15:30 | query | "microservices patterns que conozco"
Pages consulted: event-driven-architecture.md, saga-pattern.md, orbitant-architecture.md
Result saved: no (conversational answer only)

## 2026-05-06T16:00 | digest | week
Period: 2026-04-28 — 2026-05-04
Ingests: 7 | Pages new: 2 | Pages updated: 5
```

## Skills

### `/wiki:ingest <source>`

Accepts: URL, path to PDF, YouTube URL, or inline text.

Workflow:
1. Check `log.md` — if source already ingested, ask for confirmation before re-processing
2. Fetch/read source content
3. For PDFs: copy to `sources/pdfs/`. For YouTube: save transcript to `sources/transcripts/`
4. Read relevant existing pages from `pages/` (via `index.md` to find candidates)
5. Determine what to create or update: new pages for new concepts, updates for existing ones
6. Write/update pages. One concept per page. Update `[[See Also]]` cross-links
7. Update `index.md` for any new pages
8. Append entry to `log.md`

### `/wiki:query <question>`

Workflow:
1. Read `index.md` to identify candidate pages
2. Read relevant pages
3. Synthesize answer with citations to specific wiki pages
4. If the answer reveals something worth persisting, offer to create a new synthesis page

### `/wiki:lint`

Health checks:
- **Orphans**: pages in `pages/` not listed in `index.md`
- **Dead links**: `[[wikilinks]]` referencing non-existent pages
- **Missing cross-refs**: pages that mention a concept that has a page but don't link to it
- **Stale pages**: pages not updated in >90 days with recent sources available (flag, don't auto-fix)
- **Contradictions**: conflicting claims between pages (LLM judgment call)

Appends lint report to `log.md`.

### `/wiki:digest [day|week|month]`

Reads `log.md` and outputs a formatted activity summary:

```
📅 Week 2026-04-28 — 2026-05-04
Ingests: 7  (3 URLs, 2 PDFs, 1 YouTube, 1 text)
Pages new: saga-pattern, staff-plus-career
Pages updated: event-driven-architecture, orbitant-architecture
Queries: 3
Lint: 1 pass — 2 orphans resolved
```

## WIKI.md Schema (excerpt)

```markdown
# Wiki Schema

## Categories
- Technical: patterns, architectures, CS concepts
- Tools: specific software, libraries, frameworks
- Projects: accumulated knowledge about personal/work domains
- Ideas: methodologies, mental models, non-technical concepts

## Conventions
- One concept per page. Split when in doubt.
- Filenames: kebab-case, descriptive, no dates.
- Cross-links: always use [[wikilinks]] syntax.
- Sources: always cite in frontmatter — if unknown, write "derived from session".
- Language: pages in the language of the source. Index entries always in English.

## What does NOT go here
- Tasks or todos → 01 Tasks/
- Meeting notes → 07 Meetings/
- Daily notes → 03 Daily/
- Raw bookmarks without synthesis → they stay as URLs until ingested
```

## Roadmap

| Phase | Skills | Deliverable |
|-------|--------|-------------|
| 1 — Foundation | — | Vault structure: `wiki/` dir, `WIKI.md`, empty `index.md`, empty `log.md` |
| 2a — Ingest URL | `/wiki:ingest` | Process URLs → create/update pages + index + log. Validates end-to-end flow. |
| 2b — Ingest all sources | `/wiki:ingest` | Add YouTube, PDF, and text support on top of the working URL flow |
| 3 — Query | `/wiki:query` | Search pages + synthesize answer with citations |
| 4 — Lint | `/wiki:lint` | Detect orphans, dead links, missing cross-refs, contradictions |
| 5 — Digest | `/wiki:digest` | Activity summary by day/week/month from log |

## What This Is Not

- Not a replacement for `learn`/`capture` skills — those are session-based, this is permanent
- Not a task manager — tasks stay in `01 Tasks/`
- Not a search engine — it's a curated, synthesized knowledge base
- Not automated — ingest is always explicit (`/wiki:ingest <source>`)

## Future Plugins (out of scope for this spec)

- **content plugin** — content pipeline fed by wiki knowledge
- **alignment plugin** — activity tracking against monthly/quarterly goals
