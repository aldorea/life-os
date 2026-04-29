---
name: sync-telegram
description: sync-telegram
---

# sync-telegram

## Step 0 — Load configuration

Read `~/.config/life-os/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

### Load secrets

If `~/.config/life-os/secrets.yaml` exists, also read it and merge its values into the config tree. Secrets values take precedence over any matching key in `config.yaml`. For example, `secrets.telegram.api_key` becomes `config.telegram.api_key`.

The secrets file must have permissions `600` (user read/write only). If it exists but is world/group-readable, warn the user but continue.

If `config.telegram.api_key` is not defined after merging, warn the user that the Telegram API key is missing and stop gracefully. The key can be added to `~/.config/life-os/secrets.yaml` under `telegram.api_key`.

## Overview

Connector that fetches unprocessed messages from the Telegram Training Bot API and writes them to the vault. Training messages go to the training log; everything else goes to Inbox.

## Process

### 1. Fetch messages from API

Call `GET {config.telegram.api_url}/messages?date=YYYY-MM-DD` with header `Authorization: Bearer {config.telegram.api_key}`.

Use WebFetch tool. If the API is unreachable, warn the user and skip gracefully.

Fetch today's date by default. If there are 0 messages, also fetch yesterday's date (in case /close wasn't run yesterday).

### 2. Process training messages

For each message where `type == "training"`:

Append to `VAULT/10 Health/Training/{date formatted as DD-MM-YYYY}.md`:

```
### Captura Telegram ({time})

{raw_text} #training-raw

---
```

Create the file if it doesn't exist with frontmatter:

```yaml
---
date: YYYY-MM-DD
type: training-session
source: telegram
---
```

### 3. Process inbox messages

For each message where `type == "inbox"`:

Append to `VAULT/{config.structure.inbox}`:

```
- [ ] {raw_text} (via Telegram, {date} {time})
```

### 4. Acknowledge processed messages

Call `POST {config.telegram.api_url}/messages/ack` with body `{"ids": [list of processed message IDs]}` and the same auth header.

Use WebFetch tool for the POST request.

### 5. Report

Print summary:

```
sync-telegram: {N} mensajes procesados ({X} training, {Y} inbox)
```

If 0 messages: `sync-telegram: sin mensajes nuevos`

## Graceful Degradation

| Issue | Behavior |
|-------|----------|
| API unreachable | Warn user, skip. Don't block other skills. |
| API returns error | Log error, skip. |
| No messages | Print "sin mensajes nuevos", continue. |
| Ack fails | Warn user. Messages will be re-fetched next time (idempotent). |
