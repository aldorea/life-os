---
name: dump
description: Use when quickly capturing a thought, task, idea, or anything to the inbox without classification. Use when user says "dump", "anota", "apunta", "inbox", "quick capture", "captura rapida", or provides raw text to save.
---

# dump

## Step 0 -- Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Ultra-fast capture to inbox. No classification, no tags, no questions. Just append raw text with timestamp to Inbox. Under 5 seconds.

The goal is zero friction -- get the thought out of the user's head and into the system as fast as possible. Classification and tagging happen later via `/process-inbox`.

## Vault I/O

Use `obsidian` CLI for all vault writes:
- Append to inbox: `obsidian append path="{config.structure.inbox}" content="- [ ] {text} <!-- {YYYY-MM-DD HH:MM} -->"`
- If inbox doesn't exist yet: `obsidian create path="{config.structure.inbox}" content="# Inbox\n\n- [ ] {text} <!-- {YYYY-MM-DD HH:MM} -->"`

## Process

### 1. Capture input

Take the user's text exactly as provided. Do NOT reformulate, classify, or ask questions. Speed is the priority.

If user provides no text (just `/dump` with nothing else), ask: "Que quieres capturar?"

### 2. Append to Inbox

Use the obsidian CLI:
- If inbox exists: `obsidian append path="{config.structure.inbox}" content="- [ ] {text} <!-- {YYYY-MM-DD HH:MM} -->"`
- If inbox doesn't exist: `obsidian create path="{config.structure.inbox}" content="# Inbox\n\n- [ ] {text} <!-- {YYYY-MM-DD HH:MM} -->"`

Append at the end of existing content, after the last line.

### 3. Confirm

Show only:

> Capturado en Inbox. Usa /process-inbox cuando quieras procesar.

Nothing else. No summary, no suggestions, no follow-up questions.

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| Inbox file doesn't exist | Create it with `# Inbox` header |
| Empty input | Ask user what to capture |
| config.yaml missing | Standard Step 0 error (copy config.example.yaml) |

## Important Rules

- NEVER ask for classification, tags, or context -- that's /process-inbox's job
- NEVER reformulate the input -- capture raw text exactly as the user typed it
- NEVER add wikilinks or formatting -- raw capture only
- Minimize output -- one confirmation line only
- Spanish language for confirmation text
- Do NOT run git commit -- speed is priority, inbox items are ephemeral
- Do NOT show the captured item back to the user -- they just typed it, they know what it says
