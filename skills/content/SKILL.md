---
name: content
description: Use when the user wants to generate content (LinkedIn posts, blog articles, talks/presentations) from their knowledge base. Use when user says "content", "contenido", "genera un post", "escribe un artículo", "prepara una talk", "qué puedo publicar", "draft", "LinkedIn".
---

# content

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generates content drafts (LinkedIn posts, blog articles, talk outlines) from the knowledge base. Three modes of operation.

## Mode 1: Draft from topic

```
/content linkedin ai-agents
/content blog websockets-arquitectura
/content talk life-os
```

### Process

1. Read the topic note from `VAULT/{config.structure.knowledge}/[topic].md`
2. Read `VAULT/{config.structure.content_ideas}` for existing ideas about this topic
3. Read `VAULT/{config.structure.voice}` if it exists (writing style guide)
4. Read `VAULT/{config.structure.goals}` to align with current content goals
5. Generate draft adapted to format:

**LinkedIn post:**
- Hook in first line (question, bold statement, or surprising fact)
- 150-300 words
- Personal experience angle — NOT generic advice
- End with question or call to engagement
- Use line breaks for readability
- No hashtags unless user asks

**Blog article:**
- Title + subtitle
- 800-1500 words
- Structure: problem → context → solution → learnings → conclusion
- Code examples if relevant
- Personal narrative thread

**Talk/KS outline:**
- Title + one-line pitch
- Audience level
- 5-7 sections with key points per section
- Demo moments identified
- Estimated duration
- Speaker notes per section

6. Present draft conversationally for iteration
7. If user approves, save with `--save` to `VAULT/{config.structure.content_drafts}/YYYY-MM-DD-slug.md`

## Mode 2: Explore what's available

```
/content
```

No arguments → scan all knowledge notes and present:

```markdown
## Temas ready (7+ entradas)
- [topic] (N entradas) → sugerencia: "[content idea]"

## Temas growing (3-6 entradas)
- [topic] (N entradas) → X entradas más para generar contenido

## Ideas pendientes
- [from content-ideas.md, with status and linked topics]

## Alineación con goals
- [from goals.yaml, content-related goals and their status]
```

## Mode 3: From a specific idea

```
/content "quiero escribir sobre cómo uso Granola para alimentar un CRM"
```

1. Search ALL knowledge notes for relevant entries (keyword + semantic match)
2. Aggregate related entries across topics
3. Cross-reference with meeting notes in `VAULT/{config.structure.meetings}/` if relevant
4. Generate draft combining sources
5. Cite which knowledge entries were used

## Output Rules

- Output is ALWAYS conversational (shown in chat, not saved) unless `--save` flag
- Iterate with user: "¿Cambio algo? ¿Más técnico? ¿Más personal?"
- When saving, use `VAULT/{config.structure.content_drafts}/YYYY-MM-DD-slug.md` with frontmatter:

```yaml
---
type: linkedin | blog | talk
topic: [topic name]
status: draft | review | published
date: YYYY-MM-DD
sources: [list of knowledge notes used]
---
```

## Content Style

- First person — Alfonso's voice and experiences
- Concrete examples over abstract advice
- Show the thinking process, not just the result
- Technical but accessible
- Spanish by default, English if user specifies or goal requires it (e.g., "KS en inglés")
- NO generic AI hype — always grounded in real usage

## Graceful Degradation

| Missing | Behavior |
|---------|----------|
| No voice.md | Use default style (technical, personal, concrete) |
| Topic has few entries | Warn user, suggest capturing more first, or generate shorter piece |
| No content-ideas.md | Skip ideas section, generate from knowledge entries directly |
| No goals.yaml | Skip alignment section |

