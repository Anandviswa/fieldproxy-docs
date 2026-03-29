# FieldProxy Mobile App — Alignment & Spacing Patterns

Verified patterns from EFS Cleaner home screen and other mobile builds. These solve real rendering issues on mobile.

---

## 1. Empty Container Spacers

Empty containers (`"ui": []`) provide margin-based spacing but **will not render color or background**. Use them purely as invisible spacers.

```json
{
  "ui": [],
  "type": "container",
  "element_id": "spacer_example",
  "margin_bottom": 6
}
```

**Rule:** If you need a visible colored spacer, you MUST put a child element inside (see Divider pattern below).

---

## 2. Divider / Horizontal Line

`type: "divider"` does not render on mobile. Use a container with `height: 1` and a dummy text child inside — without the child, the color won't render.

```json
{
  "ui": [
    {
      "text": " ",
      "type": "text",
      "color": "#F0ECEB",
      "font_size": 14,
      "element_id": "divider_text",
      "font_weight": "600"
    }
  ],
  "type": "container",
  "color": "#C6C6C3",
  "width": 4000,
  "height": 1,
  "element_id": "divider_line",
  "margin_top": 8,
  "margin_bottom": 16
}
```

**Key points:**
- `width: 4000` forces the line to span full width (overflows are clipped)
- `height: 1` keeps it thin like a real divider
- The text child color should be close to the container color so it's invisible
- `margin_top` / `margin_bottom` control breathing room above and below

---

## 3. Badge / Pill — Prevent Color Expansion

When a colored badge container sits inside a `row`, the badge background can stretch to match the row height (e.g., matching a taller button). Fix this by wrapping the badge in a parent container with `main_axis_size: "min"`.

```json
{
  "ui": [
    {
      "ui": [
        {
          "text": "Not Clocked In",
          "type": "text",
          "color": "#92400E",
          "font_size": 13,
          "element_id": "badge_text",
          "font_weight": "600"
        }
      ],
      "type": "container",
      "color": "#FEF3C7",
      "element_id": "badge_inner",
      "padding_top": 6,
      "padding_bottom": 6,
      "padding_left": 14,
      "padding_right": 14,
      "border_radius": 20
    }
  ],
  "type": "container",
  "element_id": "badge_wrap",
  "main_axis_size": "min"
}
```

**Rule:** Always wrap colored pill/badge containers in a parent with `main_axis_size: "min"` when placed inside a row alongside taller elements.

---

## 4. Row onclick Does NOT Work

Rows (`type: "row"`) do not support `onclick` on mobile. If you need a tappable row, wrap it in a container and put the `onclick` on the container.

```json
{
  "ui": [
    {
      "ui": [
        { "icon": "alert-triangle", "type": "icon", ... },
        { "text": "Report a Missed Punch", "type": "text", ... }
      ],
      "type": "row",
      "element_id": "inner_row",
      "cross_axis_alignment": "center"
    }
  ],
  "type": "container",
  "element_id": "tappable_container",
  "onclick": {
    "1": {
      "type": "navigation",
      "app_id": "xxx",
      "view_id": "target_view",
      "element_id": "nav_action"
    }
  }
}
```

---

## 5. Icon + Text Spacing in Rows

When placing an icon next to text in a row, use `margin_right` on the icon AND an empty spacer container between them for fine-tuned control.

```json
{
  "icon": "alert-triangle",
  "type": "icon",
  "color": "#F97316",
  "icon_size": 14,
  "element_id": "my_icon",
  "margin_right": 12
},
{
  "ui": [],
  "type": "container",
  "element_id": "icon_text_spacer",
  "margin_right": 10
},
{
  "text": "Label Text",
  "type": "text",
  "color": "#78350F",
  "font_size": 13,
  "element_id": "my_label",
  "font_weight": "600"
}
```

**Why both?** `margin_right` on the icon provides base spacing. The spacer container gives additional fine-tuned gap that's easier to adjust without touching the icon definition.

---

## 6. Long Text Wrapping with `expanded: true`

On mobile, text in a row will overflow horizontally if the content is too long. Add `expanded: true` to the text widget so it wraps to the next line.

```json
{
  "text": "{{data.facility_name}}",
  "type": "text",
  "color": "#1F2937",
  "font_size": 13,
  "expanded": true,
  "element_id": "facility_name",
  "font_weight": "600"
}
```

**Rule:** `expanded: true` only works on `type: "text"` — it is silently ignored on containers and buttons. Use it whenever text content comes from dynamic data that could be long.

---

## 7. Info Card Pattern (Grouped Details)

When showing related info (e.g., facility + time), wrap them in a single colored container with padding and border_radius. Add spacer containers between rows for fine spacing.

```json
{
  "ui": [
    {
      "ui": [
        { "icon": "map-pin", "type": "icon", "color": "#059669", "icon_size": 18, ... },
        { "text": "{{facility_name}}", "type": "text", "expanded": true, ... }
      ],
      "type": "row",
      "element_id": "facility_row",
      "margin_bottom": 8,
      "cross_axis_alignment": "center"
    },
    {
      "ui": [],
      "type": "container",
      "element_id": "info_spacer",
      "padding_bottom": 2
    },
    {
      "ui": [
        { "icon": "clock", "type": "icon", "color": "#6B7280", "icon_size": 16, ... },
        { "text": "Started at {{time}}", "type": "text", ... }
      ],
      "type": "row",
      "element_id": "time_row",
      "cross_axis_alignment": "center"
    }
  ],
  "type": "container",
  "color": "#ECFDF5",
  "element_id": "info_card",
  "padding_top": 14,
  "padding_bottom": 14,
  "padding_left": 16,
  "padding_right": 16,
  "border_radius": 12,
  "margin_bottom": 14
}
```

---

## 8. Button with Icon (Container-based)

For buttons with icons on mobile, use a container (not `type: "button"`) with a row inside. This gives full control over padding, border_radius, and colors. Use `main_axis_size: "min"` on the inner row so it doesn't stretch.

```json
{
  "ui": [
    {
      "ui": [
        { "icon": "log-in", "type": "icon", "color": "#FFFFFF", "icon_size": 12, "margin_right": 8, ... },
        { "text": "Clock In", "type": "text", "color": "#FFFFFF", "font_size": 11, "font_weight": "700", ... }
      ],
      "type": "row",
      "element_id": "btn_inner",
      "main_axis_size": "min",
      "cross_axis_alignment": "center"
    }
  ],
  "type": "container",
  "color": "#14532D",
  "element_id": "btn_container",
  "padding_top": 10,
  "padding_bottom": 10,
  "padding_left": 20,
  "padding_right": 20,
  "border_radius": 24,
  "onclick": { ... }
}
```

**Why container instead of button?** Container gives you `color` for background (not `background_color`), precise padding control, and the ability to put icons + text in a row.

---

## 9. Popup Close Pattern

Do NOT use `type: "close_popup"` — it doesn't work reliably. Use navigation with `back: true` instead.

```json
{
  "type": "navigation",
  "back": true,
  "element_id": "close_popup_action"
}
```

**Standard popup action sequence:**
1. `execute_query` — INSERT/UPDATE/DELETE
2. `update_data` — refresh specific data keys with `key` + `query`
3. `navigation` with `back: true` — close popup and refresh page

---

## 10. Mobile Card Spacing Cheat Sheet

| Element | Recommended Values |
|---|---|
| Card padding (all sides) | 16 |
| Card border_radius | 16 |
| Card border | `border_color: "#E5E7EB"`, `border_width: 1` |
| Section header font_size | 11, font_weight: "700", color: "#6B7280" |
| Section header margin_bottom | 12 |
| Badge padding (top/bottom) | 6 |
| Badge padding (left/right) | 14 |
| Badge border_radius (pill) | 20 |
| Badge border_radius (rect) | 8 |
| Icon-to-text gap | margin_right: 10–12 on icon |
| Row-to-row gap inside card | margin_bottom: 6–8 |
| Divider margin_top | 4–8 |
| Divider margin_bottom | 14–16 |
| Button font_size (compact) | 11 |
| Button padding (top/bottom) | 10 |
| Button padding (left/right) | 20 |
| Button border_radius (pill) | 24 |
| Background container padding | 12 (left/right), 16 (top/bottom) |
| Background container border_radius | 16 |
| Page margin (all sides) | 16 |

---

## Quick Checklist Before Shipping Mobile UI

- [ ] All colored badges wrapped with `main_axis_size: "min"` parent
- [ ] Dynamic text fields have `expanded: true`
- [ ] Dividers use container + text child pattern (not empty container)
- [ ] Tappable rows wrapped in container with onclick
- [ ] Icon-text pairs have spacer between them
- [ ] Popup actions use `navigation` + `back: true` (not `close_popup`)
- [ ] `color` used for container background (not `background_color`)
- [ ] No `expanded: true` on containers or buttons
