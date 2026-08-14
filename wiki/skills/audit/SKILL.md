---
name: audit
description: Use when auditing the health and structure of the second brain as a whole. Triggered by `/wiki:audit [--quick]`. Acts as editor-in-chief over the vault: evaluates refinery flow (raw → atoms → threads), atomicity, link connectivity, source traceability, contradictions (FRICTION), core themes, and knowledge gaps. Produces a scored Traceability report with a prioritized backlog. Read-only.
---

# wiki:audit

Structural audit of the second brain. Acts as **editor-in-chief**, not as a proofreader:
it judges whether knowledge is *flowing* through refinement stages and whether the
network is *trustworthy*, then reports a score and a prioritized backlog.

**Read-only.** Never edits, moves, merges, or creates pages. Every fix is a
recommendation for the user to accept. The only write is the log entry in Step 8.

## Boundary vs `wiki:lint`

These two do not overlap. Do not re-run lint's work here.

| | `wiki:lint` | `wiki:audit` (this skill) |
|---|---|---|
| Scope | `wiki/` only | whole vault: `raw/`, `wiki/`, inbox |
| Nature | mechanical, per-page | structural, cross-cutting |
| Acts | fixes what it can | reports only |
| Asks | "is this page well-formed?" | "is this system still refining knowledge?" |

Lint owns: index orphans, dead wikilinks, stale `updated` dates, declared
`contradicts:` targets, deprecated cleanup. **If the audit trips over any of
those, do not fix and do not itemize them — emit one line: "N incidencias
mecánicas detectadas → corre `/wiki:lint`."**

Audit owns the six dimensions below.

## Vault Paths

Real layout (`raw/` and `wiki/` sit at **vault root**, not under `08 Resources/`):

- Atoms + threads: `wiki/pages/`
- Index: `wiki/index.md`
- Log: `wiki/log.md`
- Schema: `wiki/WIKI.md`
- Drafts (not citable, not indexed): `wiki/.drafts/`
- Raw notes (stage 0): `raw/notes/`
- Raw sources: `raw/pdfs/`, `raw/transcripts/`
- Capture entry point: `02 Inbox.md`

```bash
VAULT="/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault"
```

> Several sibling skills still reference a legacy `08 Resources/wiki/` path that
> does not exist in the vault. If a path here resolves empty, verify with
> `ls "$VAULT"` before reporting an empty stage — an empty listing is a path bug,
> not a health finding.

## Prime directive — no invention

Every finding must name the file(s) it came from. If the evidence is not in the
vault, the finding does not exist.

- Never infer a page's content from its filename — read it.
- Never assert a contradiction without quoting both sides.
- Never estimate a metric you did not compute. Report `no verificable` instead.
- If a stage directory is missing entirely, report it as **missing**, not as zero.

## Arguments

- no argument → full audit (all six dimensions)
- `--quick` → dimensions 1–3 only (pure metrics, no page-by-page reading).
  Use when the user wants the score without the semantic passes.

## Step 0: Confirm schema and shape

```
obsidian read path="wiki/WIKI.md"
```

Read the schema first — frontmatter fields and `type` values come from there, not
from memory. Then take the stage census:

```bash
for d in "raw/notes" "raw/pdfs" "raw/transcripts" "wiki/pages" "wiki/.drafts"; do
  if [ -d "$VAULT/$d" ]; then
    printf '%-18s %s\n' "$d" "$(find "$VAULT/$d" -name '*.md' -o -name '*.pdf' | wc -l)"
  else
    printf '%-18s MISSING\n' "$d"
  fi
done
```

## Dimension 1: The Refinery

Knowledge should move through **maturity stages**, not sit in topic drawers.

Map the stages onto this vault:

| Refinery stage | This vault | Meaning |
|---|---|---|
| `0-raw` | `raw/notes/`, `raw/pdfs/`, `raw/transcripts/`, `02 Inbox.md` | captured, unrefined |
| `1-pending` | `wiki/.drafts/` | awaiting approval |
| `2-atoms` | `wiki/pages/` with `type: concept` / `entity` / `comparison` | one permanent idea each |
| `3-threads` | `wiki/pages/` with `type: synthesis` | integrated lines of thought |

Type distribution:

```bash
sed -n '/^---$/,/^---$/p' "$VAULT/wiki/pages/"*.md | grep -E '^type:' \
  | sed 's/^type:[[:space:]]*//' | sort | uniq -c | sort -rn
```

Raw backlog age — is stage 0 draining?

```bash
for f in "$VAULT/raw/notes/"*.md; do
  [ -e "$f" ] || continue
  cap=$(grep -m1 -E '^(captured|created|updated):' "$f" | sed 's/^[a-z]*:[[:space:]]*//')
  echo "${cap:-unknown} $(basename "$f")"
done | sort | head -20
```

**Flags:**

- **Stalled refinery** — `raw/notes/` older than 60 days with no matching page.
  Cross-check by slug: a raw note whose topic never appears in `wiki/index.md`
  has not been promoted.
- **Inverted pyramid** — `type: synthesis` count ≥ atom count. Threads should be
  the scarce layer; too many means synthesis without atomic grounding.
- **No threads** — zero `type: synthesis` pages while atoms > 15. Knowledge is
  accumulating but never integrating. This is the most common failure mode.
- **Drawer folders** — a directory or tag holding >10 notes with no promotion
  path. Check tag concentration:

```bash
for f in "$VAULT/wiki/pages/"*.md; do
  sed -n '/^---[[:space:]]*$/,/^---[[:space:]]*$/p' "$f" \
    | sed -n 's/^tags:[[:space:]]*\[\(.*\)\]/\1/p;s/^[[:space:]]*-[[:space:]]*\([a-z][a-z0-9-]*\)$/\1/p'
done | tr ',' '\n' | tr -d ' ' | grep -vE '^-*$' | sort | uniq -c | sort -rn | head -15
```

Parse tags from **frontmatter only** — handles both `tags: [a, b]` and YAML list
form. Do not `grep -A5 '^tags:'`: it bleeds body text and the `---` delimiter into
the counts.

A tag on >30% of pages is a drawer, not a tag — it carries no discriminating
information. Report it as such.

## Dimension 2: Atomicity and Connectivity

### 2a. Atomicity

One concept per page. Two proxies for violation — length and heading spread:

```bash
wc -w "$VAULT/wiki/pages/"*.md | sort -rn | sed -n '2,16p'
```

```bash
for f in "$VAULT/wiki/pages/"*.md; do
  echo "$(grep -c '^## ' "$f") $(basename "$f" .md)"
done | sort -rn | head -12
```

The schema prescribes ~5 `##` sections. A page with **>800 words or >8 `##`
headings** is a candidate for splitting — but *confirm by reading it* before
reporting. A long page on a genuinely single concept is fine; flag only pages
that carry two or more independent ideas, and name the ideas you would split into.

### 2b. Connectivity

Every atom should link to **at least two** others, and be reachable from at least one.

Outgoing links per page:

```bash
for f in "$VAULT/wiki/pages/"*.md; do
  echo "$(grep -o '\[\[[^]]*\]\]' "$f" | sort -u | wc -l) $(basename "$f" .md)"
done | sort -n | head -20
```

Incoming links per page:

```bash
for f in "$VAULT/wiki/pages/"*.md; do
  name=$(basename "$f" .md)
  n=$(grep -rlE "\[\[$name(\|[^]]*)?\]\]" "$VAULT/wiki/pages/" 2>/dev/null \
      | grep -v "/$name\.md$" | wc -l)
  echo "$n $name"
done | sort -n | head -20
```

**Flags:**

- **True orphan** — 0 incoming **and** 0 outgoing links. Isolated from the graph.
  (Distinct from lint's orphan, which means "absent from `index.md`".)
- **Dead end** — has incoming links, 0 outgoing. Consumes but never routes.
- **Under-linked** — fewer than 2 outgoing links.
- **Hub** — top 5 by incoming links. Not a defect: these are the load-bearing
  pages. Name them, because they are where the next thread should be written.

For each orphan, propose the **specific** pages it should link to, chosen by
shared vocabulary — never a generic "add more links".

## Dimension 3: Traceability and Truth

A brain you cannot trace is a brain you cannot trust.

`sources:` is valid in two forms — inline (`sources: <url>`) or a YAML list on the
following lines — and the check must accept both. Parse the frontmatter block
properly:

```bash
for f in "$VAULT/wiki/pages/"*.md; do
  awk -v name="$(basename "$f" .md)" '
    NR==1 && /^---[[:space:]]*$/ { infm=1; next }
    infm && /^---[[:space:]]*$/  { exit }
    !infm { exit }
    {
      if ($0 ~ /^sources:/) {
        found=1; insrc=1
        val=$0; sub(/^sources:[[:space:]]*/,"",val); gsub(/[[:space:]]/,"",val)
        if (val != "" && val != "[]") inline=1
        next
      }
      if (insrc) {
        if ($0 ~ /^[[:space:]]+-[[:space:]]*[^[:space:]]/) { listitem=1; next }
        if ($0 ~ /^[^[:space:]]/) { insrc=0 }
      }
    }
    END {
      if (!found)                    print "NO-SOURCE (falta clave): " name
      else if (!inline && !listitem) print "NO-SOURCE (vacía): " name
    }
  ' "$f"
done
```

Do **not** shortcut this with `grep -A3 '^sources:'` — the trailing context picks
up the next frontmatter key, so nothing ever tests as empty and every page reads
as sourced. A traceability check that silently passes everything is worse than no
check. If a page's frontmatter is malformed enough that this parser is ambiguous,
**read the page** and decide by hand.

Then verify the citations actually resolve:

- `derived from session YYYY-MM-DD` → valid (self-authored, per the page contract).
- Local path → confirm the file exists in `raw/pdfs/` or `raw/transcripts/`.
- URL → accept as-is; do not fetch.
- Anything else → **unresolvable source**, worse than no source, because it looks
  cited. Report these first.

Also check `confidence:` against sourcing: `confidence: high` on a page with no
source or a single weak source is a **confidence inflation** finding.

## Dimension 4: Friction and Contradictions

*(skipped under `--quick`)*

Where has the user's thinking changed without the wiki catching up?

1. Read the 10 most recently `updated` pages plus the 5 hubs from 2b.
2. Compare their claims against older pages on the same subject.
3. Look for: opposite recommendations on the same tool or practice; different
   version or price facts; a stated preference that a later page abandons;
   advice that contradicts `08 Resources/CRITICAL_FACTS.md`.

```bash
grep -l -iE '<term-1>|<term-2>' "$VAULT/wiki/pages/"*.md
```

Report under a literal `[FRICTION]` heading. Each item **must** quote both sides
with their file paths, and classify the friction:

- **Evolution** — thinking genuinely changed. Recommend: keep both, add
  `## Timeline` to the newer page, mark the older `status: deprecated` with a
  `supersedes` pointer.
- **Contradiction** — both claim to be current; one is wrong. Recommend: user
  decides, then declare `contradicts:` in frontmatter.
- **Ambiguity** — different scopes read as conflict. Recommend: tighten wording.

Never resolve friction autonomously. Surfacing it *is* the deliverable.

## Dimension 5: Core Themes and Gaps

*(skipped under `--quick`)*

**Repetition → core themes.** What has been written about ≥3 times, probably
without noticing? These are the real centers of gravity.

```bash
grep -ohE '\[\[[^]|]+' "$VAULT/wiki/pages/"*.md | sed 's/^\[\[//' \
  | sort | uniq -c | sort -rn | head -20
```

Cross-reference with the raw layer, so themes forming in `raw/notes/` count too:

```bash
grep -rohiE '[A-Z][a-zA-Z0-9-]{3,}' "$VAULT/raw/notes/" 2>/dev/null \
  | sort | uniq -c | sort -rn | head -25
```

A theme appearing across ≥3 pages with **no dedicated page or thread** is the
highest-value writing task in the vault. Say so explicitly.

**Gaps.** Concepts referenced often but never developed:

```bash
grep -ohE '\[\[[^]|]+' "$VAULT/wiki/pages/"*.md | sed 's/^\[\[//' | sort | uniq -c \
  | sort -rn | while read -r n name; do
      [ "$n" -ge 3 ] && [ ! -f "$VAULT/wiki/pages/$name.md" ] && echo "GAP($n): $name"
    done
```

Also flag **stubs**: pages that exist but carry a `## Summary` and nothing more —
referenced concepts with the appearance of coverage.

## Dimension 6: Traceability Score

Compute each sub-metric from the numbers gathered above. **Do not estimate** — if
a metric could not be computed, mark it `n/a` and renormalize the weights over
the metrics that were computed. State that you renormalized.

| # | Sub-metric | Weight | Formula |
|---|---|---|---|
| 1 | Sourcing | 30 | % of `wiki/pages/` with a resolvable source |
| 2 | Connectivity | 25 | % of pages with ≥2 outgoing **and** ≥1 incoming link |
| 3 | Atomicity | 20 | % of pages ≤800 words and ≤8 `##` headings |
| 4 | Refinery flow | 15 | % of `raw/notes/` newer than 60 days **or** promoted |
| 5 | Integration | 10 | `min(1, synthesis_pages / (atoms / 8)) × 100` |

`Score = Σ (sub-metric % × weight) / 100`, rounded to the integer.

**Small-vault guard:** if `atoms < 8`, report Integration as `n/a` and renormalize
the remaining weights over 90. With only a handful of atoms the ratio saturates at
100% on a single synthesis page and flatters the score. Same rule for Refinery flow
when `raw/notes/` is empty or missing.

Bands: **90+ Sólido** · **75–89 Sano** · **60–74 Con deuda** ·
**40–59 Frágil** · **<40 Crítico**

Show the arithmetic. A score without its inputs is not auditable — and this skill
audits.

## Report Format

Output in Spanish. Concrete file names throughout, never generic advice.

```
# Informe de Salud del Segundo Cerebro — YYYY-MM-DD

**Rastreabilidad: NN/100 — <banda>**

  Sourcing        NN%  × 30  =  NN.N
  Conectividad    NN%  × 25  =  NN.N
  Atomicidad      NN%  × 20  =  NN.N
  Flujo refinería NN%  × 15  =  NN.N
  Integración     NN%  × 10  =  NN.N
                              ───────
                              NN/100

═══ 1. Refinería ═══
  0-raw       N notas  (N sin promover >60d)
  1-pending   N drafts
  2-atoms     N páginas  (concept N / entity N / comparison N)
  3-threads   N synthesis

<2-4 sentences: is knowledge flowing or pooling? Name the bottleneck stage.>

Cajones detectados: <tag/carpeta — N notas, sin ruta de promoción>

═══ 2. Atomicidad y conectividad ═══
  Huérfanas reales:  N   → [[page]] (0 in, 0 out)
  Callejones:        N   → [[page]]
  Sub-enlazadas:     N   → [[page]] (N salientes)
  Hubs:                  [[page]] (N entrantes), ...

Candidatas a dividir:
  - [[page]] (N palabras, N secciones) → dividir en «X» y «Y»

═══ 3. Trazabilidad ═══
  Sin fuente:            N  → [[page]], ...
  Fuente irresoluble:    N  → [[page]] cita «<cita>» (no existe)
  Confianza inflada:     N  → [[page]] (confidence: high, 1 fuente débil)

═══ [FRICTION] ═══
  - <Evolución|Contradicción|Ambigüedad>: [[page-a]] dice «<quote>»
    vs [[page-b]] dice «<quote>»
    → <recommended action>

═══ 4. Temas centrales ═══
  - <tema> — N páginas <(sin página dedicada ⚠️)>

═══ 5. Vacíos ═══
  - <concepto> — referenciado N veces, sin página
  - <concepto> — stub (solo Summary)

═══ Backlog priorizado ═══
  P1 (rompe confianza)
    1. <action> — <file> — <why it matters>
  P2 (rompe la red)
    2. ...
  P3 (fortalece)
    3. ...

Mecánicas: N incidencias → corre `/wiki:lint`
```

### Rules for the report

- **Prioritize by damage, not by count.** An unresolvable source outranks twelve
  under-linked pages: one poisons trust, the others only slow retrieval.
- **Backlog items are actions**, each naming its file and its payoff. "Escribe
  `[[x]]` y enlázala desde `[[a]]` y `[[b]]`", not "mejora la conectividad".
- **Cap the backlog at 10 items.** An audit that returns 60 tasks gets ignored;
  that is friction the audit itself created. If more exist, say how many were
  omitted.
- **Empty dimension → one line** (`sin incidencias`). Do not pad.
- **Lead with the single most important structural fact**, even if it is not the
  lowest sub-score.
- Offer to run `/wiki:synthesize <tema>` for the top core theme lacking a thread,
  and `/wiki:approve` if drafts are pending. Do not run them unprompted.

## Step 8: Append to log.md

```
obsidian append path="wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | audit\nScore: NN/100 (<banda>)\nSourcing: NN% | Conectividad: NN% | Atomicidad: NN% | Flujo: NN% | Integración: NN%\nStages: raw N / drafts N / atoms N / threads N\nHuérfanas: N | Sin fuente: N | Friction: N\nTop backlog: <P1 item>\n"
```

The score is logged so it can be **tracked over time** — `/wiki:digest` reads
these entries. A single audit is a snapshot; the trend is the real signal.
