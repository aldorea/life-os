---
name: migrate-vault
description: Use when migrating the vault to the new PARA-inspired structure. One-time use. Use when user says "migrate vault", "migra vault", "restructure vault", "reestructura vault".
---

# migrate-vault

## Step 0 -- Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

One-time migration skill that restructures the existing Obsidian vault from its old folder layout to the new PARA-inspired numbered convention. Creates a git backup before any changes, presents the migration plan for user approval, executes the moves/renames, adds missing frontmatter to existing notes, and updates config files.

## Process

### Step 1 -- Git safety snapshot

Create a backup commit so the user can roll back if anything goes wrong.

1. Run `git -C VAULT add -A`
2. Run `git -C VAULT commit -m "backup: pre-migration snapshot"`
3. If the commit fails (nothing to commit), that's fine -- continue anyway. The vault is already clean.

If `git` is not initialized in the vault (`git -C VAULT status` fails):
> "No se encontro repositorio git en el vault. Se recomienda inicializar git antes de migrar para tener un punto de restauracion. Continuar sin backup? (s/n)"

Only proceed if user confirms.

### Step 2 -- Scan current vault state

Read `config.yaml` `structure` section to discover the current paths. For each path in the migration map below, check if the source folder/file exists in `VAULT/`.

Also scan the vault root for any folders/files NOT covered by the migration map (to avoid orphaning content).

### Step 3 -- Build migration plan

Using the migration map, build a plan table. Only include rows where the source exists (graceful degradation -- skip what's missing).

**Migration Map:**

| Action | From (old path) | To (new path) | Notes |
|--------|-----------------|---------------|-------|
| Rename | `02 Inbox.md` | `00 Inbox.md` | Top of sidebar for quick access |
| Rename | `05 Projects/` | `02 Projects/` | Move up -- daily use |
| Rename | `09 People/` | `05 People/` | Move up -- frequent access |
| Move | `08 Resources/meetings/` | `07 Meetings/` | Promote out of Resources |
| Move | `08 Resources/knowledge/` | `08 Knowledge/` | Promote out of Resources |
| Move | `08 Resources/content-drafts/` | `10 Content/` | Promote out of Resources |
| Move | `08 Resources/calendar-cache.md` | `.cache/calendar-cache.md` | Hide non-user-facing cache |
| Move | `08 Resources/granola-processed.md` | `.cache/granola-processed.md` | Hide non-user-facing cache |
| Move | `08 Resources/claude-memory/` | `.cache/claude-memory/` | Hide non-user-facing cache |
| Rename | `10 Health/` | `09 Health/` | Renumber after People moved up |
| Create | _(new)_ | `11 Archive/` | PARA Archive folder |
| Create | _(new)_ | `.cache/` | Cache directory for non-user-facing files |

**Unchanged paths** (already at correct position -- skip these):

| Path | Reason |
|------|--------|
| `01 Backlog.md` | Already at correct number |
| `03 Daily/` | Already at correct number |
| `04 Weekly/` | Already at correct number |
| `06 Jira/` | Already at correct number |
| `config/` | Unnumbered, stays in place |

For each row in the migration map:
- If source exists: mark as "listo" (ready)
- If source doesn't exist: mark as "omitido" (skipped) and note why
- Flag any unexpected items inside old folders that won't be moved automatically

### Step 4 -- Present plan and wait for confirmation

Show the migration table to the user with status for each row. Show a summary count:

> "Plan de migracion:"
> [table]
>
> "Resumen: X carpetas a renombrar, Y carpetas a mover, Z carpetas a crear, W archivos a mover."
>
> "Elementos omitidos (fuente no encontrada): [list, if any]"
>
> "Aplico la migracion? (s/n)"

**Only proceed after explicit user confirmation.** If the user says no, stop here. Do not apply any changes.

### Step 5 -- Execute migration

Execute each row in order. For each action:

1. Create target parent directory if it doesn't exist: `mkdir -p VAULT/[parent-dir]`
2. Move/rename using `mv VAULT/[old-path] VAULT/[new-path]`
   - For renames: `mv "VAULT/02 Inbox.md" "VAULT/00 Inbox.md"`
   - For moves: `mv "VAULT/08 Resources/meetings" "VAULT/07 Meetings"`
   - For creates: `mkdir -p "VAULT/11 Archive"` and `mkdir -p "VAULT/.cache"`

**Important execution order:**
1. First, create `.cache/` directory
2. Move files OUT of `08 Resources/` to their new locations (cache files, meetings, knowledge, content-drafts)
3. After all moves from `08 Resources/`, check if it's empty. If empty, remove it: `rmdir "VAULT/08 Resources"`. If not empty, warn user about remaining contents.
4. Then do folder renames (Inbox, Projects, People, Health)
5. Finally, create new folders (Archive)

**Critical:** Only move FOLDERS, never rename individual files inside them. Obsidian resolves wikilinks by filename -- renaming files breaks `[[wikilinks]]`. Moving files to new folders is safe.

### Step 6 -- Add frontmatter to existing notes

Scan all `.md` files in the newly moved/created locations. For each file:

**If the file has NO YAML frontmatter** (no `---` block at the top), add appropriate frontmatter based on its new location:

- **Files in `05 People/`** -- person frontmatter:
  ```yaml
  ---
  tags: person
  name: [extracted from filename, e.g. "Juan Garcia" from "Juan Garcia.md"]
  created: [today's date, YYYY-MM-DD]
  last_interaction: [today's date, YYYY-MM-DD]
  ---
  ```

- **Files in `07 Meetings/`** -- meeting frontmatter:
  ```yaml
  ---
  tags: meeting
  date: [extracted from filename if pattern "YYYY-MM-DD - Title.md", otherwise today]
  attendees: []
  source: manual
  ---
  ```

- **Files in `08 Knowledge/`** -- knowledge frontmatter:
  ```yaml
  ---
  tags: knowledge
  topic: [extracted from filename, e.g. "Machine Learning" from "Machine Learning.md"]
  last_updated: [today's date, YYYY-MM-DD]
  entries: 1
  maturity: seed
  ---
  ```

**If the file ALREADY has YAML frontmatter**, check for missing required fields per the schema above and add them with sensible defaults. Do not overwrite existing values.

Before writing any frontmatter changes, present the additions to the user for review:

> "Notas que necesitan frontmatter:"
>
> | Archivo | Accion | Campos a agregar |
> |---------|--------|-----------------|
> | 05 People/Juan.md | Agregar frontmatter | tags, name, created, last_interaction |
> | 07 Meetings/old-meeting.md | Agregar campos faltantes | source |
>
> "Aplico los cambios de frontmatter? (s/n)"

Only apply after user confirmation.

### Step 7 -- Update config files

Update both `config.yaml` (user's file) and `config.example.yaml` (git-tracked) with the new structure paths.

**New structure values:**

```yaml
structure:
  inbox: "00 Inbox.md"
  backlog: "01 Backlog.md"
  projects: "02 Projects"
  daily_notes: "03 Daily"
  weekly_notes: "04 Weekly"
  people: "05 People"
  jira_notes: "06 Jira"
  meetings: "07 Meetings"
  knowledge: "08 Knowledge"
  content_ideas: "08 Knowledge/content-ideas.md"
  training_log: "09 Health/training-log.md"
  menus: "09 Health/menus"
  content_drafts: "10 Content"
  archive: "11 Archive"
  cache: ".cache"
  calendar_cache: ".cache/calendar-cache.md"
  granola_processed: ".cache/granola-processed.md"
  claude_memory: ".cache/claude-memory"
  goals: "config/goals.yaml"
  constraints: "config/constraints.yaml"
  voice: "config/voice.md"
  connectors_config: "config/connectors.yaml"
```

Also update the `tags` section in both files to include GTD contexts and status tags:

```yaml
tags:
  contexts: ["#home", "#office", "#calls", "#computer"]
  projects: ["#miportal", "#sherpa", "#previene", "#orbitant", "#lifeos", "#salud"]
  priority: ["#urgente", "#importante"]
  actionability: ["#next", "#esperando", "#someday"]
  status: ["#active", "#stalled", "#completed", "#archived"]
```

**If `config.yaml` doesn't exist:** Only update `config.example.yaml`. Warn the user:
> "No se encontro config.yaml. Solo se actualizo config.example.yaml. Copia config.example.yaml a config.yaml y rellena tus valores."

### Step 8 -- Verify migration

Read the updated `config.yaml` (or `config.example.yaml` if config.yaml doesn't exist). For each path in the `structure` section, check if it resolves to an existing file or folder in `VAULT/`.

Report results:

> "Verificacion de migracion:"
>
> | Ruta config | Path | Existe |
> |-------------|------|--------|
> | structure.inbox | 00 Inbox.md | Si/No |
> | structure.projects | 02 Projects/ | Si/No |
> | ... | ... | ... |
>
> "Rutas que no resuelven: [list, if any]"

If any critical paths don't resolve, warn the user but don't roll back (the git backup is available for that).

### Step 9 -- Git commit result

Commit the migrated vault state:

1. Run `git -C VAULT add -A`
2. Run `git -C VAULT commit -m "feat(vault): migrate to PARA-inspired structure"`

If git is not initialized (detected in Step 1), skip this step.

### Step 10 -- Report

Show a summary of everything that was done:

> "Migracion completada:"
> - X carpetas renombradas
> - Y carpetas movidas
> - Z carpetas creadas
> - W notas actualizadas con frontmatter
> - Config files actualizados: [config.yaml, config.example.yaml]
>
> "Para ver los commits de backup y migracion:"
> `git log --oneline -5`
>
> "Para revertir la migracion:"
> `git revert HEAD`
>
> "Este skill (/migrate-vault) es de un solo uso y puede ser eliminado despues de la migracion exitosa."

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| Source folder doesn't exist | Skip that move/rename, note in report as "omitido" |
| `08 Resources/` doesn't exist | Skip all moves from Resources, note in report |
| File already has frontmatter | Check for missing required fields only, don't overwrite existing values |
| `config.yaml` doesn't exist | Update only `config.example.yaml`, warn user to create config.yaml |
| Git not initialized in vault | Skip git steps (Step 1 and Step 9), warn user to backup manually before proceeding |
| Empty folder after moves | Remove with `rmdir` (only removes if truly empty) |
| Unexpected files in old folders | List them in Step 4 plan, let user decide |

## Important Rules

- **NEVER auto-migrate** -- always present the plan and wait for explicit user confirmation before any changes
- **NEVER rename files** (only rename/move folders) -- Obsidian wikilinks resolve by filename, renaming files breaks `[[links]]`
- **NEVER delete source folders** until confirmed empty (use `rmdir` which fails on non-empty dirs)
- **Spanish language** for all user-facing text (prompts, reports, table headers)
- **Config-driven paths** -- read old paths from `config.yaml` structure section, don't hardcode
- **This is a one-time skill** -- can be safely removed from `skills/` after successful migration
- **Two confirmations required** -- one for the migration plan (Step 4), one for frontmatter additions (Step 6)
