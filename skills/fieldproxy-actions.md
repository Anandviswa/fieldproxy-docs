---
name: fieldproxy-actions
description: |
  FieldProxy onclick action types reference. Load this skill when wiring up button clicks, navigation, CRUD operations, popups, notifications, or any interactive behavior in FieldProxy views.
---

# FieldProxy Actions — Tier 2 Skill

## Action Chain Format

Actions are numbered steps in an `onclick` object, executing sequentially:

```json
"onclick": {
  "1": { "type": "...", "element_id": "..." },
  "2": { "type": "...", "element_id": "..." },
  "3": { "type": "...", "element_id": "..." }
}
```

CRITICAL: Keys MUST be pure numeric strings: `"1"`, `"2"`, `"3"`. Never `"1a"`, `"1b"`, or non-sequential.

## All Action Types

### navigation

Navigate to another view, external URL, or go back.

```json
{
  "type": "navigation",
  "app_id": "APP_ID",
  "view_id": "target_view",
  "arguments": { "id": "{{rowData.id}}", "name": "{{rowData.name}}" },
  "navigation_type": "view",
  "element_id": "nav_detail"
}
```

| Property | Values | Description |
|----------|--------|-------------|
| `navigation_type` | `"view"` | Navigate to another app view |
| `navigation_type` | `"url"` | Open external URL |
| `navigation_type` | `"back"` | Go back (pop) |
| `replace` | `true` | Replace current view in stack (no back) |
| `arguments` | `{...}` | Key-value pairs passed to target view |
| `app_id` | string | Target app (required for cross-app nav) |
| `view_id` | string | Target view within the app |

### execute_query / query

Run a raw SQL query (INSERT, UPDATE, DELETE, SELECT):

```json
{
  "type": "execute_query",
  "query": "UPDATE items SET status = 'closed' WHERE id = {{arguments.id}}",
  "element_id": "q_update_status"
}
```

Results stored in `onclickResults` and accessible via `{{functions.N.field}}` where N is the step number.

### api

Make an external HTTP API call:

```json
{
  "type": "api",
  "method": "POST",
  "url": "https://api.example.com/webhook",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer {{token}}"
  },
  "body": {
    "item_id": "{{arguments.id}}",
    "status": "{{input_data.inp_status.0}}"
  },
  "element_id": "api_webhook"
}
```

Supports: `GET`, `POST`, `PUT`, `DELETE`.

### popup

Open a modal dialog with arbitrary UI:

```json
{
  "type": "popup",
  "title": "Edit Item",
  "width": 600,
  "element_id": "popup_edit",
  "ui": [
    /* Any widgets: inputs, text, buttons, etc. */
  ]
}
```

| Property | Description |
|----------|-------------|
| `width` | Popup width in pixels (e.g. 480, 600) |
| `title` | Header text |
| `fullscreen` | `true` for full-screen modal |
| `auto_close` | `true` to auto-close after delay |
| `delay` | Auto-close delay in seconds |
| `ui` | Array of widget objects |

### close_popup

Close the current popup:

```json
{ "type": "close_popup", "element_id": "close_popup_1" }
```

### toast

Show a temporary notification message:

```json
{
  "type": "toast",
  "message": "Item updated successfully!",
  "toast_type": "success",
  "duration": "3",
  "element_id": "toast_success"
}
```

`toast_type` values: `success`, `warning`, `info`, `error`
`duration`: seconds as string (e.g. `"2"`, `"3"`, `"5"`)

### snackbar

Show a snackbar notification (more options than toast):

**Standard:**
```json
{ "type": "snackbar", "message": "Processing...", "element_id": "snack_1" }
```

**Persistent (with dismiss):**
```json
{ "type": "snackbar", "message": "Saving...", "persistent": true, "element_id": "snack_persist" }
```

**With action button:**
```json
{
  "type": "snackbar",
  "message": "Item archived",
  "action_text": "Undo",
  "action_onclick": {
    "1": { "type": "execute_query", "query": "UPDATE items SET archived = false WHERE id = {{arguments.id}}", "element_id": "q_undo" },
    "2": { "type": "refresh_data", "element_id": "refresh_undo" }
  },
  "element_id": "snack_undo"
}
```

**Loading:**
```json
{ "type": "snackbar", "message": "Loading...", "loading": true, "element_id": "snack_loading" }
```

### confirm

Show a confirmation dialog before proceeding:

```json
{
  "type": "confirm",
  "title": "Delete Item?",
  "message": "This action cannot be undone. Are you sure you want to delete this item?",
  "confirm_text": "Yes, Delete",
  "cancel_text": "Cancel",
  "element_id": "confirm_delete"
}
```

If the user clicks Cancel, the entire action chain stops. If they click Confirm, execution continues to the next numbered step.

### select

Set a value in the `selectedElements` reactive map:

```json
{
  "type": "select",
  "select_type": "single",
  "element_id": "select_item",
  "value": "{{data.[index].id}}"
}
```

`select_type`: `"single"` (replaces) or `"multi"` (toggles in array).
Access selected: `{{selectedElements.select_item}}`.

### update_value

Programmatically set the text of an input field:

```json
{
  "type": "update_value",
  "target_element_id": "inp_name",
  "value": "{{detail.data.[0].name}}",
  "element_id": "update_name"
}
```

Useful for pre-filling edit forms from query data.

### update_data

Re-fetch a specific data query by key:

```json
{
  "type": "update_data",
  "data_key": "items_query",
  "element_id": "update_items_data"
}
```

Re-runs only the specified query, not all queries on the view.

### refresh_data

Re-fetch ALL data queries on the current view:

```json
{ "type": "refresh_data", "element_id": "refresh_all" }
```

Use at the end of CRUD action chains to update the UI.

### notification (email)

Send an email notification:

```json
{
  "type": "notification",
  "notification_type": "email",
  "to": "{{detail.data.[0].email}}",
  "subject": "Update on item #{{arguments.id}}",
  "body": "Your item status has been updated to {{input_data.inp_status.0}}.",
  "element_id": "notify_email"
}
```

### notification (app)

Send an in-app push notification to specific users:

```json
{
  "type": "notification",
  "notification_type": "app",
  "user_ids": ["{{detail.data.[0].assigned_user_id}}"],
  "title": "New Assignment",
  "message": "You have been assigned item #{{arguments.id}}",
  "element_id": "notify_app"
}
```

## Conditional Actions

Actions support a `condition` field — a SQL query that must return true for the action to execute:

```json
{
  "1": {
    "type": "execute_query",
    "query": "UPDATE items SET status = 'closed' WHERE id = {{arguments.id}}",
    "condition": "SELECT CASE WHEN status = 'open' THEN true ELSE false END FROM items WHERE id = {{arguments.id}}",
    "element_id": "q_conditional_update"
  }
}
```

## Common Action Chains

### Create (from popup)
```
"1": execute_query (INSERT)
"2": close_popup
"3": toast (success)
"4": refresh_data
```

### Update (from popup)
```
"1": execute_query (UPDATE)
"2": execute_query (audit log INSERT) — optional
"3": close_popup
"4": toast (success)
"5": refresh_data
```

### Delete (with confirmation)
```
"1": confirm ("Are you sure?")
"2": execute_query (soft DELETE: SET deleted_at = NOW())
"3": toast (success)
"4": navigation (back) OR refresh_data
```

### Status change (inline)
```
"1": execute_query (UPDATE status)
"2": execute_query (audit log)
"3": toast (success)
"4": refresh_data
```

## Button Types Quick Reference

| btn_type | Use for | Example |
|----------|---------|---------|
| `filled_button` | Primary actions (Save, Create, Submit) | Blue bg, white text |
| `outlined_button` | Secondary actions (Edit, Export) | Border, no fill |
| `text_button` | Tertiary actions (Cancel, Back) | No border, text only |
| `icon_button` | Compact actions (avatar, circular) | Icon in circle |
| `dropdown_button` | Multi-action menus (Quick Actions) | Menu with items |

### Dropdown button with per-item actions
```json
{
  "type": "button",
  "btn_type": "dropdown_button",
  "width": 2,
  "icon": "zap",
  "text": "Quick Actions",
  "text_color": "#FFFFFF",
  "font_weight": "600",
  "border_radius": 8,
  "background_color": "#374151",
  "element_id": "btn_quick_actions",
  "items": [
    {
      "icon": "edit",
      "text": "Edit",
      "value": "edit",
      "onclick": { "1": { "type": "popup", "element_id": "popup_edit", "ui": [...] } }
    },
    {
      "icon": "mail",
      "text": "Send Email",
      "value": "email",
      "onclick": { "1": { "type": "notification", "notification_type": "email", ... } }
    }
  ]
}
```
