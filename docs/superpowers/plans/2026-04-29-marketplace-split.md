# life-os Marketplace Split Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorganize the monolithic life-os plugin into a marketplace of 6 focused plugins within the same repo.

**Architecture:** The repo root becomes a pure marketplace (marketplace.json only, no plugin.json). Each domain gets its own subdirectory with plugin.json and skills/. Config moves from `${CLAUDE_PLUGIN_ROOT}/config.yaml` to `~/.config/life-os/config.yaml` so all plugins share a single config file.

**Tech Stack:** Claude Code plugin system, JSON manifests, Markdown skills

---

### Task 1: Create plugin scaffolds

**Files:**
- Create: `rituals/.claude-plugin/plugin.json`
- Create: `sync/.claude-plugin/plugin.json`
- Create: `capture/.claude-plugin/plugin.json`
- Create: `planning/.claude-plugin/plugin.json`
- Create: `training/.claude-plugin/plugin.json`
- Create: `content/.claude-plugin/plugin.json`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p rituals/.claude-plugin rituals/skills
mkdir -p sync/.claude-plugin sync/skills
mkdir -p capture/.claude-plugin capture/skills
mkdir -p planning/.claude-plugin planning/skills
mkdir -p training/.claude-plugin training/skills
mkdir -p content/.claude-plugin content/skills
```

Expected: 6 plugin directories at repo root, each with `.claude-plugin/` and `skills/`.

- [ ] **Step 2: Create rituals/.claude-plugin/plugin.json**

```json
{
  "name": "rituals",
  "description": "Daily and weekly rituals: morning routine, daily note, end-of-day close, and weekly review.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["gtd", "daily-notes", "rituals", "obsidian", "productivity"]
}
```

- [ ] **Step 3: Create sync/.claude-plugin/plugin.json**

```json
{
  "name": "sync",
  "description": "Sync connectors: calendar, Granola, Jira, Slack, and Telegram into the Obsidian vault.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["sync", "calendar", "jira", "slack", "telegram", "obsidian"]
}
```

- [ ] **Step 4: Create capture/.claude-plugin/plugin.json**

```json
{
  "name": "capture",
  "description": "Capture and inbox processing: quick dump, capture knowledge, process inbox, clarify tasks.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["gtd", "capture", "inbox", "obsidian", "productivity"]
}
```

- [ ] **Step 5: Create planning/.claude-plugin/plugin.json**

```json
{
  "name": "planning",
  "description": "Planning tools: meeting prep, system status dashboard, backlog review, weekly review.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["gtd", "planning", "review", "obsidian", "productivity"]
}
```

- [ ] **Step 6: Create training/.claude-plugin/plugin.json**

```json
{
  "name": "training",
  "description": "Training tracking and CSV import: query training history, personal records, and sync from Heavy/Strong.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["training", "fitness", "obsidian", "health"]
}
```

- [ ] **Step 7: Create content/.claude-plugin/plugin.json**

```json
{
  "name": "content",
  "description": "Content generation and nutrition: LinkedIn posts, blog drafts, and weekly shopping lists.",
  "version": "0.1.0",
  "author": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "repository": "https://github.com/aldorea/life-os",
  "license": "MIT",
  "keywords": ["content", "linkedin", "shopping", "obsidian"]
}
```

- [ ] **Step 8: Verify all plugin.json files are valid JSON**

```bash
for d in rituals sync capture planning training content; do
  echo -n "$d: "
  python3 -c "import json; json.load(open('$d/.claude-plugin/plugin.json')); print('valid')"
done
```

Expected: 6 lines each saying `valid`.

- [ ] **Step 9: Commit**

```bash
git add rituals sync capture planning training content
git commit -m "chore(marketplace): scaffold 6 plugin directories"
```

---

### Task 2: Move rituals skills

**Files:**
- Move: `skills/morning/` → `rituals/skills/morning/`
- Move: `skills/today/` → `rituals/skills/today/`
- Move: `skills/close/` → `rituals/skills/close/`

Note: `skills/week/` does not exist yet — create it in `rituals/skills/week/` when implementing Phase 4.

- [ ] **Step 1: Move skills**

```bash
mv skills/morning rituals/skills/morning
mv skills/today rituals/skills/today
mv skills/close rituals/skills/close
```

- [ ] **Step 2: Verify**

```bash
ls rituals/skills/
```

Expected: `close  morning  today`

- [ ] **Step 3: Commit**

```bash
git add rituals/skills/ skills/
git commit -m "chore(marketplace): move rituals skills to rituals plugin"
```

---

### Task 3: Move sync skills

**Files:**
- Move: `skills/sync/` → `sync/skills/sync/`
- Move: `skills/sync-calendar/` → `sync/skills/sync-calendar/`
- Move: `skills/sync-granola/` → `sync/skills/sync-granola/`
- Move: `skills/sync-jira/` → `sync/skills/sync-jira/`
- Move: `skills/sync-slack/` → `sync/skills/sync-slack/`
- Move: `skills/sync-telegram/` → `sync/skills/sync-telegram/`

- [ ] **Step 1: Move skills**

```bash
mv skills/sync sync/skills/sync
mv skills/sync-calendar sync/skills/sync-calendar
mv skills/sync-granola sync/skills/sync-granola
mv skills/sync-jira sync/skills/sync-jira
mv skills/sync-slack sync/skills/sync-slack
mv skills/sync-telegram sync/skills/sync-telegram
```

- [ ] **Step 2: Verify**

```bash
ls sync/skills/
```

Expected: `sync  sync-calendar  sync-granola  sync-jira  sync-slack  sync-telegram`

- [ ] **Step 3: Commit**

```bash
git add sync/skills/ skills/
git commit -m "chore(marketplace): move sync skills to sync plugin"
```

---

### Task 4: Move capture, planning, training, content skills

**Files:**
- Move: `skills/dump/` → `capture/skills/dump/`
- Move: `skills/capture/` → `capture/skills/capture/`
- Move: `skills/prep/` → `planning/skills/prep/`
- Move: `skills/review/` → `planning/skills/review/`
- Move: `skills/train/` → `training/skills/train/`
- Move: `skills/sync-training/` → `training/skills/sync-training/`
- Move: `skills/content/` → `content/skills/content/`

- [ ] **Step 1: Move capture skills**

```bash
mv skills/dump capture/skills/dump
mv skills/capture capture/skills/capture
```

- [ ] **Step 2: Move planning skills**

```bash
mv skills/prep planning/skills/prep
mv skills/review planning/skills/review
```

- [ ] **Step 3: Move training skills**

```bash
mv skills/train training/skills/train
mv skills/sync-training training/skills/sync-training
```

- [ ] **Step 4: Move content skills**

```bash
mv skills/content content/skills/content
```

- [ ] **Step 5: Verify all plugins have the right skills**

```bash
echo "capture:" && ls capture/skills/
echo "planning:" && ls planning/skills/
echo "training:" && ls training/skills/
echo "content:" && ls content/skills/
```

Expected:
```
capture: capture  dump
planning: prep  review
training: sync-training  train
content: content
```

- [ ] **Step 6: Verify old skills/ directory is now empty**

```bash
ls skills/
```

Expected: empty output (no files or directories listed).

- [ ] **Step 7: Remove empty skills/ directory**

```bash
rmdir skills/
```

- [ ] **Step 8: Commit**

```bash
git add capture/skills/ planning/skills/ training/skills/ content/skills/ skills/
git commit -m "chore(marketplace): move remaining skills to their plugins"
```

---

### Task 5: Move hooks to rituals plugin

**Files:**
- Move: `hooks/` → `rituals/hooks/`

- [ ] **Step 1: Move hooks directory**

```bash
mv hooks rituals/hooks
```

- [ ] **Step 2: Verify content is intact**

```bash
cat rituals/hooks/hooks.json
```

Expected: JSON with `SessionStart` hook containing the morning ritual echo command.

- [ ] **Step 3: Commit**

```bash
git add rituals/hooks/ hooks/
git commit -m "chore(marketplace): move hooks to rituals plugin"
```

---

### Task 6: Update marketplace.json

**Files:**
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Replace marketplace.json with 6-plugin version**

Write `.claude-plugin/marketplace.json`:

```json
{
  "name": "life-os",
  "owner": {
    "name": "aldorea",
    "url": "https://github.com/aldorea"
  },
  "plugins": [
    {
      "name": "rituals",
      "source": "./rituals",
      "version": "0.1.0",
      "description": "Daily and weekly rituals: morning routine, daily note, end-of-day close, and weekly review."
    },
    {
      "name": "sync",
      "source": "./sync",
      "version": "0.1.0",
      "description": "Sync connectors: calendar, Granola, Jira, Slack, and Telegram into the Obsidian vault."
    },
    {
      "name": "capture",
      "source": "./capture",
      "version": "0.1.0",
      "description": "Capture and inbox processing: quick dump, capture knowledge, process inbox, clarify tasks."
    },
    {
      "name": "planning",
      "source": "./planning",
      "version": "0.1.0",
      "description": "Planning tools: meeting prep, system status dashboard, backlog review, weekly review."
    },
    {
      "name": "training",
      "source": "./training",
      "version": "0.1.0",
      "description": "Training tracking and CSV import: query training history, personal records, and sync from Heavy/Strong."
    },
    {
      "name": "content",
      "source": "./content",
      "version": "0.1.0",
      "description": "Content generation and nutrition: LinkedIn posts, blog drafts, and weekly shopping lists."
    }
  ]
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -c "import json; json.load(open('.claude-plugin/marketplace.json')); print('valid')"
```

Expected: `valid`

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "chore(marketplace): update marketplace.json with 6 plugin entries"
```

---

### Task 7: Remove old top-level plugin.json

**Files:**
- Delete: `.claude-plugin/plugin.json`

- [ ] **Step 1: Remove plugin.json**

```bash
rm .claude-plugin/plugin.json
```

- [ ] **Step 2: Verify only marketplace.json remains**

```bash
ls .claude-plugin/
```

Expected: `marketplace.json` only.

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "chore(marketplace): remove top-level plugin.json — repo is now a pure marketplace"
```

---

### Task 8: Update config paths in all skills

**Files:** All `SKILL.md` files that reference `${CLAUDE_PLUGIN_ROOT}/config.yaml`

13 skills currently reference this path: morning, today, close, sync, sync-jira, sync-telegram, sync-training, sync-granola (check), dump, prep, train, sync-slack (check), sync-calendar (check). The `capture`, `content`, and `review` skills do not reference the config path and require no changes.

- [ ] **Step 1: Confirm all occurrences before changing**

```bash
grep -r "CLAUDE_PLUGIN_ROOT.*config\.yaml" --include="*.md" .
```

Record the count of matches. Expected: matches in rituals/, sync/, capture/, planning/, training/ skill directories.

- [ ] **Step 2: Replace all occurrences**

```bash
find . -name "SKILL.md" -exec sed -i '' 's|\${CLAUDE_PLUGIN_ROOT}/config\.yaml|~/.config/life-os/config.yaml|g' {} \;
```

- [ ] **Step 3: Verify no old references remain**

```bash
grep -r "CLAUDE_PLUGIN_ROOT.*config\.yaml" --include="*.md" .
```

Expected: no output.

- [ ] **Step 4: Verify new paths are present**

```bash
grep -r "~/.config/life-os/config.yaml" --include="*.md" . | wc -l
```

Expected: same count as Step 1.

- [ ] **Step 5: Commit**

```bash
git add rituals/skills sync/skills capture/skills planning/skills training/skills content/skills
git commit -m "chore(marketplace): update config path to ~/.config/life-os/config.yaml in all skills"
```

---

### Task 9: Update config.example.yaml and README

**Files:**
- Modify: `config.example.yaml` (header comment)
- Modify: `README.md`

- [ ] **Step 1: Update config.example.yaml header**

Replace the first two lines of `config.example.yaml`:

Old:
```yaml
# life-os configuration
# Copy this file to config.yaml and fill in your values.
```

New:
```yaml
# life-os configuration
# Copy this file to ~/.config/life-os/config.yaml and fill in your values.
# Create the directory first: mkdir -p ~/.config/life-os
```

- [ ] **Step 2: Verify header change**

```bash
head -3 config.example.yaml
```

Expected: Lines reference `~/.config/life-os/config.yaml`.

- [ ] **Step 3: Update README.md Installation and Configuration sections**

Replace the `## Installation` and `## Configuration` sections in `README.md` with:

```markdown
## Installation

Add the marketplace, then install the plugins you need:

```bash
/plugin marketplace add github:aldorea/life-os
/plugin install rituals@life-os
/plugin install sync@life-os
/plugin install capture@life-os
/plugin install planning@life-os
/plugin install training@life-os   # optional
/plugin install content@life-os    # optional
```

## Configuration

```bash
mkdir -p ~/.config/life-os
cp config.example.yaml ~/.config/life-os/config.yaml
# Edit ~/.config/life-os/config.yaml with your vault path and preferences
```
```

- [ ] **Step 4: Commit**

```bash
git add config.example.yaml README.md
git commit -m "docs(marketplace): update setup instructions for multi-plugin structure"
```

---

## Skill invocation after migration

| Plugin | Skills |
|--------|--------|
| `rituals` | `rituals:morning`, `rituals:today`, `rituals:close` |
| `sync` | `sync:sync`, `sync:sync-calendar`, `sync:sync-granola`, `sync:sync-jira`, `sync:sync-slack`, `sync:sync-telegram` |
| `capture` | `capture:dump`, `capture:capture` |
| `planning` | `planning:prep`, `planning:review` |
| `training` | `training:train`, `training:sync-training` |
| `content` | `content:content` |

Future skills (not yet implemented) go directly into their plugin's `skills/` directory — no migration needed.
