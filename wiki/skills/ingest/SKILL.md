---
name: ingest
description: Use when ingesting a new source into the personal wiki. Triggered by `/wiki:ingest <source>`. Source can be a URL (Phase 2a), YouTube URL, YouTube, PDF path, or inline text. Creates or updates wiki pages in 08 Resources/wiki/pages/, updates index.md and log.md.
---

# wiki:ingest

Processes a source into the personal wiki at `08 Resources/wiki/`.

## Vault Paths
- Pages: `08 Resources/wiki/pages/`
- Drafts: `08 Resources/wiki/.drafts/` (pending approval — see Step 4b)
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
Use `defuddle` to retrieve clean markdown content:
```bash
defuddle parse <url> --md
```
If defuddle is not installed: `npm install -g defuddle`. Then synthesize the output — extract main ideas, key terminology, tradeoffs. Do NOT copy the entire article.

**For YouTube URLs:**
Use `defuddle parse <url> --md` first. If it returns insufficient content (no transcript/description), fall back to WebFetch. If both fail, log the attempt and skip — do not create empty pages.

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

## Step 4b: Decide draft vs publish

For each NEW page (updates always go straight to `pages/`), decide the destination:

**Auto-publish to `pages/`:**
- Source is primary: academic paper, official docs (MDN, Python docs, RFCs, library README), book.
- Source is inline text typed by the user (assume already curated).
- The page already exists and this is just an update.
- Estimated `confidence` is `high` or `medium`.

**Send to `.drafts/` (status: draft):**
- Blog post (even reputable ones).
- Twitter/X thread.
- YouTube video without verified transcript.
- URL shared via Telegram or other social channels.
- Any new page where estimated `confidence` would be `low`.

The estimated confidence (Step 5 heuristic) is the **primary signal**. Source type is secondary.

If unclear, default to draft — it's cheap to approve later, expensive to undo a noisy publish.

## Step 5: Write Pages

**New page format (publish path):**
```
obsidian create path="08 Resources/wiki/pages/<kebab-name>.md" content="---\ntype: concept\nstatus: published\nconfidence: medium\nupdated: YYYY-MM-DD\nsources:\n  - <source>\ntags: [<category>]\n---\n\n# Title\n\n## Summary\n2-4 sentences.\n\n## Key Concepts / How It Works\nMain content.\n\n## Tradeoffs / When to Use\nHonest tradeoffs.\n\n## See Also\n- [[related-page]]\n" overwrite
```

**New page format (draft path):**

Same content, but `status: draft` and written to `.drafts/`:
```
obsidian create path="08 Resources/wiki/.drafts/<kebab-name>.md" content="---\ntype: concept\nstatus: draft\nconfidence: low\nupdated: YYYY-MM-DD\nsources:\n  - <source>\ntags: [<category>]\n---\n\n..." overwrite
```

Drafts are **not** added to `index.md`. They become discoverable only via `/wiki:approve`.

**Type rules:**
- `concept` — abstract idea, pattern, methodology
- `entity` — concrete tool, technology, library
- `comparison` — A vs B (title should be `a-vs-b.md`)
- `synthesis` — accumulated knowledge about a personal domain

**Confidence heuristic** (set in frontmatter):
- `high` — multiple primary sources (papers, official docs, books) OR direct user experience confirmed.
- `medium` — single solid secondary source (well-known blog, mature OSS docs) OR one primary source. **Default.**
- `low` — single blog post, social media thread, video without transcript verification, or any unverified opinion piece.

**Typed relations** (only when warranted, in frontmatter):
- `supersedes: ["[[old-page]]"]` — when this new page replaces an existing one. Then ALSO update the old page's frontmatter to `status: deprecated`.
- `contradicts: ["[[other-page]]"]` — when this page explicitly disagrees with another. Use sparingly; the contradiction must be substantive, not a wording difference.
- `supports: ["[[related-page]]"]` — when this page reinforces or extends another. Optional; plain `[[wikilinks]]` in body are usually enough.

**Updating existing page:**
Add new information under the relevant section. Update `updated` date. Add new source to frontmatter sources list. Preserve existing content — only add, don't remove unless there's a contradiction. If the new source raises confidence (e.g., adds a primary source), bump `confidence` accordingly.

## Step 6: Update index.md

**Skip drafts.** Only published pages enter `index.md`. Drafts live in `.drafts/` until approved.

For each new published page, add one line to the appropriate category in `index.md`:
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
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | ingest | <source>\nPages created: <comma-separated list or 'none'>\nDrafts created: <comma-separated list or 'none'>\nPages updated: <comma-separated list or 'none'>\nSummary: <one sentence describing what was captured>\n"
```

## Step 8: Report

Tell the user:
- Pages published (with names)
- Drafts created (with names) and remind them to run `/wiki:approve` to review
- Pages updated (with names and what changed)
- Cross-links added
