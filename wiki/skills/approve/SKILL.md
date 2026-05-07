---
name: approve
description: Use when reviewing wiki drafts pending publication. Triggered by `/wiki:approve [name]`. Without an argument, lists all pending drafts in `08 Resources/wiki/.drafts/`. With a name, reviews that draft and offers approve / edit / discard. Approving moves the file to `pages/`, flips `status: draft` → `published`, and adds it to `index.md`.
---

# wiki:approve

Reviews drafts produced by `/wiki:ingest` and decides their fate. The human gate between "ingested" and "wiki citizen".

## Vault Paths
- Drafts: `08 Resources/wiki/.drafts/`
- Pages: `08 Resources/wiki/pages/`
- Index: `08 Resources/wiki/index.md`
- Log: `08 Resources/wiki/log.md`

## Two modes

### Mode A: list pending drafts (`/wiki:approve` with no argument)

List every file in `.drafts/`, with: name, age (days since `updated` in frontmatter), source, one-line summary.

```bash
ls "/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/08 Resources/wiki/.drafts/"
```

For each draft, read its frontmatter + Summary section.

Format:
```
Drafts pendientes (N):
1. <name> — <age>d — source: <source-domain> — "<first sentence of Summary>"
2. ...

Run `/wiki:approve <name>` to review one.
```

If `.drafts/` is empty:
> "No hay drafts pendientes. Todos los ingests recientes fueron publicados directamente."

### Mode B: review a specific draft (`/wiki:approve <name>`)

#### Step 1: Read the draft

```
obsidian read path="08 Resources/wiki/.drafts/<name>.md"
```

If not found, list available drafts and stop.

#### Step 2: Show it and offer 3 actions

Display the full draft to the user, then ask:

```
Draft: <name>
Source: <source>
Confidence: <confidence>

¿Qué hago?
  [a] Aprobar tal cual → publica en pages/
  [e] Editar y aprobar → describe cambios, lo modifico, publico
  [d] Descartar → borra el draft
```

Wait for the user's choice. Do not assume.

#### Step 3a: Approve as-is

1. Move file from `.drafts/` to `pages/`:
   ```bash
   mv "<vault>/08 Resources/wiki/.drafts/<name>.md" "<vault>/08 Resources/wiki/pages/<name>.md"
   ```
   Use full vault path:
   `/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault`

2. Flip frontmatter `status: draft` → `status: published` and bump `updated: YYYY-MM-DD`:
   ```
   obsidian property:set name=status value=published path="08 Resources/wiki/pages/<name>.md"
   obsidian property:set name=updated value=<today> path="08 Resources/wiki/pages/<name>.md"
   ```

3. Add to `index.md` under the appropriate category. Read index, append entry, rewrite:
   ```
   - [<name>](pages/<name>.md) — <one-line description from Summary>
   ```

4. Append to log.md:
   ```
   obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | approve | <name>\nAction: published\n"
   ```

#### Step 3b: Edit and approve

1. Ask the user what to change. Common edits:
   - Tighten the Summary
   - Bump `confidence` (with justification)
   - Remove dubious claims
   - Add missing `[[wikilinks]]` to existing pages
   - Add `supersedes:` if it replaces an existing page

2. Apply edits using `obsidian` CLI or by reading + rewriting the file.

3. Then proceed with Step 3a (move + property update + index + log).

4. Log entry: `Action: published (edited)` and a one-line summary of changes.

#### Step 3c: Discard

1. Delete the draft. **Use a safe rm with the full vault path — never a relative path that could resolve to root**:
   ```bash
   rm "/Users/sito/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian Vault/08 Resources/wiki/.drafts/<name>.md"
   ```

2. Append to log.md:
   ```
   obsidian append path="08 Resources/wiki/log.md" content="\n## YYYY-MM-DDTHH:MM | approve | <name>\nAction: discarded\nReason: <user reason or 'not provided'>\n"
   ```

## Safety

- **Never** approve without showing the full draft content to the user first.
- **Never** auto-discard. Discard requires explicit user choice.
- If the user names a page that already exists in `pages/` (collision), stop and ask: should the approval overwrite, merge, or pick a new name?

## Integration

- `/wiki:digest` shows count of pending drafts in its weekly summary.
- `/wiki:lint` warns about drafts older than 14 days.
- `/wiki:query` ignores `.drafts/` — drafts are not citable until approved.
