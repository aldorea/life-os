---
name: capture
description: Use when capturing knowledge, interesting links, ideas, or learnings into the wiki. Use when user says "capture", "captura", "guarda esto", "esto es interesante", "save this", or shares a link/idea they want to remember. Writes to `08 Resources/wiki/pages/` (or `.drafts/`) following the wiki schema.
---

# capture

Captures knowledge inputs into the wiki. Light-weight alternative to `/wiki:ingest` — for short observations, ideas, and quick links that don't justify a full ingest pipeline. Larger sources (PDFs, articles) → use `/wiki:ingest`. Promotion of a query answer → `/wiki:synthesize`.

## Single knowledge layer

Everything goes to `08 Resources/wiki/`. The legacy `knowledge/` folder no longer exists. Schema: `08 Resources/wiki/WIKI.md`.

## Input Types

- A URL: `/capture https://blog.com/article` → routes to `/wiki:ingest` (defuddle pipeline)
- A text idea: `/capture "idea: usar granola para alimentar CRM"` → entry in a synthesis page
- A free-form observation: `/capture me ha gustado cómo Felipe usa obsidian como CRM` → entry
- A tool/resource: `/capture herramienta: Whisper Flow para escribir por voz` → entry

## Process

### 1. Route by input

- **URL** → invoke `/wiki:ingest <url>` and stop. The ingest skill is more thorough (defuddle, draft-gate, full frontmatter).
- **Idea / observation / tool** → continue with this skill (lightweight thematic capture).

### 2. Identify the topic

Read the wiki index:

```
obsidian read path="08 Resources/wiki/index.md"
```

- If a matching `type: synthesis` page exists → will append to it.
- If no match → propose a new synthesis page name and ask user to confirm. Use kebab-case.

### 3. Check for duplicates

If the page exists, read it and scan the `## Timeline` section. If the insight is already captured (same source or same idea) → tell user and skip.

### 4. Decide draft vs publish

Use `confidence` heuristic from `08 Resources/wiki/WIKI.md`:

- **Auto-publish** to `wiki/pages/` if it's a high-quality observation, your direct experience, or a primary source.
- **Draft** to `wiki/.drafts/` if it's a quick link, a low-confidence opinion, or you want to revisit before publishing.

Default: when in doubt, draft. The user can promote later via `/wiki:approve`.

### 5. Create or update the page

**New page** (synthesis topic, first entry):

```yaml
---
type: synthesis
status: published
confidence: low
updated: YYYY-MM-DD
sources:
  - "<source URL or 'derived from session YYYY-MM-DD'>"
tags: [<category>, <topic-tags>]
---

# <Topic Title>

## Summary
1-2 sentences describing the topic.

## Timeline

### [YYYY-MM-DD] First entry title
**Tipo:** articulo | experiencia | idea | herramienta | lectura
**Fuente:** [URL or context description]

3-5 lines: what was learned, why it matters, key takeaway.
Connect with [[wikilinks]] to people, projects, tickets, other pages.

## See Also
- [[related-page]]
```

Write with:
```
obsidian create path="08 Resources/wiki/pages/<kebab-name>.md" content="..." overwrite
```

**Existing page** (append a new entry):

Read the page, add a new entry at the **top** of `## Timeline` (most recent first), bump `updated`, append the new source to `sources:` if it's distinct, and rewrite the file.

If after this entry the page has accumulated multiple primary sources, bump `confidence` (low → medium → high) — see the schema for criteria.

### 6. Update index.md

If a new page was created, add a one-line entry under the appropriate category in `08 Resources/wiki/index.md`.

### 7. Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | capture | <topic>\nPage: <name>.md (created | updated)\nEntry type: <type>\nSource: <source>\n"
```

### 8. Check for content potential

If the page now has `confidence: high` AND >7 Timeline entries:

> "La página '<topic>' tiene N entradas y confidence high. ¿Quieres preparar un draft de contenido?"

If user accepts, also add to `08 Resources/content-ideas.md`.

## Important Rules

- ALWAYS ask before saving — never auto-save.
- Capture the LEARNING, not the task: "discovered pattern X solves Y" > "did task Z".
- Keep entries to 3–5 lines — concise.
- Always use [[wikilinks]] to connect to People, Projects, Jira tickets, other wiki pages.
- Before creating a wikilink, verify the target note exists; if not, suggest creating a stub.
- Spanish language for content.
- If the input is a URL, prefer `/wiki:ingest` (richer pipeline). Use this skill only for short text-based captures.

## Vault Path

`/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault`
