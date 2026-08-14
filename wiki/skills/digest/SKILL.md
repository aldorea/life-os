---
name: digest
description: Use when reviewing wiki activity over a time period. Triggered by `/wiki:digest [day|week|month]`. Reads log.md and produces a formatted summary of ingests, queries, lint passes, and pages created/updated. Defaults to 'week' if no argument given.
---

# wiki:digest

Activity summary for the personal wiki from `log.md`.

## Vault Paths
- Log: `08 Resources/wiki/log.md`
- Drafts: `08 Resources/wiki/.drafts/`

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
- **Raw notes created**: sum of `Raw notes created:` values across all ingest entries (excluding `none`)
- **Pages created**: sum of `Pages created:` values across all ingest entries (excluding `none`)
- **Drafts created**: sum of `Drafts created:` values across all ingest entries (excluding `none`)
- **Pages updated**: sum of `Pages updated:` values across all ingest entries (excluding `none`)
- **Queries**: total entries with `| query |`
- **Synthesize**: total entries with `| synthesize |`
- **Approves**: total entries with `| approve |` (broken down by `Action: published` vs `discarded`)
- **Lint passes**: total entries with `| lint |`
- **Audits**: total entries with `| audit |` — structural audits from `/wiki:audit`. Each carries `Score: NN/100`, per-dimension percentages, and stage counts. Report the **latest score and its delta** versus the previous audit entry in the log (which may fall outside the period — scan back for it). The trend is the signal, not the count.
- **Maintain**: total entries with `| maintain |` — bulk operations on existing pages (stub fills, refactors, status sweeps). Carries `Pages updated: N`.
- **Corrections**: total entries with `| correction |` — rollbacks or fixes to previous wiki operations. Carries `Source`, `Action`, `Reason`. Important signal — should be highlighted, not buried.
- **Daily closes**: total entries with `| daily |` — these are bridge entries appended by the `close` ritual. Each carries `Foco`, `Completadas`, `Energia`, `Note`. The digest uses them to build a per-day timeline of the period.
- **Inbox captures**: total entries with `| inbox |` — items captured to `02 Inbox.md` via dump or telegram (one log line per batch, not per item). Carries `Items: N`.
- **Process-inbox runs**: total entries with `| process-inbox |` — batches of clarification (raw → tasks). Carries `Items processed`, `Tasks created`.
- **Training captures**: total entries with `| training |` — telegram-routed gym sessions. Carries `Date`, `Lines`.

### Step 4c: Compute net deltas

Don't just sum events — compute the **net result** after corrections:

- For each `| correction |` entry, identify the operation it reverts (by source URL or page name in the `Source:`/`Action:` fields). Subtract that from the gross counts above.
- Example: ingest creates page X, later correction reverts X to raw note. Net: 0 pages created, 1 raw note created.
- Net deltas to compute:
  - `Net pages added` = (pages created) − (pages reverted by corrections)
  - `Net raw notes added` = (raw notes created) + (raw notes that absorbed reverted pages)
  - `Net pages updated` = pages updated minus any reverted updates

Net deltas are the headline numbers in the report. Gross counts go in the detail section.

### Step 4b: Pending drafts (snapshot, not from log)

List current drafts in `.drafts/`:
```bash
ls "/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/08 Resources/wiki/.drafts/"
```

For each draft, read its frontmatter `updated`. Compute age in days.

### Step 5: Format report

The report has **fixed top sections** (always render) and **collapsible bottom sections** (render only if non-zero, otherwise render as a single `sin actividad` line). Headline = net result + highlights, not gross event counts.

```
📅 [Period label]

═══ Highlights ═══
<2-3 sentences synthesizing the arc of the period — what changed, what was learned, what shipped. Group related events into a single thread, don't list them as separate items. Skip this section entirely if the period was purely operational with no narrative.>

═══ Net result ═══
  Pages added (net):       N
  Raw notes added (net):   N
  Pages updated (net):     N
  Drafts pending:          N (M older than 14 days ⚠️)

═══ Detail ═══
Knowledge (wiki)
  Ingests:           N  (X URLs, Y PDFs, Z YouTube, W text)
  Raw notes:         [note1, ...]
  Pages published:   [page1, ...]
  Pages updated:     [page1, ...]
  Synthesize:        N pages
  Queries:           N
  Approves:          N (X published, Y discarded)
  Lint:              N pass(es)
  Audit:             NN/100 (<banda>, <+/-N vs anterior>)
  Maintain:          N (describe briefly if any)
  Corrections:       N (describe briefly — what was reverted and why)

Daily closes: N días cerrados
  - YYYY-MM-DD — Foco: <foco> — N tasks, <color> — "<note>"
  - ...

Tasks (GTD): <single line if all-zero, e.g. "sin actividad">
  Inbox captures:    N (M items)
  Process-inbox:     N batches, X tasks created

Health: <single line if zero>
  Training captures: N sesiones

Drafts pendientes:
  - <name> (Nd) — <source>
  - ...
```

**Rendering rules:**

- **Highlights section is the headline.** If today's most important fact is "the workflow was updated after a correction", that goes here — not buried in `Corrections: 1`.
- **Net result trumps gross counts.** A page created and then reverted is `Net pages added: 0`, not `Pages created: 1`. Show the net up top; gross counts are detail.
- **Collapse zero sections.** Sections under `═══ Detail ═══` that have all zeros render as `<section>: sin actividad` on a single line. Don't pad reports with `0` rows.
- **Drafts >14 days old** flagged as `⚠️`.
- **Daily-close gap warning** ONLY for past periods (week/month, or yesterday). For `--day` referring to today, do NOT flag missing close — the day isn't over yet. Determine "past" by comparing the period's last date to today's date.
- **Don't mix the report with ad-hoc commentary outside the spec.** If you have a comment, put it in `Highlights`. Don't append a free-form "Notas del día" section after the rendered block.
- **Group log entries into arcs.** Multiple log entries sharing a source URL or page name within the same day are likely one story (e.g., ingest + correction + skill update). Render them as one item in Highlights, not as N independent counts.

If no activity in the period:
> "No wiki activity in the last [period]."

### Step 6: No log entry needed

`digest` is read-only — do not append to log.md.
