---
name: fieldproxy-input-forms
description: |
  FieldProxy input types, form patterns, popup forms, and CRUD action chains. Load this skill when building forms, popup dialogs, edit/create flows, or any view that collects user input.
---

# FieldProxy Input & Forms — Tier 2 Skill

## All Input Types — Properties & Stored Values

### text
```json
{ "type": "input", "input_type": "text", "label": "Notes", "placeholder": "Enter notes...", "max_lines": 3, "element_id": "inp_notes" }
```
Stored: `String` — sanitized text (< > stripped). Add `max_lines: N` for multi-line (NOT `textarea`).

### number
```json
{ "type": "input", "input_type": "number", "label": "Amount", "placeholder": "0.00", "element_id": "inp_amount" }
```
Stored: `String` of double — e.g. `"42.0"`. Supports `min`, `max` validation. Value removed if out of range.

### email
```json
{ "type": "input", "input_type": "email", "label": "Email", "placeholder": "user@example.com", "element_id": "inp_email" }
```
Stored: `String` — only stored if valid email format. Key removed if invalid.

### phone
```json
{ "type": "input", "input_type": "phone", "label": "Phone", "element_id": "inp_phone" }
```
Stored: `String` — international format e.g. `"+44 7911 123456"`.

### date
```json
{ "type": "input", "input_type": "date", "label": "Due Date", "element_id": "inp_date" }
```
Stored: `String` — date string per format mask e.g. `"2024-01-15"`.

For **date range**: stored as `List<String>` with 2 ISO strings.
Access: `{{input_data.element_id.0.[0]}}` (start), `{{input_data.element_id.0.[1]}}` (end).

### dropdown (single)
```json
{
  "type": "input", "input_type": "dropdown", "label": "Status",
  "data": [
    { "label": "Open", "value": "open" },
    { "label": "Closed", "value": "closed" }
  ],
  "store": ["value"],
  "show_key": false,
  "placeholder": "Select status",
  "element_id": "inp_status"
}
```
Stored with `store: ["value"]`: the extracted field value (e.g. `"open"`).
Stored without `store`: the raw selected item (String or Map).

IMPORTANT: Always add `show_key: false` when using object data (label/value pairs).

### dropdown (multi-select)
```json
{
  "type": "input", "input_type": "dropdown", "label": "Tags",
  "data": [...],
  "is_multi": true,
  "store": ["value"],
  "show_key": false,
  "element_id": "inp_tags"
}
```
Stored: `String` with `{}` braces — e.g. `"{tag1, tag2, tag3}"`. Use directly in PostgreSQL `ANY()` or `IN`.

### checkbox
```json
{ "type": "input", "input_type": "checkbox", "label": "Agree to terms", "element_id": "inp_agree" }
```
Stored: `bool` — `true` or `false`. In templates resolves to string `"true"` / `"false"`.

### mcq (radio — single select)
```json
{
  "type": "input", "input_type": "mcq", "label": "Priority",
  "data": ["Low", "Medium", "High", "Urgent"],
  "element_id": "inp_priority"
}
```
Stored: `List<String>` with 1 item — e.g. `["High"]`.

### mcq (multi-select)
```json
{
  "type": "input", "input_type": "mcq", "label": "Features",
  "data": ["Feature A", "Feature B", "Feature C"],
  "is_multi": true,
  "element_id": "inp_features"
}
```
Stored: `List<String>` — e.g. `["Feature A", "Feature C"]`.

### mcq (checklist)
```json
{
  "type": "input", "input_type": "mcq", "label": "Checklist",
  "data": ["Step 1", "Step 2", "Step 3"],
  "is_checklist": true,
  "element_id": "inp_checklist"
}
```
Stored: `Map` — `{"value": {"Step 1": true, "Step 2": false}, "formatted_value": {...}}`.

### rating
```json
{ "type": "input", "input_type": "rating", "label": "Rating", "element_id": "inp_rating" }
```
Stored: `String` of double — e.g. `"4.5"`.

### step_counter
```json
{ "type": "input", "input_type": "step_counter", "label": "Quantity", "element_id": "inp_qty" }
```
Stored: `String` — e.g. `"5"`.

### file
```json
{ "type": "input", "input_type": "file", "label": "Attachments", "element_id": "inp_files" }
```
Stored: `Map` — `{"path": [{"path": "...", "url": "..."}], "url": ["https://..."]}`.
Access URLs: `{{input_data.inp_files.0.url}}` (list) or `{{input_data.inp_files.0.url.[0]}}` (first).

### image
Same structure as file. Stored: `Map` — `{"path": [...], "url": [...]}`.

### signature
```json
{ "type": "input", "input_type": "signature", "label": "Signature", "element_id": "inp_sig" }
```
Stored: `Map` — `{"path": "...", "url": "..."}` (flat, not array).
Access: `{{input_data.inp_sig.0.url}}`.

### location
```json
{ "type": "input", "input_type": "location", "label": "Location", "element_id": "inp_loc" }
```
Stored: `Map` — `{"coordinates": [longitude, latitude], "address": "Full address"}`.
Access: `{{input_data.inp_loc.0.address}}`, `{{input_data.inp_loc.0.coordinates.[0]}}` (lng), `.[1]` (lat).

### scan
```json
{ "type": "input", "input_type": "scan", "label": "Scan Code", "element_id": "inp_scan" }
```
Stored: `String` — raw scanned string.

### table (editable)
```json
{ "type": "input", "input_type": "table", "label": "Line Items", "element_id": "inp_table" }
```

### catalogue
```json
{ "type": "input", "input_type": "catalogue", "label": "Products", "element_id": "inp_catalogue" }
```

## Popup Form Pattern

```json
{
  "type": "popup",
  "title": "Create Item",
  "width": 480,
  "element_id": "popup_create",
  "ui": [
    { "type": "input", "input_type": "text", "label": "Title", "placeholder": "Enter title", "element_id": "inp_title" },
    { "type": "space_widget", "space": 16, "element_id": "sp_1" },
    { "type": "input", "input_type": "dropdown", "label": "Status",
      "data": [{ "label": "Open", "value": "open" }, { "label": "Closed", "value": "closed" }],
      "store": ["value"], "show_key": false, "placeholder": "Select", "element_id": "inp_popup_status" },
    { "type": "space_widget", "space": 16, "element_id": "sp_2" },
    { "type": "input", "input_type": "text", "label": "Description", "max_lines": 4, "placeholder": "Details...", "element_id": "inp_desc" },
    { "type": "space_widget", "space": 24, "element_id": "sp_3" },
    {
      "type": "row",
      "element_id": "popup_btn_row",
      "cross_axis_alignment": "center",
      "ui": [
        { "type": "spacer", "element_id": "popup_sp" },
        { "type": "button", "btn_type": "text_button", "text": "Cancel", "font_size": 13, "text_color": "#6B7280", "element_id": "btn_cancel",
          "onclick": { "1": { "type": "close_popup", "element_id": "close_cancel" } } },
        { "type": "space_widget", "space": 12, "alignment": "horizontal", "element_id": "sp_btns" },
        { "type": "button", "btn_type": "filled_button", "width": 3, "text": "Create", "font_size": 14, "text_color": "#FFFFFF", "font_weight": "600",
          "border_radius": 8, "background_color": "#2563EB", "element_id": "btn_confirm",
          "onclick": {
            "1": { "type": "execute_query", "query": "INSERT INTO items (title, status, description) VALUES ('{{input_data.inp_title.0}}', '{{input_data.inp_popup_status.0}}', '{{input_data.inp_desc.0}}')", "element_id": "q_insert" },
            "2": { "type": "close_popup", "element_id": "close_done" },
            "3": { "type": "toast", "message": "Item created!", "toast_type": "success", "duration": "3", "element_id": "toast_ok" },
            "4": { "type": "refresh_data", "element_id": "refresh_all" }
          }
        }
      ]
    }
  ]
}
```

## CRUD Action Chain Pattern

Standard sequence (numbered keys, always sequential):

```
"1": execute_query (INSERT/UPDATE/DELETE)
"2": execute_query (audit log INSERT) — optional
"3": close_popup (if inside popup)
"4": toast (success/warning/info)
"5": refresh_data
```

### Audit log pattern
```sql
INSERT INTO audit_log (entity_id, user_id, action_type, field_changed, new_value, source, created_at)
VALUES ({{arguments.id}}, '<!getUserId()!>', 'update', 'status', '{{input_data.inp_status.0}}', 'manual', NOW())
```

### Delete with confirmation
```json
"onclick": {
  "1": { "type": "confirm", "title": "Delete?", "message": "This cannot be undone.", "confirm_text": "Delete", "cancel_text": "Cancel", "element_id": "confirm_del" },
  "2": { "type": "execute_query", "query": "UPDATE items SET deleted_at = NOW() WHERE id = {{arguments.id}}", "element_id": "q_delete" },
  "3": { "type": "toast", "message": "Deleted", "toast_type": "success", "duration": "2", "element_id": "toast_del" },
  "4": { "type": "navigation", "navigation_type": "back", "element_id": "nav_back" }
}
```

## Spacing Conventions for Forms

| Context | space value |
|---------|------------|
| Between form fields | 16 |
| Last field → button row | 24 |
| Between buttons (horizontal) | 12 |

## combo_input (Dropdown + Text in one row)

```json
{ "type": "combo_input", "element_id": "combo_1", ... }
```
Combined dropdown selector + text input for inline entry patterns.

## Input Data Access Summary

| Input type | Stored type | Template pattern |
|-----------|------------|-----------------|
| text, number, email, phone, scan, rating, step_counter | `String` | `{{input_data.id.0}}` |
| date (single) | `String` | `{{input_data.id.0}}` |
| date (range) | `List[2]` | `{{input_data.id.0.[0]}}` / `.[1]` |
| dropdown (single + store) | `String` | `{{input_data.id.0}}` |
| dropdown (multi) | `String` `{a,b}` | `{{input_data.id.0}}` |
| checkbox | `bool` | `{{input_data.id.0}}` → `"true"/"false"` |
| mcq (radio) | `List[1]` | `{{input_data.id.0}}` |
| mcq (multi) | `List[N]` | `{{input_data.id.0}}` |
| file, image | `Map{url:[]}` | `{{input_data.id.0.url.[0]}}` |
| signature | `Map{url:""}` | `{{input_data.id.0.url}}` |
| location | `Map{addr,coords}` | `{{input_data.id.0.address}}` |
