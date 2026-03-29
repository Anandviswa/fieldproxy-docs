---
name: fieldproxy-charts-kpis
description: |
  FieldProxy chart and KPI widget patterns for dashboards and analytics. Load this skill when building dashboards, KPI cards, donut charts, bar charts, or any analytics/reporting view.
---

# FieldProxy Charts & KPIs — Tier 2 Skill

## Grouped Counter (KPI Cards Row)

Most common dashboard pattern — a row of metric cards:

```json
{
  "type": "grouped_counter_1",
  "width": 12,
  "columns": 4,
  "element_count": 4,
  "element_id": "kpi_cards",
  "cards": [
    {
      "icon": "layers",
      "text": "{{stats.data.[0].total}}",
      "label": "Total Items",
      "color": "#EFF6FF",
      "icon_color": "#2563EB",
      "font_size": 16,
      "font_weight": "bold"
    },
    {
      "icon": "clock",
      "text": "{{stats.data.[0].pending}}",
      "label": "Pending",
      "color": "#FEF3C7",
      "icon_color": "#F59E0B",
      "font_size": 16,
      "font_weight": "bold"
    },
    {
      "icon": "check-circle",
      "text": "{{stats.data.[0].completed}}",
      "label": "Completed",
      "color": "#F0FDF4",
      "icon_color": "#16A34A",
      "font_size": 16,
      "font_weight": "bold"
    },
    {
      "icon": "alert-circle",
      "text": "{{stats.data.[0].overdue}}",
      "label": "Overdue",
      "color": "#FEE2E2",
      "icon_color": "#DC2626",
      "font_size": 16,
      "font_weight": "bold"
    }
  ]
}
```

Properties:
- `columns` — number of cards per row (typically 3 or 4)
- `element_count` — total number of cards
- Each card: `icon` (Lucide name), `text` (the value), `label` (subtitle), `color` (bg), `icon_color`

### Stats query pattern
```sql
SELECT
  COUNT(*)::TEXT as total,
  COUNT(CASE WHEN status = 'pending' THEN 1 END)::TEXT as pending,
  COUNT(CASE WHEN status = 'completed' THEN 1 END)::TEXT as completed,
  COUNT(CASE WHEN due_date < NOW() AND status != 'completed' THEN 1 END)::TEXT as overdue
FROM items
WHERE deleted_at IS NULL
```

Always cast to `::TEXT` for template compatibility.

## Single Counter

```json
{
  "type": "counter",
  "width": 3,
  "element_id": "counter_revenue",
  "icon": "dollar-sign",
  "text": "{{stats.data.[0].revenue}}",
  "label": "Total Revenue",
  "color": "#F0FDF4",
  "icon_color": "#16A34A",
  "font_size": 20,
  "font_weight": "bold"
}
```

## Donut Chart

```json
{
  "type": "donut_chart",
  "width": 6,
  "element_id": "status_donut",
  "data": "status_breakdown.data",
  "label_field": "status",
  "value_field": "count",
  "colors": ["#3B82F6", "#F59E0B", "#10B981", "#EF4444"]
}
```

### Query for donut chart
```sql
SELECT status as status, COUNT(*)::INTEGER as count
FROM items
WHERE deleted_at IS NULL
GROUP BY status
ORDER BY count DESC
```

## Half Donut Chart (Gauge)

```json
{
  "type": "half_donut_chart",
  "width": 6,
  "element_id": "completion_gauge",
  "data": "completion.data",
  "label_field": "label",
  "value_field": "value"
}
```

## Bar Chart

```json
{
  "type": "bar_chart",
  "width": 12,
  "element_id": "monthly_chart",
  "data": "monthly_stats.data",
  "category_field": "month",
  "value_field": "count"
}
```

### Query for bar chart (monthly)
```sql
SELECT TO_CHAR(created_at, 'Mon') as month, COUNT(*) as count
FROM items
WHERE created_at >= NOW() - INTERVAL '6 months' AND deleted_at IS NULL
GROUP BY TO_CHAR(created_at, 'Mon'), EXTRACT(MONTH FROM created_at)
ORDER BY EXTRACT(MONTH FROM created_at)
```

## Dashboard Layout Pattern

Typical dashboard structure: KPI row → charts row → recent items

```json
{
  "ui": [
    /* Sidebar */
    { "type": "sidebar", ... },
    /* Page body */
    {
      "type": "container", "width": 12, "color": "#F8FAFC", "element_id": "dashboard_body",
      "padding_top": 24, "padding_left": 32, "padding_right": 32, "padding_bottom": 32,
      "ui": [
        /* Page title */
        { "text": "Dashboard", "type": "text", "color": "#111827", "font_size": 24, "font_weight": "700", "element_id": "dash_title" },
        { "type": "space_widget", "space": 24, "element_id": "sp_1" },

        /* KPI Cards */
        { "type": "grouped_counter_1", "width": 12, "columns": 4, "element_count": 4, "element_id": "kpis", "cards": [...] },
        { "type": "space_widget", "space": 24, "element_id": "sp_2" },

        /* Charts row */
        {
          "type": "row", "element_id": "charts_row",
          "ui": [
            { "type": "container", "width": 6, "color": "#FFFFFF", "border_color": "#E5E7EB", "border_width": 1, "border_radius": 16,
              "padding_top": 20, "padding_left": 20, "padding_right": 20, "padding_bottom": 20, "element_id": "chart_card_1",
              "ui": [
                { "text": "By Status", "type": "text", "color": "#111827", "font_size": 16, "font_weight": "600", "element_id": "chart_title_1" },
                { "type": "space_widget", "space": 16, "element_id": "sp_chart_1" },
                { "type": "donut_chart", "width": 12, "element_id": "donut_status", "data": "status_breakdown.data", "label_field": "status", "value_field": "count" }
              ]
            },
            { "type": "gutter", "element_id": "chart_gutter" },
            { "type": "container", "width": 6, "color": "#FFFFFF", "border_color": "#E5E7EB", "border_width": 1, "border_radius": 16,
              "padding_top": 20, "padding_left": 20, "padding_right": 20, "padding_bottom": 20, "element_id": "chart_card_2",
              "ui": [
                { "text": "Monthly Trend", "type": "text", "color": "#111827", "font_size": 16, "font_weight": "600", "element_id": "chart_title_2" },
                { "type": "space_widget", "space": 16, "element_id": "sp_chart_2" },
                { "type": "bar_chart", "width": 12, "element_id": "bar_monthly", "data": "monthly_stats.data", "category_field": "month", "value_field": "count" }
              ]
            }
          ]
        },
        { "type": "space_widget", "space": 24, "element_id": "sp_3" },

        /* Recent items */
        { "type": "container", "width": 12, "color": "#FFFFFF", "border_color": "#E5E7EB", "border_width": 1, "border_radius": 16,
          "padding_top": 20, "padding_left": 20, "padding_right": 20, "padding_bottom": 20, "element_id": "recent_card",
          "ui": [
            { "text": "Recent Items", "type": "text", "color": "#111827", "font_size": 16, "font_weight": "600", "element_id": "recent_title" },
            { "type": "space_widget", "space": 16, "element_id": "sp_recent" },
            { "type": "table", "width": 12, "data": "recent_items.data", "element_id": "recent_table", "columns": [...] }
          ]
        }
      ]
    }
  ],
  "data": {
    "stats": "SELECT COUNT(*)::TEXT as total, ... FROM items WHERE deleted_at IS NULL",
    "status_breakdown": "SELECT status, COUNT(*)::INTEGER as count FROM items WHERE deleted_at IS NULL GROUP BY status",
    "monthly_stats": "SELECT TO_CHAR(created_at, 'Mon') as month, COUNT(*) as count FROM items WHERE created_at >= NOW() - INTERVAL '6 months' GROUP BY 1,EXTRACT(MONTH FROM created_at) ORDER BY EXTRACT(MONTH FROM created_at)",
    "recent_items": "SELECT id, title, status, TO_CHAR(created_at, 'DD Mon YYYY') as created_date FROM items WHERE deleted_at IS NULL ORDER BY created_at DESC LIMIT 10"
  }
}
```

## Custom KPI Card (manual layout)

When `grouped_counter_1` doesn't give enough control, build KPI cards manually:

```json
{
  "type": "container", "width": 3, "color": "#FFFFFF", "border_color": "#E5E7EB", "border_width": 1,
  "border_radius": 12, "padding_top": 20, "padding_left": 20, "padding_right": 20, "padding_bottom": 20,
  "element_id": "custom_kpi",
  "ui": [
    {
      "type": "row", "element_id": "kpi_row", "cross_axis_alignment": "center",
      "ui": [
        { "type": "container", "color": "#EFF6FF", "padding_top": 10, "padding_left": 10, "padding_right": 10, "padding_bottom": 10,
          "border_radius": 8, "element_id": "kpi_icon_bg",
          "ui": [{ "icon": "trending-up", "type": "icon", "color": "#2563EB", "size": 20, "element_id": "kpi_icon" }] },
        { "type": "space_widget", "space": 12, "alignment": "horizontal", "element_id": "kpi_sp" },
        { "type": "column", "element_id": "kpi_text_col", "cross_axis_alignment": "start",
          "ui": [
            { "text": "{{stats.data.[0].revenue}}", "type": "text", "color": "#111827", "font_size": 24, "font_weight": "700", "element_id": "kpi_val" },
            { "text": "Total Revenue", "type": "text", "color": "#6B7280", "font_size": 12, "element_id": "kpi_label" }
          ]
        }
      ]
    }
  ]
}
```
