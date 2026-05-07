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

Two sources to check:
1. **Declared contradictions** — frontmatter `contradicts: [[other]]`. For each, verify that `other` exists in `pages/`. Flag dangling targets.
2. **Implicit contradictions** — read pages and use judgment. Examples:
   - One page says "use X for Y", another says "avoid X for Y"
   - Different version numbers for the same tool

**Fix:** Flag the contradiction. Do not auto-resolve. Report both claims and their source pages. If implicit, suggest the user add `contradicts:` to the frontmatter to make it declared.

### Check 6: Low-confidence stale pages
Pages with `confidence: low` AND `updated` older than 30 days.

These are likely to have been written from a single weak source and never re-verified. Flag for the user to either: re-ingest with a stronger source (raises confidence), or accept as-is (mark `confidence: medium` if still trusted).

**Do not auto-fix.** Report:
> "[[page-name]] has confidence: low and is N days old. Consider re-ingesting from a primary source."

### Check 7: Deprecated pages still in index
Pages with `status: deprecated` that still appear in `index.md`.

**Fix:** Remove from `index.md`. The page stays in `pages/` for history but should not be discoverable.

### Check 8b: Stale drafts
Drafts in `.drafts/` with `updated` older than 14 days.

```bash
ls "$VAULT/08 Resources/wiki/.drafts/"
```

For each draft, read frontmatter `updated`. Flag those >14 days old.

**Do not auto-fix.** Report:
> "[[<name>]] has been in drafts for N days. Run `/wiki:approve <name>` to approve, edit, or discard."

### Check 8: Orphan deprecations
Pages with `status: deprecated` that have no `supersedes` declared on any other page pointing to them.

A deprecated page should always have a successor. If none exists, either: undeprecate it, or delete it. **Do not auto-fix.** Report to user.

## Report Format

After all checks:

```
## Lint Report — YYYY-MM-DD

**Orphans fixed:** N pages added to index
**Dead links:** N found (listed below)
**Cross-refs added:** N wikilinks added
**Stale pages:** N pages flagged
**Contradictions:** N declared / M implicit (listed below)
**Low-confidence stale:** N pages flagged
**Stale drafts:** N flagged (>14 days)
**Deprecated cleanup:** N removed from index, M orphan deprecations
```

### Dead links requiring attention
- [[missing-page]] referenced in [[source-page]] — [suggested action]

### Stale pages
- [[page-name]] — last updated YYYY-MM-DD (N days ago)

### Contradictions
- [[page-a]] says X; [[page-b]] says Y — [description of conflict]

### Low-confidence stale
- [[page-name]] — confidence: low, updated N days ago

### Stale drafts
- [[draft-name]] — N days in drafts. Run `/wiki:approve <name>`.

### Orphan deprecations
- [[page-name]] — status: deprecated, no successor declared

## Step 6: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | lint\nOrphans fixed: N\nDead links: N\nCross-refs added: N\nStale pages: N\nContradictions: N declared / M implicit\nLow-confidence stale: N\nStale drafts: N\nDeprecated cleanup: N from index, M orphan\n"
```
