---
name: sync-reminders
description: Syncs Apple Reminders from configured lists into the vault inbox for GTD processing. Read-only -- pulls reminders captured via Siri, Watch, or Phone. Use when user says "sync reminders", "sincroniza recordatorios", "revisa recordatorios", "reminders".
---

# sync-reminders

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Connector that reads Apple Reminders via osascript (AppleScript) and adds unprocessed items to the vault inbox for GTD processing. Read-only: never modifies or completes reminders in the Reminders app. Uses a processed registry to avoid re-adding items on subsequent syncs.

Apple Reminders serves as a quick capture point (Siri, Apple Watch, iPhone) and this skill pulls those captures into the GTD flow.

## Prerequisites

- macOS with Reminders.app (native, always available)
- `osascript` command available (native macOS)
- On first run, macOS may prompt for automation permission — user must grant access to Reminders

## Process

### 1. Read configured Reminders lists

Read `VAULT/{config.structure.connectors_config}` and parse the `reminders` section. Expected fields:

- `reminders.lists`: array of list names to sync (e.g., `["Inbox", "Work", "Personal"]`)
- `reminders.completed`: boolean, whether to include completed reminders (default: `false`)

If no `reminders` section exists in connectors.yaml, warn the user:

```
No hay configuración de Reminders en connectors.yaml. Agrega una sección así:

reminders:
  lists:
    - "Inbox"
    - "Work"
  completed: false
```

Then stop.

### 2. Load processed registry

Read `VAULT/{config.structure.cache}/reminders-processed.md`. If the file doesn't exist, create it with this header:

```markdown
# Reminders Processed Registry
<!-- Tracks synced reminders to avoid duplicates. One entry per line: name|creation_date -->
```

Parse existing entries into a set for O(1) lookup. Each entry is formatted as: `{reminder_name}|{creation_date_ISO}`

### 3. Fetch reminders via osascript

For each configured list name, run:

```bash
osascript -e '
tell application "Reminders"
    set theList to list "{list_name}"
    set output to ""
    repeat with r in (reminders of theList whose completed is false)
        set rName to name of r
        set rYear to year of (creation date of r)
        set rMonth to month of (creation date of r) as integer
        set rDay to day of (creation date of r)
        set rDateStr to (rYear as string) & "-" & text -2 thru -1 of ("0" & rMonth) & "-" & text -2 thru -1 of ("0" & rDay)
        set rBody to ""
        try
            set rBody to body of r
        end try
        set rDue to ""
        try
            set dYear to year of (due date of r)
            set dMonth to month of (due date of r) as integer
            set dDay to day of (due date of r)
            set rDue to (dYear as string) & "-" & text -2 thru -1 of ("0" & dMonth) & "-" & text -2 thru -1 of ("0" & dDay)
        end try
        set output to output & rName & "|" & rDateStr & "|" & rBody & "|" & rDue & linefeed
    end repeat
    return output
end tell'
```

**Important:** The AppleScript constructs ISO dates explicitly (YYYY-MM-DD) to avoid locale-dependent date formatting. Do NOT use `creation date of r as string` which returns locale-dependent format.

If `reminders.completed` is `true`, change `whose completed is false` to remove the filter (fetch all reminders regardless of completion status).

**Error handling:**

- If osascript returns error code 1 with "not authorized": print clear instructions — "Permiso de Reminders necesario. Ve a Ajustes del Sistema > Privacidad y Seguridad > Automatización > Claude/Terminal y activa el acceso a Reminders. Luego reintenta."
- If a specific list is not found: warn `"Lista '{list_name}' no encontrada en Reminders. Saltando."`, continue with other lists.
- If osascript is not available (non-macOS): warn `"Solo disponible en macOS"` and skip entirely.

### 4. Filter unprocessed reminders

For each fetched reminder, build the dedup key: `{reminder_name}|{creation_date_ISO}`. Check against the processed registry set. Only keep reminders NOT in the registry.

### 5. Write inbox items

For each unprocessed reminder, check `VAULT/{config.structure.inbox}` for existing text match (search for the reminder name text in inbox). If NOT already in inbox, append using the following format rules:

**Basic format:**
```markdown
- Reminder ({list_name}) -- {reminder_name} ({YYYY-MM-DD})
```

Where `{YYYY-MM-DD}` is today's date (the sync date, not the reminder creation date).

**With due date:**
```markdown
- Reminder ({list_name}) -- {reminder_name} [vence: {due_date}] ({YYYY-MM-DD})
```

**With body/notes:**
```markdown
- Reminder ({list_name}) -- {reminder_name}: {body_text} ({YYYY-MM-DD})
```

**With both due date and body:**
```markdown
- Reminder ({list_name}) -- {reminder_name}: {body_text} [vence: {due_date}] ({YYYY-MM-DD})
```

### 6. Update processed registry

Append all newly processed reminder dedup keys to `VAULT/{config.structure.cache}/reminders-processed.md`:

```
{reminder_name}|{creation_date_ISO}
```

The processed registry is append-only — never delete entries.

### 7. Report

```
Reminders sync completado:
- {X} recordatorios nuevos encontrados
- {Y} agregados al inbox
- {Z} ya procesados (omitidos)
Listas: {list1}, {list2}, ...
```

### 8. Graceful degradation

| Problem | Behavior |
|---------|----------|
| osascript not available | Skip entirely, warn "Solo disponible en macOS" |
| Reminders permission denied | Print permission instructions (System Settings path), skip |
| Specific list not found | Skip that list, warn, continue with others |
| No reminders in any list | Report "No hay recordatorios pendientes", continue |
| Processed registry missing | Create it, process all available reminders |
| Inbox file missing | Create it with `# Inbox` header, then append |
| Reminder has no body | Omit body from inbox item (name only) |
| Reminder has no due date | Omit due date bracket from inbox item |
| connectors.yaml missing | Warn user to create it, stop |
| connectors.yaml has no reminders section | Show example config, stop |

## Output

- Inbox items appended to `VAULT/{config.structure.inbox}`
- Updated registry at `VAULT/{config.structure.cache}/reminders-processed.md`

## Important Rules

- **Read-only:** NEVER modify, complete, or delete reminders in the Reminders app. This skill only reads.
- Never abort on partial failure — process as many lists as possible
- AppleScript dates must be constructed in ISO format explicitly (not locale-dependent)
- Spanish language for all user-facing output
- The processed registry is append-only (never delete entries)
- Check inbox for existing text before appending (D-11 dedup)
- Lists to sync come from connectors.yaml, never hardcoded (D-08)
