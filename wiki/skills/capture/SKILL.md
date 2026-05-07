---
name: capture
description: Use when capturing knowledge, links, ideas, observations, or learnings into the wiki. Triggers on "capture", "captura", "guarda esto", "esto es interesante", "save this", or sharing a link/idea worth remembering. Routes through the same pipeline as `sync-telegram` for URL inbox messages — delegates to `/wiki:ingest`.
---

# capture

Thin entrypoint to capture knowledge into the wiki. **Same pipeline as `sync-telegram` for inbox messages**: URLs go through defuddle + `/wiki:ingest`, text goes through `/wiki:ingest` text mode. No parallel logic, no duplicate page-creation flow — one capture path for everything.

## Vault Paths
- Wiki: `08 Resources/wiki/`
- Drafts: `08 Resources/wiki/.drafts/`
- Log: `08 Resources/wiki/log.md`

## Process

### Step 1 — Detect input type

The input can be:
- **URL**: starts with `http://` or `https://`
- **Text**: anything else (idea, observation, tool description, quote)

### Step 2 — Route to `/wiki:ingest`

Both paths delegate to the `/wiki:ingest` skill. This is the SAME path that `sync-telegram` uses for inbox URL messages, so behavior is identical regardless of the source (Telegram bot or interactive `/capture`).

**For URLs:**
Invoke `/wiki:ingest <url>`. The ingest skill will:
1. Run `defuddle parse <url> --md` to extract clean markdown
2. Read `wiki/index.md` to identify pages to update vs create
3. Synthesize content into one or more pages (one concept per page)
4. Apply draft-gate heuristic: low-confidence sources land in `.drafts/`, primary sources in `pages/`
5. Update `wiki/index.md` with new pages
6. Append `| ingest | <source>` entry to `wiki/log.md`

**For text:**
Invoke `/wiki:ingest "<text>"`. The ingest skill will:
1. Skip defuddle (text mode)
2. Synthesize the text into a wiki page (or update existing if topic matches)
3. Same draft-gate, same index update, same log entry

### Step 3 — Done

`capture` does NOT add steps beyond `/wiki:ingest`. Trust the ingest pipeline. If something is missing (e.g., a thematic synthesis page that accumulates entries with timeline), that gap belongs to `/wiki:ingest` or `/wiki:synthesize`, not here.

## Why this skill exists at all (vs. just using `/wiki:ingest`)

- **Muscle memory**: `/capture` is shorter and more natural for ad-hoc inputs.
- **Trigger language**: matches casual phrasing ("save this", "guarda esto") that `/wiki:ingest` does not advertise.
- **Telegram parity**: documents that the `/capture` path and the Telegram-bot path are the same pipeline — no surprise behavior.

If at some point this skill adds no behavior beyond `/wiki:ingest` and the trigger phrases don't matter, it can be deleted with no loss.

## Important rules

- ALWAYS delegate to `/wiki:ingest` — do NOT inline page-creation logic here.
- Spanish UI for any user-facing prompts.
- If `/wiki:ingest` asks for clarification (e.g., topic ambiguity, draft vs publish), let it ask the user — do not pre-empt.
- If the user provides multiple items in one message, invoke `/wiki:ingest` once per item.
