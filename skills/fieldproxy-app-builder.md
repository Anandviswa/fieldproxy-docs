---
name: fieldproxy-app-builder
description: |
  Build FieldProxy operations portal apps using their JSON-based declarative UI framework. Use this skill whenever the user wants to create, modify, or understand a FieldProxy app, portal, dashboard, or custom view. Also trigger when the user mentions: FieldProxy, operations portal, app builder JSON, custom portal, service management app, field service app, or asks to build a web app with sidebar navigation, dashboards, tables, forms, kanban boards, or CRUD operations backed by SQL queries. ALWAYS use this skill when the user wants to create or edit FieldProxy app JSON, even if they just say "build me a portal" or "create a dashboard".
---

# FieldProxy App Builder — Core Skill (Tier 1, Always Loaded)

## App Shell Structure

```json
{
  "type": "web",
  "title": "App Title",
  "app_id": "YOUR_APP_ID",
  "default_view": "home_view",
  "description": "App description",
  "element_id": "web_root",
  "metadata": { "version": "1.0.0", "lastModified": "2025-01-01T00:00:00.000Z" },
  "views": {
    "home_view": { ... },
    "detail_view": { ... }
  }
}
```

## View Structure

```json
{
  "ui": [],
  "data": {},
  "label": "Page Title",
  "margin": { "top": 0, "left": 0, "right": 0, "bottom": 0 },
  "view_id": "my_view",
  "universal_search": {}
}
```

- `ui` — Array of widget objects (the visual layout)
- `data` — Named SQL queries that fetch data for the view
- `label` — Display title for browser tab
- `view_id` — Unique identifier for this view

## Template Expressions

### Data query results
```
{{query_name.data.[index].field_name}}   — Current row in a list/table iteration (preferred)
{{query_name.data.[0].field_name}}       — Hardcoded first row (for single-record queries)
{{query_name.data}}                      — Full result set array
```
Use `[index]` as the default pattern. Use `[0]` only when the query always returns exactly one row.

### Input values (varies by input type)

**Simple inputs** (text, number, email, phone, date, rating, step_counter, scan):
```
{{input_data.element_id.0}}             — In popup/non-list context (string key "0")
{{input_data.element_id.[index]}}       — In list context (current row index)
```

**Dropdown** (single, with `store` config):
```
{{input_data.element_id.0}}             — Returns the stored field value
```

**Dropdown** (multi-select):
```
{{input_data.element_id.0}}             — Returns string like "{Option A, Option B}"
```

**Checkbox**:
```
{{input_data.element_id.0}}             — Returns "true" or "false"
```

**Date range**:
```
{{input_data.element_id.0.[0]}}         — Range start (ISO string)
{{input_data.element_id.0.[1]}}         — Range end (ISO string)
```

**File / Image**:
```
{{input_data.element_id.0.url}}         — List of uploaded URLs
{{input_data.element_id.0.url.[0]}}     — First uploaded URL
```

**Signature**:
```
{{input_data.element_id.0.url}}         — Signature image URL
```

**Location**:
```
{{input_data.element_id.0.address}}           — Address string
{{input_data.element_id.0.coordinates.[0]}}   — Longitude
{{input_data.element_id.0.coordinates.[1]}}   — Latitude
```

**MCQ** (single/radio):
```
{{input_data.element_id.0}}             — Returns list with one item e.g. ["Choice"]
```

For the full input type reference, load `fieldproxy-input-forms`.

### Table row context (tables ONLY)
```
{{rowData.field_name}}                   — Current row's field value
{{arguments.rowData.field_name}}         — Same, in action context
```
`rowData` is injected by the table widget into onclick actions. It does NOT exist in lists — lists use `{{data.[index].field}}`.

### Navigation arguments
```
{{arguments.param_name}}                — Value passed via navigation
{{arguments.item_id}}                   — Example: ID passed from list → detail
```

### Function chain results
```
{{functions.1.field}}                    — Result from onclick step 1
{{functions.2.nested.field}}             — Nested result from step 2
```

### Special functions (in SQL queries)
```
<!getUserId()!>                          — Current logged-in user ID
```

### CRITICAL: `.0` vs `.[index]` vs `.[0]`

| Syntax | Resolves to | Use when |
|--------|-------------|----------|
| `.0` | String key `"0"` in map | Popup/form context — always reads row 0 |
| `.[index]` | Runtime integer index | Inside list — reads current row's index |
| `.[0]` | Integer index `0` on a List | Indexing into an array value (date range, file URLs, coordinates) |

NEVER use `.[0]` to access input_data row 0 — use `.0` (no brackets).

## Data Queries

SQL queries in the `data` object, executed against PostgreSQL:

```json
{
  "data": {
    "items": "SELECT id, name, status FROM my_table WHERE deleted_at IS NULL ORDER BY created_at DESC",
    "stats": "SELECT COUNT(*)::TEXT as total FROM my_table WHERE deleted_at IS NULL"
  }
}
```

### Common SQL patterns
- `COALESCE(field, 'Default')` — null safety
- `TO_CHAR(field AT TIME ZONE 'UTC', 'DD Mon YYYY, HH24:MI')` — date formatting
- `COUNT(*)::TEXT` — cast to TEXT for template string comparisons
- `LEFT JOIN` — related data lookups
- `CASE WHEN ... THEN ... ELSE ... END` — conditional values
- `ILIKE '%{{input_data.search.0}}%'` — search filtering
- Dynamic filters: `AND (status = '{{input_data.filter.0}}' OR '{{input_data.filter.0}}' = '')`

### JSONB array expansion
```sql
SELECT att->>'filename' as filename
FROM emails em, jsonb_array_elements(COALESCE(em.attachments,'[]'::jsonb)) att
```

### UNION ALL for mixed sources
```sql
SELECT ... 'inbound' as direction FROM inbound_table
UNION ALL
SELECT ... 'outbound' as direction FROM outbound_table
ORDER BY created_at
```

## Sidebar Navigation

```json
{
  "type": "sidebar",
  "element_id": "main_sidebar",
  "items": [
    {
      "icon": "home",
      "label": "Dashboard",
      "onclick": {
        "1": {
          "type": "navigation",
          "app_id": "APP_ID",
          "view_id": "home_view",
          "element_id": "nav_home",
          "navigation_type": "view"
        }
      },
      "children": [],
      "active_icon_color": "#FFFFFF",
      "active_text_color": "#FFFFFF",
      "inactive_icon_color": "#9CA3AF",
      "inactive_text_color": "#D1D5DB",
      "active_background_color": "#3B82F6"
    }
  ]
}
```

Sidebar items support:
- `children[]` — nested sub-items (expandable sections)
- `badge: "{{query.data.[0].count}}"` — dynamic badge count
- `badge_color: "#EF4444"` — badge styling

## element_id Rules

Every widget MUST have a unique `element_id`. Use descriptive snake_case: `dashboard_title`, `filter_date_range`, `jobs_table`.

**FORBIDDEN values in element_id:**
- `index` or `[index]` — collides with template index resolution, silently reads wrong data
- `[0]`, `[1]`, etc. — bracket notation causes integer-index lookup, returns null
- `0`, `1` (bare numbers) — ambiguous with internal sub-key indexes
- Values containing `.` (dots) — split as nested path in template resolution

**Missing element_id behavior:** System generates `unknown_<timestamp>` fallback — causes rebuild flicker, breaks visibility conditions, state collision between elements.

## Skill Loading Reference

| Building... | Load these skills |
|-------------|-------------------|
| Page layouts, cards, columns | `fieldproxy-layout-patterns` |
| Forms, popups, CRUD | `fieldproxy-input-forms` |
| Tables, lists, data display | `fieldproxy-tables-lists` |
| Dashboards, charts, KPIs | `fieldproxy-charts-kpis` |
| Kanban, calendar, map, pivot | `fieldproxy-advanced-widgets` |
| Button actions, navigation | `fieldproxy-actions` |
| Layout bug debugging | `fieldproxy-widget-rules` |
| Dart platform code | `fieldproxy-architecture` |

## Icon Reference (Lucide Set)

`home`, `layers`, `clipboard`, `users`, `calendar`, `phone`, `dollar-sign`, `file-text`, `credit-card`, `bar-chart-2`, `settings`, `inbox`, `eye`, `edit`, `trash-2`, `plus`, `search`, `filter`, `download`, `upload`, `send`, `check-circle`, `alert-circle`, `clock`, `map-pin`, `repeat`, `mail`, `star`, `briefcase`, `truck`, `tool`, `map`, `globe`, `archive`, `refresh-cw`, `x`, `check`, `chevron-right`, `chevron-down`, `more-vertical`, `zap`, `user`, `hash`, `tag`, `flag`, `bell`, `link`, `external-link`, `copy`, `printer`, `save`, `lock`, `unlock`, `shield`, `activity`, `trending-up`, `trending-down`, `percent`, `pie-chart`, `grid`, `list`, `layout`, `sidebar`, `menu`, `maximize`, `minimize`
