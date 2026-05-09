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

Write a **plain, conversational answer in Spanish (or the user's language)** — not a formal report. The wiki pages already have formal structure; the query is the bridge between knowledge and the user's current decision, not a re-rendering of the pages.

**Default format — adaptive plain prose:**

Pick the sections that fit the question. Common pattern for "qué sé de X":

```
[1-2 sentence framing of what the user actually knows]

**El problema** — [the pain X covers, in plain language]

**La solución** — [what the approaches/tools/patterns do, in plain language]

**Lo que ya estás haciendo bien** — [bullet or prose connecting to existing pages where their own systems already apply this]

**Lo que te falta** — [explicit, actionable gaps; this is where contradictions or missing pages go, woven in as natural language, not as a "Tensions:" header]

**Lo que aún no sabes** — [honesty about single-source notes, untested assumptions, open decisions]

[Closing question inviting to deepen or change topic]
```

Other questions need other shapes. A single-fact lookup is one sentence. A "should I do X?" question is "qué dice tu wiki" + "qué falta para decidir". Pick the shape that respects the question.

**Tone rules:**
- Spanish llano y conversacional. First person where natural.
- **Zero jargon** ("triada que sobrevive", "lente de spec-driven", "fricción no resuelta" — banned). Translate every technical phrase to everyday language.
- **Zero formal meta-headers** (`**Answer:**`, `**From your wiki:**`, `**Tensions:**`, `**Gaps:**`). Their content goes in the prose.
- **Wikilinks inline**, woven into sentences ("ya lo aplicas en [[ops-suite]]"). NOT as a closing bibliography list.
- Short sentences. No nested bullets unless truly enumerating items.
- End with a question that opens the conversation: "¿Quieres profundizar en X o cambiamos de tema?"

**Honor frontmatter signals when citing:**
- Skip pages with `status: deprecated` unless the question is historical. If used, label inline: "(deprecada)".
- Skip pages with `status: stub` — mention as a gap.
- When citing a page with `confidence: low`, flag inline naturally: "lo tienes en [[page]] pero con poca confianza".
- If candidate pages have `contradicts: [[other]]`, surface the tension as plain prose inside "Lo que te falta" or in a dedicated paragraph — **do not** add a `**Tensions:**` header.
- If a page declares `supersedes: [[old]]`, prefer the newer page.

**Grounding:** answer only from the wiki. If you add external knowledge, say so plainly ("la wiki no lo cubre, pero en general...").

**When the formal format is OK:** only if the user explicitly asks for a structured/citation-heavy report (e.g., "dame un informe con citas"). Default is plain.

### Step 4: Offer to persist

If the answer crosses ≥3 pages and reveals a connection not already captured as a synthesis page, suggest promoting it:

> "Esta respuesta conecta [[a]], [[b]] y [[c]] de una forma que no está documentada. ¿Quieres ejecutar `/wiki:synthesize` para persistirla como página?"

If the user accepts, invoke the `synthesize` skill (no argument — it picks up the just-emitted answer + cited pages from session context). Do NOT inline the synthesis here; delegate to that skill so logging and frontmatter are correct.

For trivial queries (single page, simple lookup), skip this step.

### Step 5: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | query | <question>\nPages consulted: <comma-separated list>\nResult saved: yes (<page-name>.md) | no\n"
```
