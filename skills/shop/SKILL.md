---
name: shop
description: Use when the user wants to generate a weekly shopping list from nutritionist menus, plan meals, or says "shop", "compra", "lista de compra", "qué compro", "supermercado", "menú semanal", "meal plan".
---

# shop

## Step 0 — Load configuration

Read `${CLAUDE_PLUGIN_ROOT}/config.yaml`.
If it doesn't exist, tell the user to copy `config.example.yaml` to `config.yaml` and fill in their values. Stop here.

Set `VAULT` = `{config.vault_path}` for all file operations below.

## Overview

Generator that creates a weekly shopping list based on nutritionist menus stored in the vault. Aggregates ingredients across meals, groups by supermarket section, and includes macro summary.

## Process

### 1. Read available menus

Check `VAULT/{config.structure.menus}/` for PDF files or markdown notes from the nutritionist.

If PDFs exist but no parsed recipes cache:
- Read each PDF
- Parse meals: name, ingredients with quantities, macros (kcal, protein, carbs, fat)
- Write parsed data as cache

### 2. Ask user for meal selection

Present available meals and ask:
- "¿Qué menús quieres para esta semana?"
- Or suggest a balanced rotation if user says "tú elige" or "sorpréndeme"

When suggesting, aim for:
- Variety (don't repeat same protein 2 days in a row)
- Balance across the week (match macro targets if defined)
- Practical (consider meal prep — dishes that share ingredients)

### 3. Aggregate ingredients

For each selected meal:
- Extract all ingredients with quantities
- Sum quantities for same ingredient across different meals
- Round up to practical purchase quantities (e.g., 350g chicken → 400g)

### 4. Generate shopping list

Write to `VAULT/{config.structure.shopping_lists}/YYYY-W##.md`:

```markdown
---
week: YYYY-W##
meals_planned: X
---

# Lista de compra — Semana ##

## Menú de la semana

| Día | Comida | Cena |
|-----|--------|------|
| Lunes | [Plato] | [Plato] |
| Martes | [Plato] | [Plato] |
| ... | ... | ... |

## Lista de compra
```

Group items using sections from `{config.shopping.sections}`. For each section:

```markdown
### [Section Name]
- [ ] Ingredient — quantity (meals that use it)
```

Include macros table if available:

```markdown
## Macros de la semana

| Día | Kcal | Proteína | Carbs | Grasa |
|-----|------|----------|-------|-------|
| Lunes | 2100 | 150g | 200g | 70g |
| **Media** | **XXXX** | **XXXg** | **XXXg** | **XXg** |
```

### 5. Graceful degradation

- If no menus in configured folder: ask user to add them
- If macros not available: skip macro table, note "Macros no disponibles"

## Important Rules

- Ingredients grouped by supermarket section (as shopper would walk through)
- Each ingredient shows which meals use it (for traceability)
- Quantities are practical (round up, account for waste)
- Shopping list items are checkboxes `- [ ]` for use in Obsidian
- Ask before generating — meal selection is user's choice
