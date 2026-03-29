---
name: fieldproxy-layout-patterns
description: |
  FieldProxy layout patterns for building page structures, cards, multi-column layouts, headers, badges, and spacing. Load this skill when building page layouts, detail pages, dashboards, or any view with complex container nesting.
---

# FieldProxy Layout Patterns — Tier 2 Skill

## Page Structure Skeleton

```
sidebar → page_body (container width:12) → [header_banner, main_content]
```

### Page body (zero-padding wrapper)
```json
{
  "type": "container",
  "width": 12,
  "color": "#FFFFFF",
  "element_id": "page_body",
  "padding_top": 0, "padding_left": 0, "padding_right": 0, "padding_bottom": 0,
  "ui": [ /* header_banner, main_content */ ]
}
```

### Header banner (with bottom separator)
```json
{
  "type": "container",
  "width": 12,
  "element_id": "header_banner",
  "border_color": "#E5E7EB",
  "border_width": 1,
  "padding_top": 16, "padding_left": 32, "padding_right": 32, "padding_bottom": 16,
  "ui": [ /* header row */ ]
}
```

### Main content (gray background, padded)
```json
{
  "type": "container",
  "width": 12,
  "color": "#F8FAFC",
  "element_id": "main_content",
  "padding_top": 28, "padding_left": 32, "padding_right": 32, "padding_bottom": 36,
  "ui": [ /* cards, tab groups, lists */ ]
}
```

## Header Bar Pattern

Row with back button, title, spacer, and action items:

```json
{
  "type": "row",
  "element_id": "header_row",
  "cross_axis_alignment": "center",
  "ui": [
    { "type": "button", "btn_type": "text_button", "text": "← Back", "font_size": 13, "text_color": "#2563EB", "element_id": "btn_back",
      "onclick": { "1": { "type": "navigation", "navigation_type": "back", "element_id": "nav_back" } } },
    { "type": "space_widget", "space": 8, "alignment": "horizontal", "element_id": "sp_1" },
    { "type": "text", "text": "{{detail.data.[0].title}}", "color": "#111827", "font_size": 18, "font_weight": "700", "element_id": "header_title" },
    { "type": "spacer", "element_id": "header_spacer" },
    /* status badge, priority badge, action buttons go here */
  ]
}
```

## White Card

```json
{
  "type": "container",
  "width": 12,
  "color": "#FFFFFF",
  "border_color": "#E5E7EB",
  "border_width": 1,
  "border_radius": 16,
  "padding_top": 28, "padding_left": 28, "padding_right": 28, "padding_bottom": 28,
  "element_id": "my_card",
  "ui": [ /* card content */ ]
}
```

## Multi-Column Layouts

### Two columns with gutter
```json
{
  "type": "row",
  "element_id": "two_col_row",
  "cross_axis_alignment": "stretch",
  "ui": [
    { "type": "container", "width": 6.5, "element_id": "col_left", "color": "transparent", "ui": [...] },
    { "type": "gutter", "element_id": "gutter_1" },
    { "type": "container", "width": 4.5, "element_id": "col_right", "color": "transparent", "ui": [...] }
  ]
}
```

### Three columns with gutters + dividers

CRITICAL: Three columns at `width: 4` each + 2 gutters OVERFLOWS. Use `width: 3.8` max.

```json
{
  "type": "row",
  "element_id": "three_col_row",
  "cross_axis_alignment": "start",
  "ui": [
    { "type": "container", "width": 3.8, "element_id": "col_1", "color": "transparent", "ui": [...] },
    { "type": "container", "element_id": "divider_1", "color": "#E5E7EB", "width": 0.01, "height": 400 },
    { "type": "gutter", "element_id": "gutter_1" },
    { "type": "container", "width": 3.8, "element_id": "col_2", "color": "transparent", "ui": [...] },
    { "type": "container", "element_id": "divider_2", "color": "#E5E7EB", "width": 0.01, "height": 400 },
    { "type": "gutter", "element_id": "gutter_2" },
    { "type": "container", "width": 3.8, "element_id": "col_3", "color": "transparent", "ui": [...] }
  ]
}
```

## Label-Value Display Pattern

### Section heading
```json
{ "text": "SECTION NAME", "type": "text", "color": "#2563EB", "font_size": 11, "font_weight": "700", "element_id": "sect_heading" }
```

### Sub-label + value pair
```json
{ "text": "Field Label", "type": "text", "color": "#9CA3AF", "font_size": 11, "element_id": "lbl_field" },
{ "type": "space_widget", "space": 4, "element_id": "sp_lbl" },
{
  "type": "row",
  "element_id": "val_field_row",
  "main_axis_alignment": "start",
  "cross_axis_alignment": "center",
  "ui": [
    { "icon": "user", "type": "icon", "color": "#6B7280", "size": 16, "element_id": "icon_field" },
    { "type": "space_widget", "space": 8, "alignment": "horizontal", "element_id": "sp_icon" },
    { "text": "{{detail.data.[0].field}}", "type": "text", "color": "#111827", "font_size": 14, "font_weight": "600", "element_id": "val_field" }
  ]
}
```

Between field groups: `space_widget` with `space: 16-18`

## Badge / Pill Patterns

### Status badge (colored background)
```json
{
  "type": "container",
  "color": "#F0FDF4",
  "padding_top": 5, "padding_left": 14, "padding_right": 14, "padding_bottom": 5,
  "border_radius": 20,
  "element_id": "badge_status",
  "ui": [{
    "type": "row", "element_id": "badge_row", "main_axis_alignment": "start", "main_axis_size": "min",
    "ui": [
      { "text": "{{detail.data.[0].status}}", "type": "text", "color": "#16A34A", "font_size": 12, "font_weight": "600", "element_id": "badge_txt" }
    ]
  }]
}
```

### Badge with icon + text
```json
{
  "type": "container",
  "color": "#ECFDF5",
  "border_color": "#BBF7D0", "border_width": 1,
  "padding_top": 6, "padding_left": 14, "padding_right": 14, "padding_bottom": 6,
  "border_radius": 20,
  "element_id": "badge_sla",
  "ui": [{
    "type": "row", "element_id": "badge_sla_row", "main_axis_alignment": "start", "main_axis_size": "min", "cross_axis_alignment": "center",
    "ui": [
      { "icon": "clock", "type": "icon", "color": "#059669", "size": 14, "element_id": "badge_sla_icon" },
      { "type": "space_widget", "space": 6, "alignment": "horizontal", "element_id": "sp_badge" },
      { "text": "{{detail.data.[0].sla}}", "type": "text", "color": "#059669", "font_size": 13, "font_weight": "600", "element_id": "badge_sla_txt" }
    ]
  }]
}
```

### Clickable pill (opens popup)
Container with `border_radius: 20`, inner row with `main_axis_size: "min"`, and `onclick` that triggers a popup. Includes chevron-down icon for affordance.

## Avatar Pattern

```json
{
  "type": "container",
  "color": "#2563EB",
  "padding_top": 5, "padding_left": 8, "padding_right": 8, "padding_bottom": 5,
  "border_radius": 50,
  "element_id": "avatar",
  "ui": [{ "text": "{{data.[index].initial}}", "type": "text", "color": "#FFFFFF", "font_size": 12, "font_weight": "700", "element_id": "avatar_txt" }]
}
```

## Tab Group

```json
{
  "type": "tab_group",
  "width": 12,
  "element_id": "main_tabs",
  "ui": [
    { "title": "Details", "ui": [ /* tab 1 content */ ] },
    { "title": "Activity", "ui": [ /* tab 2 content */ ] },
    { "title": "Notes", "ui": [ /* tab 3 content */ ] }
  ]
}
```

## Row Alignment Quick Reference

| Goal | Properties |
|------|-----------|
| Left-aligned, shrink-wrap | `main_axis_alignment: "start"`, `main_axis_size: "min"` |
| Space between | `main_axis_alignment: "space_between"` |
| Right-aligned buttons | `spacer` + buttons in row |
| Vertically centered | `cross_axis_alignment: "center"` |
| Top-aligned columns | `cross_axis_alignment: "start"` |
| Equal-height columns | `cross_axis_alignment: "stretch"` |

Row default is `spaceBetween` — always set `main_axis_alignment: "start"` for normal rows.

## Color System

| Role | Background | Text | Border |
|------|-----------|------|--------|
| Page background | #F8FAFC | — | — |
| White card | #FFFFFF | — | #E5E7EB |
| Blue accent | #EFF6FF | #2563EB / #1D4ED8 | — |
| Green success | #F0FDF4 / #ECFDF5 | #16A34A / #059669 | #BBF7D0 |
| Yellow/warning | #FFFBEB / #FEF3C7 | #D97706 / #92400E | #FDE68A |
| Red/danger | #FEE2E2 | #DC2626 | — |
| Purple/notes | #F5F3FF | #7C3AED | — |
| Gray labels | — | #9CA3AF (sub) / #6B7280 (icons) | — |
| Dark text | — | #111827 (primary) / #374151 (secondary) | — |
| Divider | #E5E7EB / #F1F5F9 | — | — |

## Spacing Conventions

| Context | space value |
|---------|------------|
| Section title → first field | 18 |
| Sub-label → value | 4-6 |
| Between field groups | 16-18 |
| Card internal padding | 20-28 |
| Between cards (vertical) | 20-24 |
| Horizontal icon → text | 6-10 |
| Header elements horizontal | 8-12 |
