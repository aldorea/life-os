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
