---
name: synthesize
description: Use when promoting a `/wiki:query` answer (or an ad-hoc topic) into a persistent synthesis page. Triggered by `/wiki:synthesize [topic]`. Without an argument, promotes the most recent query in this session. Creates a `type: synthesis` page in `08 Resources/wiki/pages/`, updates index.md and log.md. This is what closes the loop between query (ephemeral) and wiki (persistent).
---

# wiki:synthesize

Promotes synthesized knowledge into a persistent wiki page. The verb that distinguishes a wiki that **accumulates** from a RAG that only retrieves.

## Vault Paths
- Pages: `08 Resources/wiki/pages/`
- Index: `08 Resources/wiki/index.md`
- Log: `08 Resources/wiki/log.md`
- Schema: `08 Resources/wiki/WIKI.md`

## When to use this skill

- After a `/wiki:query` whose answer crosses ≥3 pages and exposes a connection not yet documented.
- After a `/wiki:query` that the user found useful enough to want to keep ("save this", "guarda esto", "make it a page").
- Ad-hoc: user gives a topic and wants the LLM to consolidate what already exists in the wiki into one synthesis page.

**Do NOT use this skill for:**
- A single-source ingest → that's `/wiki:ingest`.
- A new concept with no existing pages → that's `/wiki:ingest` with inline text.
- Raw thoughts → those go to capture / inbox, not the wiki.

## Step 1: Determine input

Three input modes:

1. **No argument** — promote the most recent `/wiki:query` answer in this session. Read the last answer + the pages it cited from your conversation context.
2. **Topic argument** (`/wiki:synthesize <topic>`) — first run an internal query for `<topic>`: read `index.md`, identify candidate pages, read them. Then synthesize.
3. **Explicit answer to persist** — the user pasted text. Use it as-is, but still identify which existing pages support each claim.

If mode is unclear, ask:
> "¿Quieres que promueva la última respuesta de query, o sintetizo un tema nuevo desde el wiki?"

## Step 2: Verify there is something to synthesize

A synthesis page must:
- Cite ≥2 existing wiki pages, OR
- Combine ≥1 existing page with new external knowledge (must be flagged as such).

If neither holds: stop and tell the user `/wiki:ingest` is the right verb instead. Do not create a synthesis with a single source — that's just an ingest.

## Step 3: Check for existing synthesis

Read `index.md` and look for an existing `type: synthesis` page that already covers this topic.

If found:
> "Ya existe [[<page>]] sobre este tema. ¿Quiero actualizarla en lugar de crear una nueva?"

Default to **update** rather than create — synthesis pages are meant to accumulate.

## Step 4: Write the page

**Naming**: kebab-case. Synthesis page names should describe the *connection* or *domain*, not just a single concept. Good: `event-driven-architecture-tradeoffs.md`, `rag-vs-fine-tuning-when.md`. Bad: `kafka.md` (that's an entity page).

**Frontmatter**:
```yaml
---
type: synthesis
status: published
confidence: <inherit from weakest cited page, or set explicitly>
updated: YYYY-MM-DD
sources:
  - "[[page-a]]"
  - "[[page-b]]"
  - "derived from session YYYY-MM-DD"
tags: [<categories>, synthesis]
supports: ["[[page-a]]", "[[page-b]]"]   # synthesis usually supports its inputs
---
```

**Confidence rule**: a synthesis is no more confident than its weakest input. If any cited page is `confidence: low`, the synthesis defaults to `low` unless the user overrides.

**Body** — same structure as any wiki page:
1. `## Summary` — 2–4 sentences stating the connection or conclusion.
2. `## Key Concepts / How It Works` — the synthesis itself, citing `[[wikilinks]]` to inputs at every claim.
3. `## Tradeoffs / When to Use` — when this synthesis applies and when it doesn't.
4. `## See Also` — wikilinks to all cited pages plus related concepts.

**Critical rule**: every non-trivial claim must cite a source page. If a claim cannot be cited from existing pages, either: (a) flag it as external knowledge with `> [!note] External: ...`, or (b) drop it.

Write with:
```
obsidian create path="08 Resources/wiki/pages/<kebab-name>.md" content="<full content>" overwrite
```

## Step 5: Update cited pages

For each page cited as a source, optionally add the new synthesis to its `## See Also` section as `[[<new-synthesis>]]`. This makes the synthesis discoverable from its inputs.

Skip pages that already have ≥5 entries in See Also — keep cross-link density manageable.

## Step 6: Update index.md

Add an entry under the most relevant category (or a `## Synthesis` section if you want to keep them grouped):

```
- [<name>](pages/<name>.md) — synthesis: <one-line description of the connection>
```

If updating an existing synthesis, just bump the description if it changed.

## Step 7: Append to log.md

```
obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | synthesize | <topic>\nPage: <name>.md (created | updated)\nFrom: [[page-a]], [[page-b]], [[page-c]]\nTrigger: query | topic | explicit\n"
```

## Step 8: Report

Tell the user:
- Page created or updated (with name)
- Pages it cites (as `[[wikilinks]]`)
- Confidence level (and why)
- Any external claims flagged
- Suggestion to run `/wiki:lint` if the synthesis introduced new dangling links

## Integration with /wiki:query

After answering a query that crosses ≥3 pages and reveals a non-trivial connection, `/wiki:query` should suggest:
> "Esta respuesta conecta [[a]], [[b]] y [[c]] de una forma que no está documentada como página. ¿Quieres ejecutar `/wiki:synthesize` para persistirla?"

The user accepts → invoke this skill in mode 1 (no argument).
