<!-- GSD:project-start source:PROJECT.md -->
## Project

**Life OS**

A personal operating system that centralizes task management, goal tracking, knowledge capture, CRM, and content generation into a unified system. Runs as Claude Code CLI skills + Obsidian vault + web dashboard, built on GTD methodology. Designed for a professional managing multiple projects across different Jira instances who wants one place to think, plan, and execute.

**Core Value:** A reliable GTD system that captures everything, centralizes all task sources, and ensures nothing falls through the cracks — the foundation everything else builds on.

### Constraints

- **Interaction model**: CLI (Claude Code skills) + Obsidian (visualization/navigation) + Web (dashboard)
- **Vault**: Obsidian-first — all persistent data lives as markdown in the vault
- **Meeting notes**: Source-agnostic design — abstract away from any specific provider
- **Jira**: Support dynamic project configuration (not hardcoded instances)
- **Stack (web)**: To be determined by research — no strong preference
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

Full research lives in `.planning/research/STACK.md`. Summary:

- **Web dashboard**: Next.js 15 + React 19 + TypeScript + Tailwind 4 + shadcn/ui, deployed on Vercel.
- **Skills/scripts**: Node.js 22 LTS with native `--strip-types`; `tsx` only when needed.
- **Vault utils**: `gray-matter`, `remark`/`unified`, `yaml`, `globby`.
- **Lint/test**: Biome + Vitest.
- **No database.** Obsidian vault is the data store; read via `fs` or the `obsidian` CLI.
<!-- GSD:stack-end -->

## Vault I/O

Use the `obsidian` CLI for vault file operations — it routes through Obsidian's API (handles sync, plugins, wikilinks correctly).

| Operación | Comando |
|-----------|---------|
| Leer archivo | `obsidian read path="03 Daily/04-05-2026.md"` |
| Crear / sobreescribir | `obsidian create path="..." content="..." overwrite` |
| Append | `obsidian append path="02 Inbox.md" content="..."` |
| Propiedad | `obsidian property:set name=status value=done path="..."` |
| Daily (leer) | `obsidian daily:read` |
| Daily (append) | `obsidian daily:append content="..."` |
| Listar tareas | `obsidian tasks todo path="01 Backlog.md"` |
| Toggle tarea | `obsidian task toggle path="..." line=N` |

Paths relative to vault root. Quote paths with spaces. Use `\n` for newlines in `content`. For large multi-line templates (full daily note, full Jira ticket), Write tool is acceptable — use CLI for reads, appends, and property updates.

## Operational context

`08 Resources/CRITICAL_FACTS.md` — stable identity layer (role, timezone, managers, training target, operating heuristics). Loaded as base context by `morning` / `close` and any other ritual that needs identity grounding. Calendar is reality; this file is the ideal shape used to detect drift. Lives outside `wiki/` because it is operational config, not synthesized knowledge.

## Wiki (LLM Knowledge Base)

A personal, LLM-friendly knowledge base lives at `08 Resources/wiki/` in the vault. It is the **synthesized** layer — atomic pages cross-linked with `[[wikilinks]]`, written for retrieval by future LLM sessions, not for human-only reading. Distinct from `08 Resources/knowledge/` (raw capture / topic notes).

### Vault layout

| Path | Purpose |
|------|---------|
| `08 Resources/wiki/WIKI.md` | Schema, categories, page types, conventions. **Read this before ingesting or editing pages.** |
| `08 Resources/wiki/index.md` | Curated index of all pages by category. English entries. |
| `08 Resources/wiki/log.md` | Append-only activity log (ingests, queries, lint passes, digests). |
| `08 Resources/wiki/pages/` | One markdown file per concept (kebab-case, no dates in filename). |
| `08 Resources/wiki/.drafts/` | Pending-approval pages from low-confidence sources. Not in index, not citable. |
| `08 Resources/wiki/sources/pdfs/` | PDFs ingested as sources. |

### Page contract

Every page must have:

1. Frontmatter: `type` (concept / entity / comparison / synthesis), `updated`, `sources`, `tags`.
2. `## Summary` — 2–4 sentences.
3. `## Key Concepts` / `## How It Works` — main content.
4. `## Tradeoffs` / `## When to Use`.
5. `## See Also` — wikilinks to related pages.

Rules:

- **One concept per page.** Split when in doubt.
- **Cross-link aggressively** with `[[wikilink]]` syntax — the graph is the value.
- **Language**: page in the source's language; index entries always in English.
- **Sources**: cite in frontmatter. If derived from a Claude session, write `derived from session YYYY-MM-DD`.
- **Updates**: bump the `updated` field on every edit.

### Operations (skills + slash commands)

| Skill | Slash | Purpose |
|-------|-------|---------|
| `wiki:ingest` | `/wiki:ingest <source>` | Process URL / YouTube / PDF / inline text. Routes to `pages/` (high/medium confidence + primary sources) or `.drafts/` (low confidence). |
| `wiki:approve` | `/wiki:approve [name]` | Review pending drafts. Without arg: list. With arg: show + offer approve / edit / discard. |
| `wiki:query` | `/wiki:query <question>` | Search pages and synthesize an **ephemeral** answer with citations. Ignores drafts. Suggests `/wiki:synthesize` when worth persisting. |
| `wiki:synthesize` | `/wiki:synthesize [topic]` | Promote a query answer (or a topic over existing pages) into a persistent `type: synthesis` page. Closes the loop between query and wiki. |
| `wiki:lint` | `/wiki:lint` | Health check: orphans, dead wikilinks, missing cross-refs, stale pages, contradictions, low-confidence stale, stale drafts, deprecated cleanup. |
| `wiki:audit` | `/wiki:audit [--quick]` | Structural audit of the whole second brain (editor-in-chief). Refinery flow (raw → atoms → threads), atomicity, graph connectivity, source traceability, `[FRICTION]` contradictions, core themes, gaps. Emits a scored Traceability report + prioritized backlog. Read-only — recommends, never fixes. Complements `lint` (mechanical, fixes) rather than duplicating it. |
| `wiki:digest` | `/wiki:digest [day\|week\|month]` | Activity summary from `log.md` + snapshot of pending drafts. Defaults to `week`. |

Telegram URL shares are routed to `wiki:ingest` automatically by the Telegram sync.

The `close` ritual appends a one-line `| daily |` entry to `wiki/log.md` with foco, completadas, energía y nota. `wiki:digest` reads those entries to build a per-day timeline alongside ingests/queries/synthesizes — the wiki log is the canonical cross-day timeline.

### When to use the wiki vs other surfaces

- **Synthesized concept worth retrieving later** → `wiki:ingest` or write directly into `pages/` following the schema.
- **Raw idea, link, or quote without synthesis yet** → `capture` skill into `08 Resources/knowledge/`.
- **Action / task** → Inbox / Backlog (never the wiki).
- **Meeting / daily / journal** → `07 Meetings/` / `03 Daily/`.
- **Question over my own knowledge** → `/wiki:query`. Cite the page(s) you used.

### Editing rules for Claude

- Before creating or editing a page, read `WIKI.md` to confirm the schema is current.
- Use the `obsidian` CLI (or the `obsidian:obsidian-cli` / `obsidian:obsidian-markdown` skills) for vault I/O — never write through raw fs paths to iCloud unless a skill explicitly does so.
- Never duplicate a page; check `index.md` first and prefer updating an existing page.
- Always append a one-line entry to `log.md` for any wiki write (ingest / lint fix / page edit).

<!-- GSD:profile-start -->
<!-- GSD:profile-end -->

@_bmad-output/project-context.md
