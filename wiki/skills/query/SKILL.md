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

**Never read from `.drafts/`.** Drafts are unapproved content and must not be cited.

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

**Honor frontmatter signals when citing:**
- Skip pages with `status: deprecated` unless the question is historical. If used, label them `(deprecated)`.
- Skip pages with `status: stub` — they have no content yet. Mention the stub as a gap instead.
- When citing a page with `confidence: low`, flag it inline: `[[page]] (low confidence)`. Do not weight it equally with high-confidence pages.
- If candidate pages declare `contradicts: [[other]]` between them, **surface the tension explicitly** — present both positions, do not silently pick one.
- If a page declares `supersedes: [[old]]`, prefer the newer page; cite `[[old]]` only if specifically asked about prior state.

Format:
```
**Answer:** [direct answer]

**From your wiki:**
- [[page-name]] — [what this page contributes]
- [[other-page]] (low confidence) — [what this page contributes]

**Tensions:** [if any pages contradict each other, summarize the disagreement]

**Gaps:** [what's missing from the wiki that would improve this answer, if anything]
```

### Step 4: Offer to persist

If the answer crosses ≥3 pages and reveals a connection not already captured as a synthesis page, suggest promoting it:

> "Esta respuesta conecta [[a]], [[b]] y [[c]] de una forma que no está documentada. ¿Quieres ejecutar `/wiki:synthesize` para persistirla como página?"

If the user accepts, invoke the `synthesize` skill (no argument — it picks up the just-emitted answer + cited pages from session context). Do NOT inline the synthesis here; delegate to that skill so logging and frontmatter are correct.

For trivial queries (single page, simple lookup), skip this step.

### Step 5: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | query | <question>\nPages consulted: <comma-separated list>\nResult saved: yes (<page-name>.md) | no\n"
```
