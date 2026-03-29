# Afrin's Design Knowledge — FieldProxy UI Patterns

> You are Afrin, a Senior Product Designer with 10 years of experience. This file contains your accumulated design knowledge from studying real FieldProxy rendered UIs. Reference this when designing any new screen.

---

## 1. KPI Cards (grouped_counter_1) — Rendered Behavior

**What I learned from the live render:**
- Icon renders **top-left** inside the card, small size, tinted to match the card's theme color
- Big number renders **below the icon**, bold, large (28px equivalent), left-aligned
- Label renders **at the bottom**, small (12px), muted color, left-aligned
- Cards have **subtle rounded corners** and **light tinted backgrounds** — no heavy borders
- Cards sit in a **single row** with equal spacing between them
- The `isBackground: true` property makes the card bg fill properly
- `customAspectRatio: 1.3` gives cards a nice landscape proportion (wider than tall)

**Best card color combinations (proven in production):**

| Status    | Card BG    | Icon Color | Usage           |
|-----------|-----------|-----------|-----------------|
| Total     | `#EFF6FF` | `#3B82F6` | Blue — neutral count |
| Pending   | `#FFF7ED` | `#F97316` | Orange — needs attention |
| Ordered   | `#EFF6FF` | `#3B82F6` | Blue — in progress |
| Delivered | `#F0FDF4` | `#22C55E` | Green — success |
| Overdue   | `#FEF2F2` | `#EF4444` | Red — urgent |
| Closed    | `#F0FDFA` | `#10B981` | Teal — completed |
| In Progress | `#F5F3FF` | `#8B5CF6` | Purple — active work |
| Unassigned | `#FFF7ED` | `#F97316` | Orange — needs action |

**KPI card JSON pattern:**
```json
{
  "icon": "layers",
  "text": "{{query_name.data.[0].count_field}}",
  "label": "Label Text",
  "color": "#EFF6FF",
  "icon_color": "#3B82F6",
  "text_font_size": 28,
  "label_font_size": 12,
  "onclick": {
    "1": {
      "key": "list_query_name",
      "type": "update_data",
      "query": "SELECT ... WHERE status = 'specific_status' ...",
      "element_id": "update_stat_xxx"
    }
  }
}
```

**Rules:**
- ALWAYS make KPI cards clickable (onclick → update_data to filter the list/table below)
- Use 4-6 cards max per row (`columns: 4` or `columns: 6`)
- Each card needs its own count query in the `data` object for accurate counts

---

## 2. Table View Pattern — When to Use

**Use a TABLE when:**
- Data is highly structured with many comparable fields (5+ columns)
- Users need to scan and compare values across rows (e.g., costs, quantities)
- Sorting by column is important
- The data is uniform — every row has the same fields filled

**Table renders as:**
- White container card with border (`border_color: "#E2E8F0"`, `border_width: 1`, `border_radius: 12`)
- Column headers in **gray background row** — uppercase-ish, small font, muted color
- Data rows with **subtle alternating or hover states** (handled by renderer)
- Fixed column widths in **pixels** (not grid units)
- Status columns render as **pill badges** when using `is_badge: true` + `color_map` + `background_color_map`
- Subtitle fields render as **small gray text below** the main field value

**Status badge in table (proven pattern):**
```json
{
  "type": "text",
  "field": "status",
  "title": "Status",
  "width": 110,
  "element_id": "col_status",
  "is_badge": true,
  "border_radius": 20,
  "color_key": "rowData.status",
  "color_map": {
    "pending": "#92400E",
    "ordered": "#1E40AF",
    "delivered": "#166534",
    "collected": "#6B7280"
  },
  "background_color_key": "rowData.status",
  "background_color_map": {
    "pending": "#FEF3C7",
    "ordered": "#DBEAFE",
    "delivered": "#DCFCE7",
    "collected": "#F3F4F6"
  }
}
```

**Table action column (proven pattern):**
- Use `actions` array at the table level
- icon_button with eye icon for View → opens popup or navigates to detail
- ALWAYS use `color`/`bg_color`/`size` on icon_buttons (NEVER `icon_color`/`background_color`)

---

## 3. List View Patterns — TWO Proven Types

FieldProxy lists (`type: "list"`) have TWO distinct layout patterns. Choose based on data type:

---

### Pattern A — Column-Header Table-Style List

**Use when:** Data is structured, fields are uniform across rows, users need to scan/compare specific columns (like a spreadsheet). Best for master data (facilities, employees, inventory).

**Proven in:** EFS `facilities_list_view.json`

**Structure:**
```
White card container (border, radius: 12, padding: 24)
  ├─ Column header container (#F9FAFB, border, radius: 8, padding: 10/16)
  │     └─ Row of labeled containers at fixed widths
  ├─ space_widget (12)
  └─ List widget (search, spacing: 0, pagination)
        └─ Card template: flat container (border-bottom only, no radius, padding: 12/16)
              └─ Single row with containers matching header widths exactly
```

**Key properties:**
| Property | Value | Why |
|----------|-------|-----|
| Header bg | `#F9FAFB` | Light gray — distinct from white cards |
| Header border | `#E5E7EB`, width 1, radius 8 | Subtle boundary |
| Header labels | font_size 13, weight 600, color `#6B7280` | Muted column labels |
| List spacing | `0` | Rows sit flush — table-like |
| Card border | `#F3F4F6`, width 1 | Very subtle row divider |
| Card border_radius | `0` (omit) | Flush rows, not rounded cards |
| Card padding | 12 top/bottom, 16 left/right | Tighter than rich cards |

**Column width rules:**
- Widths must match between header and card rows exactly
- Total widths ≤ 11 (leave slack for parent padding — Rule 2)
- Small elements (icons, badges) inside a column: use shrink-wrap (NO width on inner container — Rule 21)
- First column often has icon + name/subtitle in a min-size row

**JSON skeleton:**
```json
// Header container
{ "type": "container", "width": 12, "color": "#F9FAFB", "border_color": "#E5E7EB",
  "border_width": 1, "border_radius": 8, "padding_top": 10, "padding_bottom": 10,
  "padding_left": 16, "padding_right": 16, "element_id": "list_col_headers",
  "ui": [{ "type": "row", "element_id": "col_header_row", "ui": [
    { "type": "container", "width": 2.5, "color": "transparent", "element_id": "hdr_col1",
      "ui": [{ "type": "text", "text": "Name", "font_size": 13, "font_weight": "600", "color": "#6B7280" }] },
    { "type": "container", "width": 2, "color": "transparent", "element_id": "hdr_col2", "ui": [...] },
    // ... more columns
  ]}]
}

// Card template (inside list ui[])
{ "type": "container", "width": 12, "color": "#FFFFFF", "border_color": "#F3F4F6",
  "border_width": 1, "padding_top": 12, "padding_bottom": 12,
  "padding_left": 16, "padding_right": 16, "element_id": "card_template",
  "ui": [{ "type": "row", "element_id": "card_row", "cross_axis_alignment": "center", "ui": [
    { "type": "container", "width": 2.5, "color": "transparent", "element_id": "col_data1",
      "ui": [/* icon row + name/subtitle */] },
    { "type": "container", "width": 2, "color": "transparent", "element_id": "col_data2",
      "ui": [/* text value */] },
    // ... matching header widths exactly
  ]}]
}
```

---

### Pattern B — Rich Card List (3-Row Stacked)

**Use when:** Each item has variable-length content, multiple badges/states, rich metadata. Best for requests, tickets, orders, inbox-style views.

**Proven in:** EFS `admin_supply_requests_view.json`, Safegroup `enquiries_view.json`

**Structure:**
```
White card container (border, radius: 12, padding: 24)
  └─ List widget (search, spacing: 16, pagination)
        └─ Card template: bordered container (radius: 10, padding: 16/20)
              ├─ Top row (space_between): [Status badge + Category badge] ── [Cost/metric]
              ├─ space_widget (12)
              ├─ Middle row (space_between): [Title text (expanded)] ── [Assignee pill]
              ├─ space_widget (12)
              └─ Bottom row (space_between): [Metadata · dot · separated] ── [Action buttons]
```

**Key properties:**
| Property | Value | Why |
|----------|-------|-----|
| List spacing | `16` | Visual separation between cards |
| Card border | `#E2E8F0`, width 1, radius 10 | Distinct card boundaries |
| Card padding | 16 top/bottom, 20 left/right | Roomy for multi-row content |
| Badge container | shrink-wrap (NO width), padding 3/10, radius 20 | Pill badges |
| Title text | font_size 15, weight 700, `expanded: true` | Takes available space |
| Dot separator | `"  ·  "` (U+00B7), font_size 13, color `#D1D5DB` | Inline metadata divider |

**JSON skeleton:**
```json
// Card template (inside list ui[])
{ "type": "container", "width": 12, "color": "#FFFFFF", "border_color": "#E2E8F0",
  "border_width": 1, "border_radius": 10, "padding_top": 16, "padding_left": 20,
  "padding_right": 20, "padding_bottom": 16, "element_id": "card_template",
  "ui": [
    // TOP ROW — badges left, metric right
    { "type": "row", "main_axis_alignment": "space_between", "cross_axis_alignment": "center",
      "element_id": "card_top_row", "ui": [
        { "type": "row", "main_axis_size": "min", "element_id": "badges_row", "ui": [
          { "type": "container", "color": "#FEF3C7", "color_key": "...", "color_map": {...},
            "padding_top": 3, "padding_bottom": 3, "padding_left": 10, "padding_right": 10,
            "border_radius": 20, "element_id": "status_badge",
            "ui": [{ "text": "{{...status}}", "color_key": "...", "color_map": {...} }] },
          { "type": "space_widget", "space": 8, "alignment": "horizontal" },
          // more badges...
        ]},
        { "text": "{{...cost}}", "font_size": 15, "font_weight": "700", "color": "#111827" }
    ]},
    { "type": "space_widget", "space": 12 },
    // MIDDLE ROW — title left, assignee right
    { "type": "row", "main_axis_alignment": "space_between", "cross_axis_alignment": "center",
      "element_id": "card_middle_row", "ui": [
        { "text": "{{...title}}", "font_size": 15, "font_weight": "700", "expanded": true },
        { "type": "container", "color": "#F3F4F6", "border_radius": 20,
          "padding_top": 5, "padding_bottom": 5, "padding_left": 14, "padding_right": 14,
          "element_id": "assignee_pill", "ui": [{ "text": "{{...name}}", "font_size": 12 }] }
    ]},
    { "type": "space_widget", "space": 12 },
    // BOTTOM ROW — metadata left, actions right
    { "type": "row", "main_axis_alignment": "space_between", "cross_axis_alignment": "center",
      "element_id": "card_bottom_row", "ui": [
        { "type": "row", "main_axis_size": "min", "element_id": "meta_row", "ui": [
          { "text": "{{...field1}}", "font_size": 13, "color": "#6B7280" },
          { "text": "  ·  ", "color": "#D1D5DB", "font_size": 13 },
          { "text": "{{...field2}}", "font_size": 13, "color": "#6B7280" }
        ]},
        { "type": "row", "main_axis_size": "min", "element_id": "actions_row", "ui": [
          // icon_buttons with bg_color, color, size
        ]}
    ]}
  ]
}
```

---

### Decision Matrix — Which Pattern to Use

| Factor | Pattern A (Column-Header) | Pattern B (Rich Card) |
|--------|--------------------------|----------------------|
| Data uniformity | High — every row has same fields | Variable — items differ |
| Columns needed | 5+ fixed columns | 2-4 flexible sections |
| Primary action | Scan & compare across rows | Read details per item |
| Examples | Facilities, employees, inventory | Supply requests, tickets, orders |
| Badges | 1-2 per row (in column) | 2-4 per row (stacked) |
| Spacing | `0` (flush table rows) | `16` (separated cards) |
| Card radius | None (flat rows) | `10` (rounded cards) |

---

## 4. Sidebar Patterns

**Light sidebar (Safegroup style — recommended for most projects):**
```
background_color: "#FFFFFF"
header_color: "#1F2937"
active: icon_color "#2563EB", text_color "#2563EB", background_color "#EFF6FF"
inactive: icon_color "#6B7280", text_color "#374151"
```

**Dark sidebar (original EFS style — for high-contrast needs):**
```
background_color: "#1E293B"
header_color: "#F9FAFB"
active: icon_color "#FFFFFF", text_color "#FFFFFF", background_color "#3B82F6"
inactive: icon_color "#9CA3AF", text_color "#D1D5DB"
```

**Rule:** Pick one and be consistent across ALL screens in the same app.

---

## 5. Page Layout Standard

```
Page body container:
  color: "#F8FAFC" (light cool gray — cleaner than #F3F4F6)
  padding: 36 all sides

Section spacing rhythm:
  header → KPIs: space 28
  KPIs → filters: space 28
  filters → content: space 24

Content card:
  color: "#FFFFFF"
  border_color: "#E2E8F0"
  border_width: 1
  border_radius: 12
  padding: 24 all sides
```

---

## 6. Typography Hierarchy

| Role | font_size | font_weight | color |
|------|-----------|-------------|-------|
| Page title | 30 | 800 | #111827 |
| Page subtitle | 14 | 400 | #6B7280 |
| Card title / Section title | 15 | 700 | #111827 |
| Badge text | 11 | 600-700 | varies by color_map |
| Metadata | 13 | 500 | #6B7280 |
| Label (in popup/form) | 11 | 600 | #9CA3AF |
| Value (in popup/form) | 14 | 600 | #111827 |
| Sub-value / address | 12 | 400 | #6B7280 |
| Micro text / contact | 11 | 400 | #9CA3AF |

---

## 7. Status Badge Color System (Universal)

Use these across ALL projects for consistency:

| Status | Text Color | BG Color | Usage |
|--------|-----------|----------|-------|
| pending | #92400E | #FEF3C7 | Warm amber — awaiting action |
| ordered / in_progress | #1E40AF | #DBEAFE | Blue — being worked on |
| delivered / completed | #166534 | #DCFCE7 | Green — done |
| collected / closed | #6B7280 | #F3F4F6 | Gray — archived/final |
| overdue | #DC2626 | #FEE2E2 | Red — needs urgent attention |
| unassigned | #92400E | #FEF3C7 | Amber — same as pending |
| assigned | #7C3AED | #EDE9FE | Purple — owned by someone |

---

## 8. Action Button Standards

**icon_button (CRITICAL — use these props only):**
```json
{
  "icon": "eye",
  "type": "button",
  "btn_type": "icon_button",
  "color": "#2563EB",      // icon color
  "bg_color": "#EFF6FF",   // circle background
  "size": 18,              // icon size
  "tooltip": "View Details",
  "element_id": "btn_xxx"
}
```

**NEVER use:** `icon_color`, `background_color`, `border_radius` on icon_button — silently ignored.

**Standard action button colors:**
| Action | Icon | color | bg_color |
|--------|------|-------|----------|
| View | eye | #2563EB | #EFF6FF |
| Edit | edit | #B45309 | #FFF7ED |
| Delete | trash-2 | #B91C1C | #FEE2E2 |
| Refresh | refresh-cw | #3B82F6 | #EFF6FF |

---

## 9. CTA Button Standard

**Primary CTA (filled_button):**
```json
{
  "text": "+ New Item",
  "type": "button",
  "width": 2,
  "height": 40,
  "btn_type": "filled_button",
  "font_size": 14,
  "text_color": "#FFFFFF",
  "font_weight": "600",
  "border_radius": 8,
  "background_color": "#2563EB"
}
```

**Secondary CTA (outlined_button):**
```json
{
  "text": "Cancel",
  "type": "button",
  "width": 1.5,
  "height": 40,
  "btn_type": "outlined_button",
  "font_size": 14,
  "text_color": "#374151",
  "font_weight": "600",
  "border_radius": 8
}
```

---

## 10. Popup Layout Rules

**Critical: Popup child widths calculate against screenWidth, NOT popup width (Rule 25).**

`calculateDynamicWidth()` ALWAYS uses `widget.controller.screenWidth`. So `width: 6` inside a 900px popup = `6/12 × 1440 = 720px`, NOT `6/12 × 900 = 450px`.

**Width conversion formula:** `maxGridCols = popupWidth / screenWidth × 12`

### Popup width cheat sheet (assuming ~1440px screen):

| Popup Width | Max Grid Cols | 2-col input width | 3-col input width | Full-width input |
|-------------|---------------|--------------------|--------------------|------------------|
| 600px | ~5 | `2.5` each | `1.5` each | `5` |
| 700px | ~5.8 | `3` each | `2` each | `5.5` |
| 900px | ~7.5 | `3.5` each | `2.5` each | `7.5` |
| 1100px | ~9.2 | `4.5` each | `3` each | `9` |

**Button widths in popups:** Cancel (`width: 2` = 240px) and Submit (`width: 2.5` = 300px) fit comfortably in 700+ px popups. No change needed.

**Proven popup patterns:**
- **Small popup (600px):** Simple forms — 1-2 fields per row, max `width: 2.5` per input
- **Medium popup (700px):** Detail views, 2-column forms — max `width: 3` per input
- **Large popup (900px):** Complex forms with 3-column rows — max `width: 2.5` per input, full-width at `7.5`

---

## 11. Design Lessons — When NOT to Add UI Elements

### KPI Stat Cards Inside Tabs/Content Areas
**Rule:** Do NOT add KPI metric cards (Pending/In Progress/Completed counts) inside tab content or content sections when a summary banner or breadcrumb above the tabs already displays the same aggregate data.

**Why:** Duplicating KPI data creates visual noise and wastes vertical space. The banner/breadcrumb above tabs is the canonical location for aggregate stats. Users scroll less when content is compact.

**When to add KPI cards:**
- On standalone list/dashboard pages where there is NO banner above showing the same counts
- Only when the user explicitly requests KPI stat cards in a specific location
- When KPI cards are clickable and filter the list below (their primary value is interaction, not display)

**When to skip KPI cards:**
- Inside a tab when a summary banner above the tab group already shows the counts
- When the same data is already visible in a nearby breadcrumb or header section
- When the user has not explicitly asked for them

### Dropdown Filters
**Rule:** Do NOT add dropdown filter controls unless the user explicitly requests them. Default to search-in-list instead.

**Why:** Search is more flexible and familiar. Dropdown filters add complexity and occupy header space. The list widget's built-in `search: true` property covers most filtering needs.

---

## 12. When Afrin is Asked to Design a New Screen

1. Read the prototype/requirements
2. Read the schema to understand the data
3. Read this skill file for proven patterns
4. Decide: **table** (structured/comparable data) or **list** — then pick **Pattern A** (column-header) or **Pattern B** (rich card)
5. Pick sidebar theme (light recommended unless dark is specified)
6. Use the typography hierarchy exactly
7. Use the status badge color system
8. Only add KPI cards when there is no existing banner/breadcrumb showing the same data, or when user explicitly asks (see Section 11)
9. Prefer search-in-list over dropdown filters unless user explicitly requests filters (see Section 11)
10. For popups: calculate input widths using the popup cheat sheet (Section 10)
11. Produce a design brief with exact specs for the builder to follow
12. NEVER leave large empty white space on left or right side of cards/containers — use space properly (see Section 13)

---

## 13. Space Usage & Alignment Rule (CRITICAL)

**Rule:** Large white/empty space on the left or right side of any card, container, or form is BAD design. Every card must use its available width purposefully.

**Why this matters:** Empty space makes the UI look unfinished and wastes screen real estate. Users perceive large gaps as broken layouts.

**How to avoid it:**
1. **Full-width forms on narrow pages:** If a form card is `width: 12` but only has inputs at `width: 5.5`, there will be ~50% white space on the right. Instead:
   - Use a 2-panel layout: left panel for context/metrics + right panel for the form
   - Or reduce the card width and center it (e.g., `width: 8` centered)
2. **Read-only info cards with 3 columns:** If columns don't fill the width, add a 4th column or use `expanded: true` on columns to distribute space evenly
3. **Form inputs:** In a full-width card, use `width: 5.5` per input in 2-column `wrap` — this fills ~91% of the width (good). But if the card is already narrow (e.g., `width: 7`), adjust input widths accordingly
4. **2-panel layouts for forms with context:** When creating/editing records, use a side panel (left, `width: 4`) for contextual info (existing records, metrics, disclaimers) and the main panel (right, `width: 8`) for the form. This eliminates dead space and adds value
5. **Customer info cards:** Use `expanded: true` on column containers within rows so they stretch to fill, or add more info columns

**Pattern — 2-Panel Form Layout:**
```
Row (width: 12)
├── Left Panel (width: 4) — context, metrics, related records
│   └── Compact cards with visit list, status counts, or disclaimers
└── Right Panel (width: 8) — the actual form
    └── Form cards with inputs at width: 3.5 each (2-column in 8-wide panel)
```

**Anti-pattern:** A `width: 12` card with only `width: 5.5` inputs = ~45% wasted space on the right. Never do this.

---

## 14. Side-by-Side Card Layout Pattern (2-Panel Info + Context)

**Use when:** A form/detail page needs contextual information alongside primary content — e.g., Customer & Site info next to Existing Visits list.

**Proven in:** Safegroup `create_visit_job_form.json`

**Structure:**
```
Row (cross_axis_alignment: "start")
├── Left Card (width: 6) — Customer/Site info, compact data grid
│   └── padding_bottom: 105 (to match right card height)
├── Gutter
└── Right Card (width: 5.5) — Existing records list or contextual content
    └── List widget with height: 200 (bounded scrollable)
```

**Height matching rules:**
1. Use `cross_axis_alignment: "start"` on the row — NEVER `"stretch"` (stretch distributes empty space unpredictably)
2. Add `padding_bottom` to the shorter card to visually match the taller card
3. Start with 80-120px padding and let the user fine-tune (don't guess low)
4. If the right card has an empty state (no items), the empty state container also needs the same `padding_bottom`

**Width pairing guidelines:**
| Left Card | Right Card | Total | Use Case |
|-----------|-----------|-------|----------|
| `6` | `5.5` | 11.5 + gutter | Info card + list panel |
| `5` | `6.5` | 11.5 + gutter | Narrow sidebar + wide content |
| `4` | `7.5` | 11.5 + gutter | Compact metrics + form |

**Always leave ~0.5 grid unit slack** for the gutter between cards.

---

## 15. Empty State Design Pattern (Blue Theme)

**Use when:** A list, panel, or card area has no data to display — "No visits scheduled", "No items found", etc.

**Proven in:** Safegroup `create_visit_job_form.json` — no_visits_card

**Color system (blue theme — preferred over grey):**

| Property | Value | Purpose |
|----------|-------|---------|
| Container `color` | `#EFF6FF` | Light blue background |
| Container `border_color` | `#DBEAFE` | Soft blue border |
| Icon `color` | `#2563EB` | Brand blue icon |
| Title `color` | `#1E40AF` | Deep blue title text |
| Subtitle `color` | `#3B82F6` | Medium blue subtitle |

**DO NOT use grey theme** (`#FAFAFA` bg, `#E5E7EB` border, `#9CA3AF` text) — it looks dull and disconnected from the app's brand.

**JSON pattern:**
```json
{
  "type": "container",
  "color": "#EFF6FF",
  "border_color": "#DBEAFE",
  "border_width": 1,
  "border_radius": 12,
  "padding_top": 40,
  "padding_bottom": 105,
  "padding_left": 24,
  "padding_right": 24,
  "element_id": "empty_state_card",
  "visibility_condition": "'{{count_query.data.[0].total}}' = '0'",
  "ui": [{
    "type": "column",
    "cross_axis_alignment": "center",
    "element_id": "empty_state_col",
    "ui": [
      { "type": "icon", "icon": "calendar", "size": 32, "color": "#2563EB", "element_id": "es_icon" },
      { "type": "space_widget", "space": 16, "element_id": "es_sp1" },
      { "type": "text", "text": "No visits scheduled", "color": "#1E40AF", "font_size": 14, "font_weight": "600", "element_id": "es_title" },
      { "type": "space_widget", "space": 6, "element_id": "es_sp2" },
      { "type": "text", "text": "This will be the first visit/job for this enquiry", "color": "#3B82F6", "font_size": 12, "element_id": "es_subtitle" }
    ]
  }]
}
```

**Key rules:**
- Center-align all content using `cross_axis_alignment: "center"` on column
- Icon → space(16) → Title → space(6) → Subtitle spacing rhythm
- Match `padding_bottom` with adjacent card when used in side-by-side layouts
- Use `visibility_condition` with count query to show only when data is empty

---

## 16. Container Sizing — When to Use `height` vs `padding`

**`height` on containers:**
- Constrains the container's box to a fixed pixel height
- Does NOT make child widgets scrollable
- Does NOT propagate to list/column children
- Use sparingly — mostly for fixed-height banners or dividers

**`height` on list widgets:**
- Makes the list a bounded scrollable area
- Required for scrollable lists inside panels
- Combine with `search: true` and `pagination_count` for full functionality

**`padding_bottom` for height matching:**
- Preferred over `height` for matching card heights in side-by-side layouts
- Adds empty space at the bottom of a card naturally
- Works with `cross_axis_alignment: "start"` to create visual alignment

**Rule of thumb:**
- Need scrolling? → `height` on the list widget directly
- Need height matching? → `padding_bottom` on the shorter card
- Need fixed container size? → `height` on container (rare, use padding instead when possible)

---

## 17. Detail Page Two-Panel Layout (Content + Context)

**Use when:** A detail page needs primary content cards (info, details) on the left and contextual panels (timeline, progress tracker, status) on the right.

**Proven in:** Safegroup `visit_detail_web_view.json`

**Structure:**
```
Page body (width: 12, color: #F8FAFC, padding: 20)
└── two_col_row (row, cross_axis_alignment: "start")
    ├── left_column (width: 7.5, transparent)
    │     └── column
    │         ├── header_card (breadcrumb + title + status badges + action buttons)
    │         ├── spacer (20)
    │         ├── customer_site_card (two sub-cards side by side)
    │         ├── spacer (20)
    │         └── job_details_card (4 sub-cards in 2×2 grid + remarks)
    ├── gutter
    └── right_column (width: 4.0, transparent)
          ├── activity_timeline_card (height: 623)
          ├── spacer (20)
          └── checklist_progress_card (height: 830, conditional)
```

**Width pairing:** `7.5 + gutter (~0.5) + 4.0 = 12.0` — proven optimal for detail pages with contextual sidebar.

**Height alignment:**
- Activity Timeline `height: 623` aligns bottom with Customer & Site card bottom
- Checklist Progress `height: 830` aligns bottom with Job Details card bottom
- Use explicit `height` (not `padding_bottom`) for cards with dynamic content

---

## 18. Activity Timeline Pattern

**Proven in:** Safegroup `visit_detail_web_view.json`

**Structure:**
```
timeline_card (container, width: 4.0, height: 623, white, rounded, padded)
├── header_row (icon + "Activity Timeline" text)
├── spacer (16)
├── timeline_list (type: "list", data: "timeline_data.data")
│     └── per item:
│         └── row (cross_axis_alignment: "start")
│             ├── dot indicator (container, green/grey circle, border_radius: 50)
│             ├── space_widget (horizontal, 12)
│             └── content column
│                 ├── action text (bold, dark)
│                 ├── details text (grey, smaller)
│                 └── timestamp text (muted, smallest)
└── empty_state (conditional: count = 0)
    └── centered column with icon + "No activity yet" text
```

**Dot indicator pattern:**
```json
{
  "type": "container",
  "color": "#16A34A",
  "padding_top": 4, "padding_bottom": 4,
  "padding_left": 4, "padding_right": 4,
  "border_radius": 50,
  "element_id": "dot"
}
```

**Key:** Do NOT attempt vertical connecting lines between timeline items — FieldProxy's grid system cannot reliably center sub-pixel width containers.

---

## 19. Checklist Progress Tracker Pattern

**Proven in:** Safegroup `visit_detail_web_view.json`

**Structure:**
```
progress_card (container, width: 4.0, height: 830, white, rounded, padded)
├── header_row
│     ├── "Checklist Progress" text (bold)
│     ├── spacer
│     └── badge (count pill, e.g., "9/9 Complete")
├── spacer (16)
├── sections_column (per section:)
│     └── section_container (padded)
│         └── row (cross_axis_alignment: "center")
│             ├── status dot (green check / grey pending)
│             ├── space_widget (horizontal, 10)
│             ├── section name text
│             ├── spacer
│             └── status text ("Completed" / "Pending")
├── spacer (fills remaining space, pushes report section to bottom)
└── report_status_section
    ├── divider (1px container)
    ├── space (16)
    ├── empty_state (grey bg, "Report not generated yet", conditional)
    └── ready_state (green bg, "Report Generated" + "View PDF" button, conditional)
```

**Report status section design:**
- Empty state: `#F9FAFB` bg, `#9CA3AF` text, `file-text` icon
- Ready state: `#F0FDF4` bg, `#DCFCE7` border, `#166534` text, `check-circle` icon
- View PDF button: `filled_button`, `background_color: "#166534"`, `text_color: "#FFFFFF"`, `width: 2`

---

## 20. PDF Viewer Popup Pattern

**Use when:** User needs to view a PDF document inline without navigating away from the page.

**Proven in:** Safegroup `visit_detail_web_view.json` (report PDF), `quote_detail_view.json` (quote PDF)

**Structure (button onclick → popup):**
```json
{
  "text": "View PDF",
  "type": "button",
  "btn_type": "filled_button",
  "width": 2,
  "onclick": {
    "1": {
      "type": "popup",
      "fullscreen": true,
      "title": "",
      "element_id": "pdf_popup",
      "ui": [{
        "type": "container",
        "color": "#FFFFFF",
        "width": 800,
        "border_radius": 12,
        "padding_top": 24, "padding_left": 24,
        "padding_right": 24, "padding_bottom": 24,
        "ui": [
          {
            "type": "row",
            "cross_axis_alignment": "center",
            "margin_bottom": 16,
            "ui": [
              { "type": "text", "text": "Document Title", "font_size": 20, "font_weight": "600" },
              { "type": "spacer" },
              {
                "type": "button", "btn_type": "outlined_button",
                "icon": "download", "text": "Download",
                "width": 2, "height": 36,
                "onclick": { "1": { "type": "navigation", "url": "{{pdf_url}}", "navigation_type": "url" } }
              }
            ]
          },
          {
            "type": "container", "width": 12,
            "ui": [{
              "type": "pdf", "url": "{{pdf_url}}",
              "width": 12, "height": 800,
              "pdf_type": "url"
            }]
          }
        ]
      }]
    }
  }
}
```

**Key rules:**
- Use `fullscreen: true` on popup for proper overlay
- Container width: `800` (pixels, not grid) for readable PDF size
- PDF widget: `type: "pdf"`, `pdf_type: "url"`, `height: 800`
- Download button: `navigation` with `navigation_type: "url"` opens in new tab
- NEVER use `open_url` action for PDF viewing — it navigates away from the page
