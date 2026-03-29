---
name: fieldproxy-advanced-widgets
description: |
  FieldProxy advanced widget patterns for kanban boards, calendars, maps, pivot tables, and import/export. Load this skill when building kanban views, scheduling calendars, map displays, pivot reports, or data import/export features.
---

# FieldProxy Advanced Widgets — Tier 2 Skill

## Kanban Board

Displays items as draggable cards organized in status columns.

```json
{
  "type": "kanban",
  "width": 12,
  "data": "items_query.data",
  "element_id": "items_kanban",
  "status_field": "status",
  "columns": [
    { "value": "open", "label": "Open", "color": "#3B82F6" },
    { "value": "in_progress", "label": "In Progress", "color": "#F59E0B" },
    { "value": "review", "label": "Review", "color": "#8B5CF6" },
    { "value": "completed", "label": "Completed", "color": "#10B981" }
  ],
  "card_template": {
    "title_field": "title",
    "subtitle_field": "assigned_to",
    "date_field": "due_date"
  },
  "onclick": {
    "1": {
      "type": "navigation",
      "view_id": "detail_view",
      "arguments": { "id": "{{rowData.id}}" },
      "navigation_type": "view",
      "element_id": "nav_kanban_detail"
    }
  },
  "on_drop": {
    "1": {
      "type": "execute_query",
      "query": "UPDATE items SET status = '{{new_status}}' WHERE id = {{card_id}}",
      "element_id": "q_kanban_move"
    },
    "2": { "type": "refresh_data", "element_id": "refresh_kanban" }
  }
}
```

The kanban widget has its own `KanbanController` managing column state and drag-drop operations.

## Calendar

### Simple calendar
```json
{
  "type": "calendar",
  "width": 12,
  "data": "events_query.data",
  "element_id": "events_calendar",
  "date_field": "event_date",
  "title_field": "title",
  "color_field": "color"
}
```

### Full calendar (`calendar_full`)
```json
{
  "type": "calendar_full",
  "width": 12,
  "element_id": "full_calendar",
  "data": "events_query.data",
  "date_field": "start_date",
  "end_date_field": "end_date",
  "title_field": "title"
}
```

### Calendar Task Widget (advanced)

Full-featured task calendar with day/week/month views, drag-drop, multi-user support:

```json
{
  "type": "calendar_task_widget",
  "width": 12,
  "element_id": "task_calendar",
  "data": "tasks_query.data",
  "config": {
    "date_field": "scheduled_date",
    "start_time_field": "start_time",
    "end_time_field": "end_time",
    "title_field": "title",
    "assignee_field": "assigned_to",
    "status_field": "status",
    "color_field": "color",
    "views": ["day", "week", "month"],
    "default_view": "week",
    "enable_drag_drop": true,
    "user_split": true
  }
}
```

Has its own `CalendarTaskController` with `CalendarConfig` and `TaskModel`.

## Map Widget

Display markers on a map from query data (Azure Maps / flutter_map):

```json
{
  "type": "map_widget",
  "width": 12,
  "element_id": "locations_map",
  "data": "locations_query.data",
  "latitude_field": "latitude",
  "longitude_field": "longitude",
  "title_field": "name",
  "subtitle_field": "address"
}
```

## Pivot Table

Configurable pivot table with row/column grouping, value aggregation, and drill-down:

```json
{
  "type": "pivot_table",
  "width": 12,
  "element_id": "sales_pivot",
  "data": "sales_query.data",
  "row_fields": ["region", "product"],
  "column_fields": ["quarter"],
  "value_fields": [
    { "field": "revenue", "aggregation": "sum", "label": "Revenue" },
    { "field": "quantity", "aggregation": "sum", "label": "Qty" }
  ]
}
```

## Import Data

Excel/CSV upload widget with column mapping:

```json
{
  "type": "import_data",
  "width": 12,
  "element_id": "import_items",
  "target_table": "items",
  "column_mapping": [
    { "source": "Name", "target": "title" },
    { "source": "Status", "target": "status" },
    { "source": "Email", "target": "email" }
  ]
}
```

## Export Data

Excel download from query data:

```json
{
  "type": "export_data",
  "width": 12,
  "element_id": "export_items",
  "data": "items_query.data",
  "filename": "items_export",
  "columns": [
    { "field": "title", "label": "Title" },
    { "field": "status", "label": "Status" },
    { "field": "created_date", "label": "Created" }
  ]
}
```

## Expandable Card

Card with expand/collapse toggle:

```json
{
  "type": "expandable_card",
  "width": 12,
  "element_id": "exp_card",
  "card_type": "expandable",
  "title": "Section Title",
  "ui": [ /* card content shown when expanded */ ]
}
```

`card_type` values: `expandable` (starts collapsed), `expanded` (starts open), `fixed` (no toggle).

## Vertical Tab Group

```json
{
  "type": "vertical_tab_group",
  "width": 12,
  "element_id": "vtabs",
  "ui": [
    { "title": "General", "ui": [...] },
    { "title": "Advanced", "ui": [...] }
  ]
}
```

## PDF Viewer

```json
{
  "type": "pdf",
  "width": 12,
  "element_id": "pdf_viewer",
  "url": "{{detail.data.[0].document_url}}"
}
```

## WebView

```json
{
  "type": "webview",
  "width": 12,
  "element_id": "embedded_web",
  "url": "https://example.com/embed"
}
```

## Wrap Layout

Horizontal flow layout that wraps to next line:

```json
{
  "type": "wrap",
  "element_id": "tags_wrap",
  "ui": [
    /* tag pills, badges, etc. — wraps automatically */
  ]
}
```
Run spacing is 16px between items.
