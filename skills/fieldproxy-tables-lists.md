---
name: fieldproxy-tables-lists
description: |
  FieldProxy table and list widget patterns for data display. Load this skill when building data tables, dynamic lists, item templates, pagination, search, empty states, or any data-driven display.
---

# FieldProxy Tables & Lists — Tier 2 Skill

## Table Widget

Tables render data as a grid with sorting, filtering, column config, and row actions.

```json
{
  "type": "table",
  "width": 12,
  "data": "items_query.data",
  "element_id": "items_table",
  "columns": [
    { "field": "id", "title": "ID", "width": 80 },
    { "field": "title", "title": "Title", "width": 200 },
    { "field": "status", "title": "Status", "width": 120 },
    { "field": "created_date", "title": "Created", "width": 150 }
  ],
  "actions": [
    {
      "icon": "eye",
      "text": "View",
      "element_id": "act_view",
      "onclick": {
        "1": {
          "type": "navigation",
          "app_id": "APP_ID",
          "view_id": "detail_view",
          "arguments": { "item_id": "{{rowData.id}}" },
          "navigation_type": "view",
          "element_id": "nav_detail"
        }
      }
    },
    {
      "icon": "trash-2",
      "text": "Delete",
      "element_id": "act_delete",
      "onclick": {
        "1": { "type": "confirm", "title": "Delete?", "message": "Are you sure?", "element_id": "confirm_del" },
        "2": { "type": "execute_query", "query": "UPDATE items SET deleted_at = NOW() WHERE id = {{rowData.id}}", "element_id": "q_del" },
        "3": { "type": "toast", "message": "Deleted", "toast_type": "success", "element_id": "toast_del" },
        "4": { "type": "refresh_data", "element_id": "refresh_del" }
      }
    }
  ]
}
```

### Table data access: `{{rowData.field}}`

`rowData` is ONLY available in table row actions. It is injected by the table widget into each row's onclick context. Contains all column field values for the clicked row.

```
{{rowData.id}}          — ID of clicked row
{{rowData.status}}      — Status field
{{rowData.email}}       — Any column field
```

Also available as `{{arguments.rowData.field}}` in action visibility conditions.

### Row onclick (navigate on row click)
```json
{
  "type": "table",
  "onclick": {
    "1": {
      "type": "navigation",
      "view_id": "detail_view",
      "arguments": { "id": "{{rowData.id}}" },
      "navigation_type": "view",
      "element_id": "nav_row"
    }
  }
}
```

### Bulk actions with selected rows
```
{{input_data.table_id.ids}}              — Selected row IDs
{{input_data.table_id.val.[0].field}}    — First selected row's field
```

## List Widget

Lists render repeating item templates from query data.

```json
{
  "type": "list",
  "data": "items_query.data",
  "spacing": 12,
  "element_id": "items_list",
  "empty": "No items found.",
  "ui": [
    {
      "type": "container",
      "width": 12,
      "color": "#FFFFFF",
      "border_color": "#E5E7EB",
      "border_width": 1,
      "border_radius": 12,
      "padding_top": 16, "padding_left": 16, "padding_right": 16, "padding_bottom": 16,
      "element_id": "list_item_card",
      "ui": [
        {
          "type": "row",
          "element_id": "list_item_row",
          "cross_axis_alignment": "center",
          "ui": [
            { "text": "{{data.[index].title}}", "type": "text", "color": "#111827", "font_size": 14, "font_weight": "600", "element_id": "list_title" },
            { "type": "spacer", "element_id": "list_sp" },
            { "text": "{{data.[index].status}}", "type": "text", "color": "#6B7280", "font_size": 12, "element_id": "list_status" }
          ]
        }
      ]
    }
  ]
}
```

### List data access: `{{data.[index].field}}`

Inside list item templates, `[index]` resolves to the current row's absolute index (accounting for pagination). This is the ONLY way to access per-item data in lists. `rowData` does NOT work in lists.

```
{{data.[index].title}}       — Current item's title
{{data.[index].status}}      — Current item's status
{{data.[index].id}}          — Current item's ID
```

### List with pagination
```json
{
  "type": "list",
  "data": "query.data",
  "pagination": true,
  "page_size": 10,
  "element_id": "paged_list"
}
```

### List with search
Combine a text input above the list with a query that uses the input value:
```sql
SELECT * FROM items WHERE title ILIKE '%{{input_data.search_input.0}}%' ORDER BY created_at DESC
```

## Empty State Pattern

Use a stats count query + visibility conditions to toggle between content and empty state.

### Stats query
```json
"data": {
  "items_stats": "SELECT COUNT(*)::TEXT as total_count FROM items WHERE deleted_at IS NULL"
}
```

### Empty state (visible when count = 0)
```json
{
  "type": "container",
  "color": "transparent",
  "width": 12,
  "padding_top": 32, "padding_bottom": 32,
  "element_id": "empty_state",
  "visibility_condition": "'{{items_stats.data.[0].total_count}}' == '0'",
  "ui": [{
    "type": "column",
    "element_id": "empty_col",
    "cross_axis_alignment": "center",
    "ui": [
      { "icon": "inbox", "type": "icon", "color": "#D1D5DB", "icon_size": 40, "element_id": "empty_icon" },
      { "type": "space_widget", "space": 16, "element_id": "sp_empty_1" },
      { "text": "No Items Yet", "type": "text", "color": "#374151", "font_size": 16, "font_weight": "700", "element_id": "empty_title" },
      { "type": "space_widget", "space": 8, "element_id": "sp_empty_2" },
      { "text": "Create your first item to get started.", "type": "text", "color": "#9CA3AF", "font_size": 13, "element_id": "empty_desc" }
    ]
  }]
}
```

### Content (visible when count > 0)
```json
{
  "type": "list",
  "visibility_condition": "'{{items_stats.data.[0].total_count}}' != '0'",
  ...
}
```

IMPORTANT: Always cast COUNT to `::TEXT` and wrap template values in single quotes in visibility conditions.

## Conditional Rendering in Lists

### Chat bubbles pattern (direction-based styling)
Two containers inside each list item, each with opposing visibility conditions:

```json
{
  "type": "container",
  "element_id": "bubble_inbound",
  "visibility_condition": "'{{data.[index].direction}}' == 'inbound'",
  "color": "#EFF6FF",
  "width": 10,
  "border_radius": 16,
  "ui": [/* inbound message content */]
},
{
  "type": "container",
  "element_id": "bubble_outbound",
  "visibility_condition": "'{{data.[index].direction}}' == 'outbound'",
  "color": "#F0FDF4",
  "width": 10,
  "border_radius": 16,
  "ui": [/* outbound message content */]
}
```

Use `width: 10` (not 12) to leave alignment gap. Wrap in a row with `main_axis_alignment: "start"` for inbound, `"end"` for outbound.

## Table vs List — When to Use Which

| Feature | Table | List |
|---------|-------|------|
| Data access | `{{rowData.field}}` | `{{data.[index].field}}` |
| Column sorting | Yes | No |
| Column filtering | Yes | No |
| Custom item layout | Limited (column cells) | Full (any widget template) |
| Bulk selection | Yes (`input_data.table_id.ids`) | No |
| Pagination | Built-in | Configure with `pagination: true` |
| Best for | Structured data grids | Custom card/tile layouts |
