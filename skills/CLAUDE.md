# FieldProxy Project — Claude Instructions

## MANDATORY: Before Writing ANY FieldProxy JSON

**ALWAYS load these skills before generating or modifying any `.json` view file:**

1. Read `.claude/skills/fieldproxy-widget-rules.md` — 34 verified rules + pre-flight checklist
2. Read `.claude/skills/fieldproxy-error-log.md` — 86+ past bugs to avoid repeating

**Do NOT skip this step.** These files contain hard-won fixes from real production bugs. Skipping them leads to the SAME bugs being reintroduced — costing hours of debugging.

## Top 10 Most Repeated Mistakes (Quick Reference)

These errors have been made 2+ times each. Memorize them:

1. **CSS `border` doesn't work** — use `border_color` + `border_width` separately (Rule 3)
2. **`icon_button` uses `bg_color`/`color`/`size`** — NOT `background_color`/`icon_color` (Rule 17, repeated 5+ times)
3. **`input_type: "search"` doesn't exist** — use `"text"` (Rule 14)
4. **`visibility_condition` on inputs is ignored** — wrap in container (Rule 26)
5. **`expanded: true` only works on `text`** — not containers/buttons (Rule 4, repeated 8+ times)
6. **Container bg uses `color`** — NOT `background_color` (Rule 32)
7. **`show_if` doesn't exist** — use `visibility_condition` (Rule 11)
8. **`fixed_height` doesn't exist** — use `height` (Rule 29)
9. **`type: "divider"` doesn't render on web** — use container with height:1 (Rule 31)
10. **`input_data` keyed by `element_id`** — NEVER by `field_name` (Rule 19)

## Data Query Rules

- `data` object queries are **SELECT-only** — INSERT/UPDATE/DELETE silently ignored
- Use `execute_query` inside `onclick` for mutations
- After mutations, use `update_data` (NOT `refresh_data`) to refresh specific data keys
- Always `COALESCE` color/status fields with fallback values

## fp_users Table

- Columns are `id` and `name` (NOT `username`, NO `is_active` column)
- Dropdown pattern: `"show": "[name]", "store": ["name"], "show_key": false`

## Popup Width Gotcha

`calculateDynamicWidth()` ALWAYS uses screenWidth. Inside a 700px popup:
- 3-column: use `width: 2` per column
- 2-column: use `width: 3` per column

## Never Modify

- The "+ New Enquiry" button on `enquiries_view` — do not touch it

## Team

- **Afrin** — Product Designer (design briefs from reference UIs)
- **Nandy** — Layout Engineer (grid widths, overflow, row/wrap/height decisions)
