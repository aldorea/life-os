# Wiki Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a `wiki` Life OS plugin with 4 skills (ingest, query, lint, digest) that maintains a persistent LLM wiki in the Obsidian vault following the Karpathy pattern.

**Architecture:** Skills are markdown files (SKILL.md) read by Claude Code. All data lives in the Obsidian vault at `08 Resources/wiki/`. The plugin is registered in the life-os marketplace. Skills use the `obsidian` CLI for vault I/O and WebFetch for remote sources.

**Tech Stack:** Obsidian CLI, WebFetch, Claude Code skills (markdown), life-os marketplace plugin system.

---

## File Map

### life-os repo (new)
| File | Responsibility |
|------|---------------|
| `wiki/.claude-plugin/plugin.json` | Plugin manifest for marketplace |
| `wiki/skills/ingest/SKILL.md` | Ingest skill: process URL/YouTube/PDF/text → wiki pages |
| `wiki/skills/query/SKILL.md` | Query skill: search wiki + synthesize answer |
| `wiki/skills/lint/SKILL.md` | Lint skill: health check the wiki |
| `wiki/skills/digest/SKILL.md` | Digest skill: activity summary from log |

### life-os repo (modified)
| File | Change |
|------|--------|
| `.claude-plugin/marketplace.json` | Add wiki plugin entry |

### Obsidian vault (new, created via obsidian CLI)
| Path | Responsibility |
|------|---------------|
| `08 Resources/wiki/WIKI.md` | Schema: categories, conventions, page structure |
| `08 Resources/wiki/index.md` | Content catalog organized by category |
| `08 Resources/wiki/log.md` | Append-only operation log |
| `08 Resources/wiki/sources/pdfs/` | Downloaded PDF files (immutable after ingest) |
| `08 Resources/wiki/sources/transcripts/` | YouTube transcripts and pasted text |
| `08 Resources/wiki/pages/` | Synthesized wiki pages (flat, no subfolders) |

---

## Task 1: Vault Structure

**Files:**
- Create: `08 Resources/wiki/WIKI.md` (vault)
- Create: `08 Resources/wiki/index.md` (vault)
- Create: `08 Resources/wiki/log.md` (vault)

- [ ] **Step 1: Create wiki directory and subdirectories in vault**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
mkdir -p "$VAULT/08 Resources/wiki/sources/pdfs"
mkdir -p "$VAULT/08 Resources/wiki/sources/transcripts"
mkdir -p "$VAULT/08 Resources/wiki/pages"
```

Expected: directories created, no errors.

- [ ] **Step 2: Create WIKI.md schema file**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
cat > "$VAULT/08 Resources/wiki/WIKI.md" << 'EOF'
---
type: schema
updated: 2026-05-06
---

# Wiki Schema

## Categories
- **Technical**: patterns, architectures, CS concepts, algorithms
- **Tools**: specific software, libraries, frameworks, CLIs
- **Projects**: accumulated knowledge about personal/work domains
- **Ideas**: methodologies, mental models, non-technical concepts

## Page Types
- **concept**: Abstract idea, pattern, methodology
- **entity**: Concrete tool, technology, library
- **comparison**: A vs B with explicit tradeoffs
- **synthesis**: Accumulated knowledge about a personal/work domain

## Conventions
- One concept per page. Split when in doubt.
- Filenames: kebab-case, descriptive, no dates (e.g., `event-driven-architecture.md`)
- Cross-links: always use [[wikilinks]] syntax for references between pages
- Sources: always cite in frontmatter — if from a Claude session, write "derived from session YYYY-MM-DD"
- Language: pages in the language of the source. Index entries always in English.
- Updates: when updating an existing page, always update the `updated` date in frontmatter

## Page Structure
Every page must have:
1. Frontmatter (type, updated, sources, tags)
2. ## Summary (2-4 sentences)
3. ## Key Concepts / How It Works (main content)
4. ## Tradeoffs / When to Use
5. ## See Also (wikilinks to related pages)

## What Does NOT Go Here
- Tasks or todos → 01 Tasks/
- Meeting notes → 07 Meetings/
- Daily notes → 03 Daily/
- Raw bookmarks without synthesis → log the URL, ingest when ready
- Session insights → use capture skill, not wiki

## Operations
- `/wiki:ingest <source>` — process a source into wiki pages
- `/wiki:query <question>` — search and synthesize from wiki
- `/wiki:lint` — health check the wiki
- `/wiki:digest [day|week|month]` — activity summary from log
EOF
```

- [ ] **Step 3: Create empty index.md**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
cat > "$VAULT/08 Resources/wiki/index.md" << 'EOF'
---
updated: 2026-05-06
---

# Wiki Index

## Technical
<!-- concept and architecture pages -->

## Tools
<!-- tool and library pages -->

## Projects
<!-- project synthesis pages -->

## Ideas
<!-- methodology and mental model pages -->
EOF
```

- [ ] **Step 4: Create empty log.md**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
cat > "$VAULT/08 Resources/wiki/log.md" << 'EOF'
---
updated: 2026-05-06
---

# Wiki Log

Append-only. Format: `## YYYY-MM-DDTHH:MM | <operation> | <detail>`

EOF
```

- [ ] **Step 5: Verify vault structure**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
find "$VAULT/08 Resources/wiki" -type f -o -type d | sort
```

Expected output:
```
.../08 Resources/wiki
.../08 Resources/wiki/WIKI.md
.../08 Resources/wiki/index.md
.../08 Resources/wiki/log.md
.../08 Resources/wiki/pages
.../08 Resources/wiki/sources
.../08 Resources/wiki/sources/pdfs
.../08 Resources/wiki/sources/transcripts
```

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat(wiki): phase 1 — vault structure (WIKI.md, index.md, log.md)"
```

---

## Task 2: Plugin Skeleton

**Files:**
- Create: `wiki/.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create plugin directory and plugin.json**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/.claude-plugin
cat > /Users/sito/Documents/life-os/wiki/.claude-plugin/plugin.json << 'EOF'
{
  "name": "wiki",
  "description": "LLM Wiki: ingest, query, lint, and digest of personal knowledge following the Karpathy pattern.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["wiki", "knowledge", "obsidian", "productivity", "llm"]
}
EOF
```

- [ ] **Step 2: Create skills directory**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/skills
```

- [ ] **Step 3: Add wiki entry to marketplace.json**

Edit `.claude-plugin/marketplace.json` — add to the `plugins` array:

```json
{
  "name": "wiki",
  "source": "./wiki",
  "version": "0.1.0",
  "description": "LLM Wiki: ingest, query, lint, and digest of personal knowledge."
}
```

- [ ] **Step 4: Verify marketplace.json is valid JSON**

```bash
cat /Users/sito/Documents/life-os/.claude-plugin/marketplace.json | python3 -m json.tool > /dev/null && echo "valid JSON"
```

Expected: `valid JSON`

- [ ] **Step 5: Commit**

```bash
git add wiki/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "feat(wiki): phase 1 — plugin skeleton and marketplace registration"
```

---

## Task 3: Ingest Skill — Phase 2a (URL)

**Files:**
- Create: `wiki/skills/ingest/SKILL.md`

- [ ] **Step 1: Create ingest skill directory**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/skills/ingest
```

- [ ] **Step 2: Write ingest SKILL.md**

Create `wiki/skills/ingest/SKILL.md` with this exact content:

````markdown
---
name: ingest
description: Use when ingesting a new source into the personal wiki. Triggered by `/wiki:ingest <source>`. Source can be a URL (Phase 2a), YouTube URL, PDF path, or inline text (Phase 2b). Creates or updates wiki pages in 08 Resources/wiki/pages/, updates index.md and log.md.
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
````

- [ ] **Step 3: Validate skill file exists and is readable**

```bash
cat /Users/sito/Documents/life-os/wiki/skills/ingest/SKILL.md | head -5
```

Expected: shows the frontmatter with `name: ingest`.

- [ ] **Step 4: Commit**

```bash
git add wiki/skills/ingest/SKILL.md
git commit -m "feat(wiki): phase 2a — ingest skill for URLs"
```

---

## Task 4: Validate Ingest with Real URL

This task validates the end-to-end flow of the ingest skill with a real URL before building additional source types.

- [ ] **Step 1: Invoke the ingest skill**

Run in Claude Code:
```
/wiki:ingest https://martinfowler.com/articles/201701-event-driven.html
```

- [ ] **Step 2: Verify log.md was updated**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
tail -10 "$VAULT/08 Resources/wiki/log.md"
```

Expected: a new entry with today's date, the URL, and pages created/updated.

- [ ] **Step 3: Verify at least one page was created in pages/**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
ls "$VAULT/08 Resources/wiki/pages/"
```

Expected: at least one `.md` file.

- [ ] **Step 4: Verify index.md was updated**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
cat "$VAULT/08 Resources/wiki/index.md"
```

Expected: the new page appears under the appropriate category.

- [ ] **Step 5: Verify page has correct structure**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
cat "$VAULT/08 Resources/wiki/pages/$(ls "$VAULT/08 Resources/wiki/pages/" | head -1)"
```

Expected: frontmatter with `type`, `updated`, `sources`, `tags` + sections: Summary, Key Concepts, Tradeoffs, See Also.

- [ ] **Step 6: If validation passes, commit test artifacts**

```bash
git add -A
git commit -m "test(wiki): validate ingest skill end-to-end with martinfowler.com URL"
```

---

## Task 5: Ingest Skill — Phase 2b (YouTube, PDF, Text)

**Files:**
- Modify: `wiki/skills/ingest/SKILL.md` (extend with new source types — already included in Task 3's SKILL.md, this task validates them)

Phase 2b source types are already documented in the ingest SKILL.md from Task 3. This task validates each one works.

- [ ] **Step 1: Test YouTube ingest**

```
/wiki:ingest https://www.youtube.com/watch?v=STKCRSUsyP0
```

Verify: log entry created, at least one page created or updated, index updated.

- [ ] **Step 2: Test text ingest**

```
/wiki:ingest "El patrón Outbox garantiza que los eventos se publican si y solo si la transacción de base de datos se confirma. Usa una tabla outbox en la misma DB y un proceso separado que la lee y publica al broker."
```

Verify: log entry created, a page for outbox pattern (or similar) created or updated.

- [ ] **Step 3: Test PDF ingest (if a PDF is available)**

If no PDF is handy, skip and note in log:
```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
echo "## $(date -u +%Y-%m-%dT%H:%M) | ingest | PDF test skipped — no test PDF available" >> "$VAULT/08 Resources/wiki/log.md"
```

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "test(wiki): validate ingest phase 2b — YouTube and text sources"
```

---

## Task 6: Query Skill

**Files:**
- Create: `wiki/skills/query/SKILL.md`

- [ ] **Step 1: Create query skill directory**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/skills/query
```

- [ ] **Step 2: Write query SKILL.md**

Create `wiki/skills/query/SKILL.md`:

````markdown
---
name: query
description: Use when querying the personal wiki. Triggered by `/wiki:query <question>`. Searches wiki pages and synthesizes an answer with citations to specific pages. Optionally saves the result as a new wiki page.
---

# wiki:query

Searches the wiki and synthesizes answers from your own knowledge.

## Vault Paths
- Index: `08 Resources/wiki/index.md`
- Pages: `08 Resources/wiki/pages/`
- Log: `08 Resources/wiki/log.md`

## Process

### Step 1: Read the index

```
obsidian read path="08 Resources/wiki/index.md"
```

Scan for pages relevant to the question. Identify 3-7 candidate pages.

### Step 2: Read candidate pages

For each candidate page:
```
obsidian read path="08 Resources/wiki/pages/<page-name>.md"
```

### Step 3: Synthesize answer

Write an answer that:
- Directly addresses the question
- Cites specific wiki pages using [[wikilinks]]
- Notes gaps or contradictions between pages
- Is grounded only in what the wiki contains — don't add external knowledge without flagging it

Format:
```
**Answer:** [direct answer]

**From your wiki:**
- [[page-name]] — [what this page contributes]
- [[other-page]] — [what this page contributes]

**Gaps:** [what's missing from the wiki that would improve this answer, if anything]
```

### Step 4: Offer to persist

If the synthesized answer reveals a new perspective or synthesis worth keeping:
> "Este resultado conecta [[page-a]] y [[page-b]] de una forma que no está documentada. ¿Quiero crear una página de síntesis?"

If yes, create the page following the standard page format with `type: synthesis`.

### Step 5: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | query | <question>\nPages consulted: <comma-separated list>\nResult saved: yes (<page-name>.md) | no\n"
```
````

- [ ] **Step 3: Validate query skill with a real question**

Run (after Task 4 has created at least one page):
```
/wiki:query "qué sé sobre event-driven architecture"
```

Verify:
- Answer cites at least one wiki page
- Log updated with query entry

- [ ] **Step 4: Commit**

```bash
git add wiki/skills/query/SKILL.md
git commit -m "feat(wiki): phase 3 — query skill"
```

---

## Task 7: Lint Skill

**Files:**
- Create: `wiki/skills/lint/SKILL.md`

- [ ] **Step 1: Create lint skill directory**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/skills/lint
```

- [ ] **Step 2: Write lint SKILL.md**

Create `wiki/skills/lint/SKILL.md`:

````markdown
---
name: lint
description: Use when running a health check on the wiki. Triggered by `/wiki:lint`. Detects orphan pages, dead wikilinks, missing cross-references, stale pages, and contradictions. Fixes what can be fixed automatically. Appends report to log.md.
---

# wiki:lint

Health check for `08 Resources/wiki/`. Finds and fixes structural issues.

## Vault Paths
- Index: `08 Resources/wiki/index.md`
- Pages dir: `08 Resources/wiki/pages/`
- Log: `08 Resources/wiki/log.md`

## Checks (run all, then report)

### Check 1: Orphan pages
Pages in `pages/` that are not listed in `index.md`.

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
ls "$VAULT/08 Resources/wiki/pages/"
```

Read `index.md` and compare. Any page filename not referenced in index = orphan.

**Fix:** Add orphan to appropriate category in index.md with a one-line description generated from the page's Summary section.

### Check 2: Dead wikilinks
`[[links]]` in any page that reference a page that doesn't exist in `pages/`.

For each page, read it and extract all `[[wikilink]]` patterns. Check that `pages/<wikilink>.md` exists.

**Fix:** Do NOT auto-create pages. Flag the dead link and suggest either:
- Creating the missing page (if the concept deserves one)
- Removing the link (if it was a mistake)

Report each dead link to the user with the source page and the missing target.

### Check 3: Missing cross-references
Pages that mention a concept by name that has a wiki page, but don't link to it with `[[wikilinks]]`.

Example: `event-driven-architecture.md` mentions "saga" but doesn't link to `[[saga-pattern]]`.

**Fix:** Add the missing wikilink to the `## See Also` section of the relevant page.

### Check 4: Stale pages
Pages with `updated` date older than 90 days.

Read each page's frontmatter `updated` field. Compare to today. Flag pages not updated in >90 days.

**Do not auto-fix.** Report to user:
> "[[page-name]] was last updated on [date] (N days ago). Consider re-ingesting a recent source on this topic."

### Check 5: Contradictions
Pages that make conflicting claims about the same concept.

Read all pages and use judgment to identify contradictions. Examples:
- One page says "use X for Y", another says "avoid X for Y"
- Different version numbers for the same tool

**Fix:** Flag the contradiction. Do not auto-resolve. Report both claims and their source pages.

## Report Format

After all checks:

```
## Lint Report — YYYY-MM-DD

**Orphans fixed:** N pages added to index
**Dead links:** N found (listed below)
**Cross-refs added:** N wikilinks added
**Stale pages:** N pages flagged
**Contradictions:** N found (listed below)

### Dead links requiring attention
- [[missing-page]] referenced in [[source-page]] — [suggested action]

### Stale pages
- [[page-name]] — last updated YYYY-MM-DD (N days ago)

### Contradictions
- [[page-a]] says X; [[page-b]] says Y — [description of conflict]
```

## Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | lint\nOrphans fixed: N\nDead links: N\nCross-refs added: N\nStale pages: N\nContradictions: N\n"
```
````

- [ ] **Step 3: Validate lint skill**

Run:
```
/wiki:lint
```

Verify:
- At least runs without error
- Log updated with lint entry
- Any orphans or dead links are reported

- [ ] **Step 4: Commit**

```bash
git add wiki/skills/lint/SKILL.md
git commit -m "feat(wiki): phase 4 — lint skill"
```

---

## Task 8: Digest Skill

**Files:**
- Create: `wiki/skills/digest/SKILL.md`

- [ ] **Step 1: Create digest skill directory**

```bash
mkdir -p /Users/sito/Documents/life-os/wiki/skills/digest
```

- [ ] **Step 2: Write digest SKILL.md**

Create `wiki/skills/digest/SKILL.md`:

````markdown
---
name: digest
description: Use when reviewing wiki activity over a time period. Triggered by `/wiki:digest [day|week|month]`. Reads log.md and produces a formatted summary of ingests, queries, lint passes, and pages created/updated. Defaults to 'week' if no argument given.
---

# wiki:digest

Activity summary for the personal wiki from `log.md`.

## Vault Paths
- Log: `08 Resources/wiki/log.md`

## Process

### Step 1: Determine period

- `day` → today (current date)
- `week` → last 7 days
- `month` → last 30 days
- No argument → default to `week`

### Step 2: Read log

```
obsidian read path="08 Resources/wiki/log.md"
```

### Step 3: Filter entries for the period

Parse entries with format `## YYYY-MM-DDTHH:MM | <operation> | <detail>`. Select only entries within the date range.

### Step 4: Aggregate stats

From filtered entries, count:
- **Ingests**: total entries with `| ingest |`
  - Break down by source type: URL (starts with `http`), YouTube (`youtube.com`/`youtu.be`), PDF (ends with `.pdf`), text (everything else)
- **Pages created**: sum of `Pages created:` values across all ingest entries (excluding `none`)
- **Pages updated**: sum of `Pages updated:` values across all ingest entries (excluding `none`)
- **Queries**: total entries with `| query |`
- **Lint passes**: total entries with `| lint |`

### Step 5: Format report

```
📅 [Period label]
─────────────────────────────
Ingests: N  (X URLs, Y PDFs, Z YouTube, W text)
Pages new: [page1, page2, ...]
Pages updated: [page1, page2, ...]
Queries: N
Lint: N pass(es)
─────────────────────────────
```

If no activity in the period:
> "No wiki activity in the last [period]."

### Step 6: No log entry needed

`digest` is read-only — do not append to log.md.
````

- [ ] **Step 3: Validate digest skill**

Run (after at least one ingest from previous tasks):
```
/wiki:digest week
```

Verify: shows a summary with at least the ingest activity from the test runs.

- [ ] **Step 4: Commit**

```bash
git add wiki/skills/digest/SKILL.md
git commit -m "feat(wiki): phase 5 — digest skill"
```

---

## Task 9: Final Validation

- [ ] **Step 1: Verify complete plugin structure**

```bash
find /Users/sito/Documents/life-os/wiki -type f | sort
```

Expected:
```
wiki/.claude-plugin/plugin.json
wiki/skills/digest/SKILL.md
wiki/skills/ingest/SKILL.md
wiki/skills/lint/SKILL.md
wiki/skills/query/SKILL.md
```

- [ ] **Step 2: Verify vault structure**

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
find "$VAULT/08 Resources/wiki" | sort
```

Expected: WIKI.md, index.md, log.md, sources/pdfs/, sources/transcripts/, pages/ + any pages from test ingests.

- [ ] **Step 3: Verify marketplace.json includes wiki**

```bash
cat /Users/sito/Documents/life-os/.claude-plugin/marketplace.json | python3 -c "import json,sys; plugins=[p['name'] for p in json.load(sys.stdin)['plugins']]; print('wiki plugin found' if 'wiki' in plugins else 'MISSING wiki plugin')"
```

Expected: `wiki plugin found`

- [ ] **Step 4: Run all 4 skills one final time**

```
/wiki:ingest https://www.swyx.io/writing/llm-wiki
/wiki:query "what do I know about knowledge management"
/wiki:lint
/wiki:digest week
```

Verify each produces expected output without errors.

- [ ] **Step 5: Final commit**

```bash
git add -A
git commit -m "feat(wiki): complete plugin — all 4 skills validated (ingest, query, lint, digest)"
```
