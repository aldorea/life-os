---
name: ingest
description: Use when ingesting a new source into the personal wiki. Triggered by `/wiki:ingest <source>`. Source can be a URL (Phase 2a), YouTube URL, YouTube, PDF path, or inline text. Creates or updates wiki pages in 08 Resources/wiki/pages/, updates index.md and log.md.
---

# wiki:ingest

Processes a source into the personal wiki at `08 Resources/wiki/`.

## Vault Paths
- Pages: `08 Resources/wiki/pages/`
- Index: `08 Resources/wiki/index.md`
- Log: `08 Resources/wiki/log.md`
- Schema: `08 Resources/wiki/WIKI.md`
- PDF sources: `08 Resources/wiki/sources/pdfs/`
- Transcript sources: `08 Resources/wiki/sources/transcripts/`

## Supported Source Types
- **URL** — any web article or documentation page
- **YouTube URL** (contains `youtube.com` or `youtu.be`) — fetch transcript via WebFetch on the video page, extract auto-generated transcript or description
- **PDF path** — local file path ending in `.pdf`
- **Text** — inline text passed directly after the command

## Step 1: Duplicate Check

Read the log to check if source was already processed:
```
obsidian read path="08 Resources/wiki/log.md"
```

Search for the exact source URL/path in the log entries. If found:
> "Esta fuente ya fue ingestada el [date]. ¿Quieres procesarla de nuevo?"

Stop unless the user confirms.

## Step 2: Fetch Source Content

**For URLs:**
Use WebFetch to retrieve the page. Extract:
- Main ideas and concepts
- Key terminology
- Tradeoffs or comparisons mentioned
- Do NOT copy the entire article — synthesize

**For YouTube URLs:**
Use WebFetch on the URL. Look for transcript, auto-generated captions, or video description to extract the main ideas.

**For PDFs:**
Read the file from the local path. Copy it to vault:
```bash
cp "<source-path>" "/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/08 Resources/wiki/sources/pdfs/<filename>.pdf"
```
Then extract the key ideas.

**For text:**
Use the text directly as the source content.

## Step 3: Read Existing Wiki State

```
obsidian read path="08 Resources/wiki/index.md"
```

Identify pages that might need updating based on the source content.

## Step 4: Plan Pages

For each distinct concept in the source, decide:
- If a matching page exists in `pages/` → update it
- If no matching page exists → create it

One concept = one page. When in doubt, create two smaller pages rather than one large one.

For each page to update, read it first:
```
obsidian read path="08 Resources/wiki/pages/<page-name>.md"
```

## Step 5: Write Pages

**New page format:**
```
obsidian create path="08 Resources/wiki/pages/<kebab-name>.md" content="---\ntype: concept\nupdated: YYYY-MM-DD\nsources:\n  - <source>\ntags: [<category>]\n---\n\n# Title\n\n## Summary\n2-4 sentences.\n\n## Key Concepts / How It Works\nMain content.\n\n## Tradeoffs / When to Use\nHonest tradeoffs.\n\n## See Also\n- [[related-page]]\n" overwrite
```

**Type rules:**
- `concept` — abstract idea, pattern, methodology
- `entity` — concrete tool, technology, library
- `comparison` — A vs B (title should be `a-vs-b.md`)
- `synthesis` — accumulated knowledge about a personal domain

**Updating existing page:**
Add new information under the relevant section. Update `updated` date. Add new source to frontmatter sources list. Preserve existing content — only add, don't remove unless there's a contradiction.

## Step 6: Update index.md

For each new page created, add one line to the appropriate category in `index.md`:
```
- [page-name](pages/page-name.md) — one-line description of what the page covers
```

Categories: Technical, Tools, Projects, Ideas.

Read index, add entries, rewrite with:
```
obsidian create path="08 Resources/wiki/index.md" content="<updated content>" overwrite
```

## Step 7: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | ingest | <source>\nPages created: <comma-separated list or 'none'>\nPages updated: <comma-separated list or 'none'>\nSummary: <one sentence describing what was captured>\n"
```

## Step 8: Report

Tell the user:
- Pages created (with names)
- Pages updated (with names and what changed)
- Cross-links added
