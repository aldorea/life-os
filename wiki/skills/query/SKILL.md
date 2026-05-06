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
