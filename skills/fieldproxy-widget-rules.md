---
name: fieldproxy-widget-rules
description: |
  FieldProxy layout debug rules and validation checklist. Load this skill BEFORE outputting any FieldProxy view JSON, or when debugging layout issues. Contains 15 verified rules extracted from source code that prevent common rendering bugs.
---

# FieldProxy Widget Rules — Validation & Debug Skill (Tier 1)

> Run this checklist before outputting any view JSON. Every rule is verified against source code.

## RULE 1 — `column` and `row` ignore their own `width`

column/row render as Flutter Column/Row — width is never applied to themselves.

```json
// WRONG
{ "type": "column", "width": 10, "ui": [...] }
{ "type": "row",    "width": 10, "ui": [...] }

// CORRECT — wrap in container
{ "type": "container", "width": 10, "element_id": "wrapper", "ui": [
    { "type": "column", "element_id": "col_inner", "ui": [...] }
]}
```

## RULE 2 — Width is always absolute (against screenWidth)

`calculateDynamicWidth` ALWAYS uses screenWidth as reference — not the parent's width. Nested elements don't inherit parent width.

Elements that must align should be at the SAME nesting level with matching width values.

**Nested overflow rule:** When children are inside a container with `width: N`, the sum of all children widths must be ≤ N. Since widths are absolute against screenWidth, children summing to 12 (100% screen) inside a `width: 7.5` parent will overflow by 37.5% — causing items to be clipped/hidden. Scale children proportionally to fit the parent.

**Combining with Rule 1:** `column`/`row` children inside a row will NOT have their width applied (Rule 1). To give a column or row child a fixed width inside a parent row, wrap it in a `container` with the desired width, then nest the column/row inside. Otherwise the child takes its natural content width, not the specified grid width.

## RULE 3 — CSS `border` syntax doesn't work

The renderer reads `border_color` and `border_width` separately. The CSS `border` string is ignored.

```json
// WRONG
{ "border": "1px solid #E5E7EB" }

// CORRECT
{ "border_color": "#E5E7EB", "border_width": 1 }
```

## RULE 4 — `expanded: true` only works on `text`

Only the "text" case in `getChildren()` checks `expanded`. On buttons, containers, icons — silently ignored.

```json
// WRONG
{ "type": "button", "expanded": true }

// CORRECT alternatives
{ "type": "text",   "expanded": true, "element_id": "txt" }
{ "type": "button", "width": 2, "element_id": "btn" }
{ "type": "spacer", "element_id": "gap" }
```

## RULE 5 — `space_widget` default alignment is vertical

Default is `SizedBox(height: N)`. Inside a row, a vertical spacer adds height, not horizontal gap.

```json
// WRONG — inside a row this does nothing visible
{ "type": "space_widget", "space": 16, "element_id": "sp" }

// CORRECT — inside a row
{ "type": "space_widget", "space": 16, "alignment": "horizontal", "element_id": "sp" }
```

## RULE 6 — Every element MUST have a unique `element_id`

Missing element_id → fallback `unknown_<timestamp>` → rebuild flicker, broken visibility conditions, state collision.

Forbidden element_id values: `index`, `[index]`, `[0]`, `0`, or anything containing `.` (dots).

**When modifying an existing file:** New elements must be checked against ALL existing element_ids in the file, not just other new elements. Duplicate IDs across old + new elements cause the same state collision bugs. Before adding any element_id, search the full file for that string.

## RULE 7 — Centering in a row needs TWO spacers

One spacer pushes element right. Two spacers (before + after) split space 50/50.

```json
// WRONG — pushes to right
{ "type": "spacer" }, { "type": "text", "text": "Title" }

// CORRECT — true center
{ "type": "spacer", "element_id": "sp_l" },
{ "type": "text", "text": "Title", "element_id": "title" },
{ "type": "spacer", "element_id": "sp_r" }
```

## RULE 8 — `onclick` on container auto-triggers hover

ContainerWidget auto-adds shadow + 2px lift + pointer cursor when onclick is present. No way to disable. Only add onclick on containers where full-area hover is intentional.

## RULE 9 — `color: "transparent"` must be explicit

Default container color is `#FFFFFF` (white). Inner containers that should show parent background must set `"color": "transparent"`.

## RULE 10 — `main_axis_size: "min"` on inner rows

Default `main_axis_size` is `"max"` — inner rows stretch full width. Set `"min"` on rows that should only take their content's natural width (pills, badges, icon+text groups).

## RULE 11 — `show_if` does not exist — use `visibility_condition`

The renderer only checks `visibility_condition`. `show_if` is silently ignored — element always visible.

```json
// WRONG
{ "show_if": "'{{data}}' != '0'" }

// CORRECT
{ "visibility_condition": "'{{data}}' != '0'" }
```

## RULE 12 — Template values in visibility conditions MUST be in single quotes

After template resolution, unquoted values cause unreliable comparisons.

```json
// WRONG
{ "visibility_condition": "{{stats.data.[0].count}} != '0'" }

// CORRECT
{ "visibility_condition": "'{{stats.data.[0].count}}' != '0'" }
```

## RULE 13 — Buttons use `width` (grid 1-12) — `fixed_width` does not exist for buttons

`fixed_width` is ContainerWidget-only. Buttons use grid width resolved via `calculateDynamicWidth()`.

```json
// WRONG
{ "type": "button", "fixed_width": 120 }

// CORRECT
{ "type": "button", "width": 2, "element_id": "btn" }
```

Button width reference (1440px screen, 220px sidebar):
- width 1 ≈ 95px | width 2 ≈ 190px | width 3 ≈ 285px | width 4 ≈ 380px

## RULE 14 — Valid `input_type` values — `textarea` and `select` do not exist

Unknown input types produce a null widget — input silently disappears.

```json
// WRONG
{ "input_type": "textarea" }
{ "input_type": "select" }

// CORRECT
{ "input_type": "text", "max_lines": 5 }
{ "input_type": "dropdown", "show_key": false }
```

Valid types: `text`, `number`, `email`, `phone`, `date`, `dropdown`, `checkbox`, `file`, `image`, `mcq`, `rating`, `signature`, `location`, `scan`, `step_counter`, `table`, `catalogue`

## RULE 15 — `input_data` path syntax: `.0` vs `.[index]` vs `.[0]`

| Syntax | Resolves to | Use when |
|--------|-------------|----------|
| `.0` | String key `"0"` in map | Popup/form (non-list) — always row 0 |
| `.[index]` | Runtime row index integer | Inside list — current row |
| `.[0]` | Integer index 0 on a List | Indexing into array values (date range, file URLs) |

NEVER use `.[0]` when you mean `.0` for input_data access.

## RULE 16 — Use `type: "wrap"` (not `type: "row"`) for multi-column layouts inside padded containers

`calculateDynamicWidth()` in `container_widget.dart` **always** calculates from `widget.controller.screenWidth` — never from the parent's actual pixel constraints. This means children inside a padded parent container can overflow even when their grid widths sum to less than the parent's grid width, because the parent's padding reduces available pixels but children don't know about it.

A `type: "row"` (Flutter Row) **silently clips** overflow — hidden items just disappear. A `type: "wrap"` (Flutter Wrap) **wraps** overflow to the next line — items are always visible.

```json
// WRONG — children may overflow and be clipped inside a padded parent
{ "type": "row", "element_id": "data_row", "ui": [
    { "type": "container", "width": 2 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 2 }
]}

// CORRECT — wrap ensures items are always visible
{ "type": "wrap", "element_id": "data_row", "ui": [
    { "type": "container", "width": 2 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 1 },
    { "type": "container", "width": 2 }
]}
```

**When to use `row` vs `wrap`:**
- `row`: 2-3 items that are guaranteed to fit (e.g., label + value), or items using `spacer`/`expanded`
- `wrap`: 4+ items with explicit widths, especially inside padded parent containers

## RULE 17 — `icon_button` uses `bg_color`, `color`, and `size` — NOT `background_color` or `icon_color`

`btn_type: "icon_button"` renders as a Flutter `CircleAvatar` + `IconButton`. The source code reads:
- `bg_color` (default `#FFFFFF`) → CircleAvatar background color
- `color` (default `#FFFFFF`) → icon color
- `size` (default `20`) → both CircleAvatar radius AND icon size

**`background_color`**, **`icon_color`**, **`border_radius`** are all **silently ignored** — the button renders white-on-white (invisible).

```json
// WRONG — properties silently ignored, icon invisible
{
  "type": "button",
  "btn_type": "icon_button",
  "icon": "check-square",
  "background_color": "#E0F2FE",
  "icon_color": "#0891B2"
}

// CORRECT
{
  "type": "button",
  "btn_type": "icon_button",
  "icon": "check-square",
  "bg_color": "#E0F2FE",
  "color": "#0891B2",
  "size": 16
}
```

**Note:** `filled_button` uses `background_color` and `text_color`. `icon_button` uses `bg_color` and `color`. Different property names for different btn_types.

## RULE 18 — List item element_ids must NOT contain `_[index]` — the framework handles row differentiation internally

Inside a `type: "list"`, every child element_id should be a **stable string** (e.g., `"btn_verify"`, `"mach_serial"`). The framework passes the row index separately as a parameter and uses `visibilityConditions[element_id]["$index"]` / `inputData[element_id]["$index"]` to differentiate rows.

**`_[index]` in element_ids is NOT replaced by the framework** — it stays as the literal string `"btn_verify_[index]"` for ALL rows, which:
1. Causes state collision (all rows share the exact same element_id string)
2. Breaks `input_data` access (path segment `verify_notes_[index]` is not just `[index]`, so `getNestedData` tries `int.parse("verify_notes_index")` → crashes silently)

```json
// WRONG — _[index] stays as literal string, collisions + broken input_data
{
  "element_id": "btn_verify_[index]",
  "field_name": "verify_notes_[index]"
}
// query: "{{input_data.verify_notes_[index].0}}"  → FAILS

// CORRECT — stable element_id, framework differentiates via $index internally
{
  "element_id": "btn_verify",
  "field_name": "verify_notes"
}
// query: "{{input_data.verify_notes.[index]}}"  → WORKS
```

**Input data access inside list popups:**
- Non-list popup (index always 0): `input_data.element_id.0`
- List popup (index = row index): `input_data.element_id.[index]`

**Data paths in templates still use `[index]`:**
- `{{ticket_machines.data.[index].serial_number}}` — CORRECT, this is a data path resolved by `getNestedData`
- `element_id: "mach_serial"` — CORRECT, no `_[index]` suffix

---

## RULE 19 — `input_data` is ALWAYS keyed by `element_id`, NEVER by `field_name`

**Critical**: `field_name` is used for form submission/validation labels only. `input_data` in template expressions ALWAYS uses `element_id` as the key.

```json
// Input widget definition:
{ "element_id": "rpl_machine_input", "field_name": "rpl_machine", "input_type": "dropdown" }

// WRONG — field_name as key, always null
"{{input_data.rpl_machine.[index]}}"

// CORRECT — element_id as key
"{{input_data.rpl_machine_input.[index]}}"
```

**Access patterns by input type:**

| Input type | Context | Template |
|---|---|---|
| Text / Date / Number | Normal popup | `{{input_data.ELEMENT_ID.0}}` |
| Text / Date / Number | List popup | `{{input_data.ELEMENT_ID.[index]}}` |
| Dropdown (simple value) | Normal popup | `{{input_data.ELEMENT_ID.0}}` |
| Dropdown (object-data) | List popup | `{{input_data.ELEMENT_ID.[index].FIELD_NAME}}` |

**Confirmed working example** (Waterade replacement_job):
```
element_id: "rpl_machine_input", field_name: "rpl_machine"
Query: {{input_data.rpl_machine_input.[index].machine_id}}  ✓ WORKS
Query: {{input_data.rpl_machine.[index]}}                   ✗ ALWAYS NULL
```

---

## RULE 20 — FieldProxy uses Feather icons — unrecognized names render as star (★) fallback

**Verified icons from working screens:** `map-pin`, `package`, `alert-circle`, `layers`, `clock`, `truck`, `check-circle`, `refresh-cw`, `eye`, `edit`, `edit-3`, `trash-2`, `clipboard`, `users`, `file-text`, `columns`, `alert-triangle`, `check`, `plus-circle`, `user-plus`, `save`, `send`, `info`, `mail`, `inbox`, `x-circle`, `home`, `box`, `briefcase`, `calendar`, `shield`.

**NOT valid:** `building`, `warehouse`, `factory`, `office`. These are NOT in the Feather icon set.

**For facility/building concepts, use:** `home` (house icon).

---

## RULE 21 — Small containers inside rows: do NOT set `width` — let them shrink-wrap

`calculateDynamicWidth()` always uses `screenWidth`. Even `width: 1` = `1/12 × screenWidth ≈ 120px` — far too large for a small icon wrapper or badge.

**Inside a row (especially with `main_axis_size: "min"`):** A container **without** an explicit `width` property will shrink-wrap to its content size. Use `padding` to control the size instead.

```json
// WRONG — 120px wide blue rectangle
{ "type": "container", "width": 1, "color": "#EFF6FF", "padding_top": 8, "padding_bottom": 8, "padding_left": 8, "padding_right": 8 }

// CORRECT — shrink-wraps to icon (36px square)
{ "type": "container", "color": "#EFF6FF", "border_radius": 8, "padding_top": 8, "padding_bottom": 8, "padding_left": 8, "padding_right": 8 }
```

**Exception:** Containers that ARE intended to take grid-column width (e.g., `width: 6` for a half-width card) should still use explicit width.

---

## RULE 22 — Avoid nested rows with grid-width children inside a parent row

When you nest `type: "row"` widgets inside a parent `type: "row"`, each inner row's children calculate their `width` against `screenWidth`. Multiple nested rows with grid-width children will almost always exceed the screen width → text overlap and horizontal overflow.

**BAD pattern (3 inner rows in 1 parent row):**
```
Parent row [space_between]:
  ├─ Inner row (identity):  icon + name     → ~200px
  ├─ Inner row (details):   width:2.5 + 1.5 + 1.5 + 2 = 7.5 grid cols → ~900px
  └─ Inner row (actions):   3 buttons        → ~150px
  TOTAL: ~1250px → OVERFLOWS on most screens
```

**GOOD pattern (2 stacked rows):**
```
Row 1 [space_between]:  [Icon + Name/subtitle]  ────────  [Action buttons]
Row 2 [left-aligned]:   Address  ·  City  ·  Frequency badge  ·  Geofence status
```

This mirrors the proven Safegroup enquiries_view pattern where each list card has stacked rows (top, middle, bottom) with `space_between`.

---

## RULE 23 — Filled dot indicators: use text `"●"` not icon `"circle"`

The Feather `circle` icon is an **outline/stroke** circle (○), not a filled dot. For status dot indicators (active/inactive), use a text widget with the Unicode filled circle character:

```json
// WRONG — renders as hollow outline ring ○
{ "type": "icon", "icon": "circle", "size": 8, "color": "#22C55E" }

// CORRECT — renders as filled dot ●
{ "type": "text", "text": "●", "font_size": 8, "color": "#22C55E",
  "color_key": "data.[index].status",
  "color_map": { "active": "#22C55E", "inactive": "#9CA3AF" },
  "element_id": "status_dot" }
```

Alternative: Use the text dot separator `"·"` (U+00B7) for inline metadata separators (font_size: 13, color: #D1D5DB).

---

## RULE 24 — `grouped_counter_1` requires 5 properties beyond `cards` array

The `cards` array alone is insufficient. Missing properties cause the widget to not render at all.

**Required properties:**
```json
{
  "type": "grouped_counter_1",
  "width": 12,
  "columns": 4,
  "element_count": 4,
  "isBackground": true,
  "customAspectRatio": 1.3,
  "element_id": "my_kpi_cards",
  "cards": [...]
}
```

| Property | Purpose |
|----------|---------|
| `width` | Grid width (always `12` for full-width) |
| `columns` | Number of cards per row |
| `element_count` | Total number of cards |
| `isBackground` | Enables card background tinting |
| `customAspectRatio` | Controls card height ratio (1.3 is standard) |

---

## RULE 25 — Popup children width is calculated against screenWidth, NOT popup width

`calculateDynamicWidth()` ALWAYS uses `widget.controller.screenWidth`. A popup with `width: 700` does NOT create a 700px-wide grid for its children. Children with `width: 4` still calculate as `4/12 × screenWidth`.

**Formula:** max grid columns that fit inside popup = `popupWidth / screenWidth × 12`

| Screen Width | Popup Width | Max Usable Grid Cols | Safe per-column width (3-col) | Safe per-column width (2-col) |
|-------------|-------------|---------------------|-------------------------------|-------------------------------|
| 1440px | 700px | ~5.8 | `2` (=240px) | `3` (=360px) |
| 1280px | 700px | ~6.6 | `2` (=213px) | `3` (=320px) |
| 1920px | 700px | ~4.4 | `1.5` (=240px) | `2` (=320px) |

**Safe approach:** For popups with `width: 700`, use `width: 2` for 3-column rows and `width: 3` for 2-column rows. Or increase popup width to accommodate larger grid columns.

---

## RULE 27 — List scrolling requires `height` on the LIST widget, NOT on the parent container

A `type: "list"` only becomes a bounded scrollable area when `height` is set **directly on the list widget**. Setting `height` on the parent `container` constrains the container's box but does NOT make the list inside scroll — items will overflow or be clipped.

```json
// WRONG — container height doesn't enable list scrolling
{
  "type": "container", "height": 280, "element_id": "panel",
  "ui": [
    { "type": "list", "data": "items.data", "element_id": "my_list", "ui": [...] }
  ]
}

// CORRECT — height on the list widget directly
{
  "type": "container", "element_id": "panel",
  "ui": [
    { "type": "list", "data": "items.data", "height": 200,
      "element_id": "my_list", "search": true, "pagination_count": 5, "ui": [...] }
  ]
}
```

**List scrolling properties:**
- `height` (number) — constrains the list to a fixed pixel height, enabling vertical scroll
- `search: true` — adds a search bar above the list
- `pagination_count: N` — limits visible items per page with pagination controls

---

## RULE 28 — Side-by-side card height matching: use `cross_axis_alignment: "start"` + manual `padding_bottom`

When two cards sit side-by-side in a `type: "row"`, do NOT use `cross_axis_alignment: "stretch"` to match heights — it forces cards to stretch to the tallest card's height, distributing empty space unpredictably inside the shorter card.

**Correct approach:** Use `"start"` so each card takes its natural height, then add `padding_bottom` to the shorter card to visually match the taller card's height.

```json
// CORRECT — manual height matching
{
  "type": "row", "cross_axis_alignment": "start", "element_id": "side_by_side",
  "ui": [
    { "type": "container", "width": 6, "padding_bottom": 105,
      "element_id": "shorter_card", "ui": [...] },
    { "type": "gutter", "element_id": "gutter_cards" },
    { "type": "container", "width": 5.5,
      "element_id": "taller_card", "ui": [...] }
  ]
}
```

**When both cards have an empty state (e.g., "No items" message):** Apply the same `padding_bottom` to the empty state container so it also matches the other card's height.

**Padding estimation:** Start with 80-120px and let the user fine-tune. Don't guess conservatively (e.g., 40-60px) — it usually needs more padding than expected.

---

## RULE 29 — Container `height` property — `fixed_height` does NOT exist

`fixed_height` is **not a valid FieldProxy container property** — it is silently ignored, causing the container to collapse to 0px height (invisible).

**Correct property:** `height`

This is especially critical for **horizontal divider lines** (1px containers used as visual separators).

```json
// ❌ WRONG — container renders as 0px (invisible)
{ "type": "container", "color": "#E5E7EB", "width": 7.0, "fixed_height": 1, "element_id": "divider" }

// ✅ CORRECT — renders as 1px gray line
{ "type": "container", "color": "#E5E7EB", "width": 7.0, "height": 1, "element_id": "divider" }
```

**Common use cases for `height` on containers:**
- Horizontal divider lines: `"height": 1`
- Timeline connector lines: `"height": 40`
- Fixed-height sections: `"height": 200`

---

## RULE 30 — `update_data` uses `key` + `query` (single data key per action) — NOT `queries` object

After an `execute_query` mutation, use `type: "update_data"` to re-fetch specific data keys so the UI reflects the change. Each `update_data` action refreshes ONE data key. For multiple keys, use separate numbered onclick steps.

**Do NOT use `refresh_data`** — it reloads the entire page, resets all dropdown/filter selections, and flashes the UI. Use `update_data` for targeted, in-place data refresh. This applies to ALL buttons including "Clear filter" buttons — use `update_data` with the default (unfiltered) queries, NOT `refresh_data`.

```json
// WRONG — queries object, refresh_data
{
  "type": "refresh_data",
  "element_id": "refresh_after_save"
}
{
  "type": "update_data",
  "queries": {
    "key1": "SELECT ...",
    "key2": "SELECT ..."
  }
}

// CORRECT — single key + query per step
"4": {
  "key": "enquiry_detail",
  "type": "update_data",
  "query": "SELECT ... WHERE id = {{arguments.enquiry_id}}",
  "element_id": "update_enquiry_detail_after_save"
},
"5": {
  "key": "audit_timeline",
  "type": "update_data",
  "query": "SELECT ... WHERE enquiry_id = {{arguments.enquiry_id}} ORDER BY created_at DESC LIMIT 20",
  "element_id": "update_audit_after_save"
}
```

**Properties:**
| Property | Required | Purpose |
|----------|----------|---------|
| `key` | Yes | Name of the data key (must match a key in the view's `data` section) |
| `type` | Yes | Always `"update_data"` |
| `query` | Yes | The SELECT query to re-fetch data for this key |
| `element_id` | Yes | Unique element ID for this action |

**Action ordering in popup onclick chains:**
`close_popup` and `toast` must come AFTER all `update_data` steps. `close_popup` can kill the action chain, preventing subsequent `update_data` from executing.

```
// CORRECT order
"1": execute_query (INSERT/UPDATE)
"2": execute_query (audit log)
"3": update_data (key 1)
"4": update_data (key 2)
"5": close_popup        ← AFTER all update_data
"6": toast              ← AFTER close_popup
```

**Which keys to refresh per action type:**
- Status/triage changes → `enquiry_detail`
- Assignment changes → `enquiry_detail` + `audit_timeline`
- Escalation → `enquiry_detail` + `audit_timeline`
- Note added → `enquiry_notes` + `notes_stats` + `audit_timeline`
- Visit/job created → `enquiry_visits` + `visit_stats`

---

## RULE 31 — `"type": "divider"` does NOT render on web — use container pattern

FieldProxy's web renderer does not support `"type": "divider"`. It renders nothing. Always use a 1px container instead.

```json
// WRONG — silently renders nothing
{ "type": "divider", "color": "#E5E7EB", "element_id": "my_divider" }

// CORRECT — visible 1px line
{ "type": "container", "color": "#E5E7EB", "width": 12, "height": 1, "element_id": "my_divider" }
```

## RULE 32 — Container `color` vs `background_color` vs `transparent`

- **`color`** — the correct property for container background color
- **`background_color`** — only valid on `sidebar`, `filled_button`, and list/card templates. Silently ignored on containers.
- **`"color": "transparent"`** — makes the container invisible (no fill). Use for wrapper containers that should not paint a background.
- **`"color": "#FFFFFF"`** — explicit white. Use for card containers that need a white fill against a grey page background (`#F1F5F9`).

**Wrapper containers** (like `bottom_section_wrapper` that only exists to group children) should always be `"color": "transparent"` so they don't paint an unwanted white rectangle.

## RULE 33 — Borders on list item containers create visual artifacts

When a list item wrapper has `border_color` + `border_width` but `color: "transparent"`, the border draws a visible rectangle against the page/card background that looks like an unwanted white box.

```json
// WRONG — creates white bordered box artifact
{ "type": "container", "color": "transparent", "border_color": "#F3F4F6", "border_width": 1, ... }

// CORRECT — subtle grey fill, no border
{ "type": "container", "color": "#F9FAFB", "border_radius": 10, ... }
```

For list items that need visual separation: use `color: "#F9FAFB"` (light grey fill) with `border_radius` — no borders needed.

## RULE 34 — Button color patterns for detail view headers

| Button Style | `background_color` | `text_color` | `icon_color` | Use Case |
|---|---|---|---|---|
| **Blue primary** | `#2563EB` | `#FFFFFF` | `#FFFFFF` | Actions dropdown (quote detail) |
| **Dark slate** | `#374151` | `#FFFFFF` | `#F0EC84` (yellow) | Quick Actions dropdown (enquiry detail) |
| **Green confirm** | `#059669` | `#FFFFFF` | — | Resolve/Accept buttons |
| **Red destructive** | `#DC2626` | `#FFFFFF` | — | Archive/NFA/Escalate/Delete buttons |

Status badge in headers should use outlined style: `"color": "#FFFFFF"` + `"border_color": "#D1D5DB"` + `"border_width": 1` + `"border_radius": 20`.

---

## PRE-FLIGHT CHECKLIST

Before submitting any view JSON, verify:

```
[ ] No width on type: "column" or type: "row"
[ ] No "border": "..." CSS strings → use border_color + border_width
[ ] All space_widget inside rows have alignment: "horizontal"
[ ] Every element has a unique element_id (no index, [index], [0], dots)
[ ] When modifying existing files: new element_ids checked against ALL existing IDs in the file
[ ] expanded: true only on type: "text"
[ ] Inner/wrapper containers have color: "transparent" where needed
[ ] No "type": "divider" — use container with height: 1 instead (Rule 31)
[ ] List item containers: use color fill (#F9FAFB), NOT border on transparent (Rule 33)
[ ] Elements that must align are siblings at the same nesting level
[ ] Inner rows that wrap content use main_axis_size: "min"
[ ] Centering = two spacers (before + after), not one
[ ] onclick only on containers where full-area hover is intentional
[ ] visibility_condition (NOT show_if) with single-quoted templates
[ ] Buttons use width (1-12), not fixed_width
[ ] All input_type values are from the valid list
[ ] onclick keys are pure numeric strings: "1", "2", "3" (never "1a")
[ ] show_key: false on object-data dropdowns
[ ] Multi-column layouts (4+ items) inside padded parents use type: "wrap" not "row"
[ ] icon_button uses bg_color + color + size (NOT background_color/icon_color/border_radius)
[ ] List item element_ids have NO _[index] suffix (framework handles row differentiation via $index)
[ ] Input data in list popups uses .[index] not .0 (popup renders at list item's index)
[ ] All icon names are valid Feather icons (no "building", "warehouse", "factory")
[ ] Small containers in rows (icon wrappers, badges) have NO width — use padding to size
[ ] No nested rows-in-rows where inner children have grid widths (use stacked rows instead)
[ ] Status dot indicators use text "●" not icon "circle" (which renders as outline)
[ ] grouped_counter_1 has width + columns + element_count + isBackground + customAspectRatio
[ ] Popup child widths account for screenWidth calculation (use width:2 for 3-col in 700px popup)
[ ] visibility_condition NEVER placed directly on type: "input" — wrap input in container with visibility_condition instead
[ ] List scrolling uses `height` on the list widget directly — NOT on parent container
[ ] Side-by-side card height matching uses cross_axis_alignment: "start" + padding_bottom (NOT "stretch")
[ ] Container height uses `height` (NOT `fixed_height` — silently ignored)
[ ] Data refresh after mutations uses `update_data` with `key` + `query` (NOT `refresh_data` or `queries` object)
[ ] "Clear filter" buttons use `update_data` with default queries (NOT `refresh_data` which resets all page state)
[ ] Child input widths inside nested containers account for parent grid width (Rule 35)
[ ] Dropdown `init` on dropdowns with `value_field` uses ID, not display name (Rule 36)
[ ] Edit forms with many fields use separate view (navigation), NOT popup — popups can't scroll (Rule 37)
[ ] `default_value` is NOT a valid property — use `init` for prefilling inputs (Rule 38)
[ ] Status badges in list items use separate containers with visibility_condition + static color per status — NEVER dynamic color templates (Rule 39)
```

---

## RULE 35 — Child widths inside nested containers overflow [LEARNED: 2026-03-20]

`calculateDynamicWidth()` ALWAYS uses `screenWidth`, even for children inside a nested container. A `width: 6` input inside a `width: 8.5` container calculates as 50% of screen — but the container is only 70% of screen — causing overflow.

**Formula for safe child width:** `desired_fraction × parent_grid_width`

| Parent Container Width | 2-column child width | Full-width child width |
|------------------------|---------------------|----------------------|
| 12 (full screen) | 6 | 12 |
| 8.5 | 4 | 8 |
| 8 | 4 | 7.5 |
| 6 | 3 | 5.5 |

**Rule:** Never use `width: 6` for 2-column layouts inside containers smaller than `width: 12`. Use `width: 4` for containers around `width: 8-9`.

## RULE 36 — Dropdown init with display name breaks onchange [LEARNED: 2026-03-20]

When a dropdown has `value_field: "id"` and `init` is set to a display name (e.g. "Steve Electricals"), FieldProxy stores the name as the selected value. Onchange queries like `WHERE id = 'Steve Electricals'` fail with type mismatch, causing "Processing" spinner.

**Safe patterns:**
- **Don't prefill dropdowns on edit screens** — show current values in a separate read-only panel
- If you must prefill, use the ID: `"init": "{{data.[0].id}}"` (shows the raw ID number, not pretty but works)
- For simple string arrays (no `value_field`), `init` with the string value works fine

## RULE 37 — Popups cannot scroll, use separate views for long forms [LEARNED: 2026-03-20]

From source code: regular popups use `Column(mainAxisSize: min)` — content overflows with no scroll. `fullscreen: true` wraps in `SingleChildScrollView` but still unreliable for complex forms with dropdowns.

**Fix:** Move long edit forms to a dedicated view with `navigation` instead of a popup. Views scroll naturally. Pass the record ID via `arguments`.

## RULE 38 — `default_value` is not a valid property [LEARNED: 2026-03-20]

`default_value` is silently ignored on all input types. Use `init` for prefilling text inputs, number inputs, and dropdowns.

## RULE 39 — Status badges in lists: use separate containers with `visibility_condition`, NOT dynamic `color` templates [LEARNED: 2026-03-20]

**Dynamic template values in container `color` do NOT render in list items.** The template `{{data.[index].status_color}}` resolves to a hex string (confirmed via data inspection), but the container renderer does not repaint with the resolved color — badges appear invisible/white.

**ONLY working pattern — visibility_condition per status:**

```json
// WRONG — dynamic color template, renders invisible
{
  "type": "container",
  "color": "{{quotes.data.[index].status_color}}",
  "border_radius": 12,
  "element_id": "status_badge",
  "ui": [{ "type": "text", "text": "{{quotes.data.[index].status_label}}" }]
}

// CORRECT — one container per status with hardcoded color + visibility_condition
{
  "type": "container",
  "color": "#FEF3C7",
  "border_radius": 12,
  "visibility_condition": "'{{quotes.data.[index].status_code}}' == 'DRAFT'",
  "element_id": "badge_draft",
  "ui": [{ "type": "text", "text": "Draft", "color": "#92400E", "font_size": 11 }]
},
{
  "type": "container",
  "color": "#DBEAFE",
  "border_radius": 12,
  "visibility_condition": "'{{quotes.data.[index].status_code}}' == 'PENDING_APPROVAL'",
  "element_id": "badge_pending",
  "ui": [{ "type": "text", "text": "Pending Approval", "color": "#1E40AF", "font_size": 11 }]
}
// ... repeat for each status value
```

**Key rules:**
- Each status gets its own container with a **static hardcoded `color`** (not a template)
- `visibility_condition` controls which container is shown based on the data value
- Template values in `visibility_condition` MUST be in single quotes (Rule 12)
- This is the SAME proven pattern used for visit status badges in the Visits & Jobs tab
- Works because `visibility_condition` is evaluated as a string comparison, while `color` template resolution fails in list item rendering context

---

## RULE 40 — List without `height` stretches last item to fill viewport

When a `list` widget is a direct child of the root `ui` array without a `height` property, FieldProxy's flex layout stretches the last list item to fill remaining viewport space — creating large empty areas inside cards.

```json
// WRONG — last card stretches to fill screen
{ "type": "list", "data": "items.data", "element_id": "my_list", "ui": [...] }

// CORRECT — list gets scrollable area, cards stay content-sized
{ "type": "list", "data": "items.data", "element_id": "my_list", "height": 700, "ui": [...] }
```

**Fix**: Set `height` on the list widget (Rule 27 pattern). This creates a scrollable area where cards render at their natural content height. Use `700` as a good default for full-page lists.

---

## RULE 41 — `init` in popup inputs resolves `[index]` at render time, not click time

When a popup is opened from a list item's onclick, `init` on inputs inside the popup evaluates `[index]` at **list render time** (always `0`), not at click time. Every row's edit popup pre-fills with the first row's data.

**onclick actions** (`execute_query`, `update_data`) resolve `[index]` correctly at click time — this is why delete works but init doesn't.

```json
// WRONG — init always shows first row's data
"init": "{{items.data.[index].name}}"

// CORRECT — fetch the row first, then reference [0]
"onclick": {
  "1": {
    "key": "edit_data",
    "type": "update_data",
    "query": "SELECT * FROM table WHERE id = {{items.data.[index].id}}",
    "element_id": "fetch_row"
  },
  "2": {
    "type": "popup",
    "ui": [{
      "type": "input",
      "init": "{{edit_data.data.[0].name}}",
      ...
    }]
  }
}
```

**Pattern**: Use `update_data` (step 1) to fetch the specific row by ID into a separate data key. The `[index]` in the query resolves correctly at click time. Then the popup's `init` uses `[0]` on the fetched single-row result.

**Status**: Ticket filed with product team (FIE-402). Workaround above not yet verified.

---

## RULE 42 — MCQ multi-select: use `is_multi: true` + plain string arrays

MCQ inputs default to single-select (radio buttons). For multi-select (checkboxes), add `is_multi: true`.

Static MCQ data must be **plain string arrays**, not `{label, value}` objects:

```json
// WRONG — shows raw objects "{label: Monday, value: Monday}"
"data": [{"label": "Monday", "value": "Monday"}, ...]

// CORRECT — shows clean text "Monday"
"data": ["Monday", "Tuesday", "Wednesday", ...]
```

For query-driven MCQ, return a single column (the display value):
```json
"data": "facilities_list.data"
// where query is: SELECT name FROM facilities ORDER BY name
```

**Web MCQ `init` limitation**: `init` with `ARRAY_AGG` only pre-selects the **first** value on web. Multi-value pre-fill does NOT work. `is_checklist: true` is mobile-only and breaks web MCQ completely. Workaround: show "Currently assigned" text above the MCQ for context.

**SQL operators in JSON strings**: `!=` and `<>` get split with spaces by FieldProxy's JSON parser, causing SQL errors. Use `LENGTH(TRIM(val)) > 0` instead of `TRIM(val) <> ''`.

To store multi-select values, use `.0.formatted_value` in queries:
```json
"query": "INSERT INTO table (days) VALUES ('{{input_data.weekly_days.0.formatted_value}}')"
```

**Note**: Values stored by MCQ may include `{` `}` braces. Strip them in display queries:
```sql
REPLACE(REPLACE(COALESCE(weekly_days, ''), '{', ''), '}', '') AS weekly_days
```
