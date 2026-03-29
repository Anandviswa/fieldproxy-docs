---
name: fieldproxy-architecture
description: |
  FieldProxy platform internals, Dart code architecture, ViewController, state management, API layer, and auth system. Load this skill ONLY when modifying Dart source code in the platform, debugging runtime issues, or understanding how the JSON engine works internally.
---

# FieldProxy Architecture — Tier 3 Skill (Dart Platform Internals)

## Architecture Overview

Flutter (Dart) web application using **GetX** state management + **AutoRoute** navigation. All views are JSON-driven — the `ViewController` interprets JSON config stored in the `fp_apps` database table.

```
JSON config (fp_apps.compiled_data)
  → ViewController.getChildren() dispatches widget types
    → Flutter widgets rendered reactively via Obx()
      → User interactions → onclick action chains → SQL/API calls
```

## Core: ViewController

Located: `view/controllers/view_controller.dart`

The single engine that powers everything. Key methods:

| Method | Purpose |
|--------|---------|
| `getChildren(ui, {index})` | Main dispatch — reads `type` field, returns Flutter widget |
| `sanitizeStringSync(string, {index})` | Template resolution — replaces `{{...}}` with live data |
| `sanitizeStringAsync(string, {index})` | Async version — returns raw object if entire string is a template |
| `getNestedData(path, {index})` | Walks dot-separated path to resolve data |
| `elementClicked(action, {index})` | Executes onclick action chains |
| `checkVisibilityCondition(element, index)` | Evaluates visibility/disabled/required conditions |

### Template Resolution Flow

`sanitizeStringSync` finds all `{{...}}` patterns, splits on `.`, calls `getNestedData`:

```
"{{items.data.[index].title}}"
  → split: ["items", "data", "[index]", "title"]
  → getNestedData walks: viewData["items"]["data"][runtimeIndex]["title"]
  → returns resolved string
```

Special path handling:
- `[index]` → uses runtime integer index (from list iteration)
- `[0]`, `[1]` → integer index on a List value
- `.0` (no brackets) → string key `"0"` in a Map
- `index` (bare, no brackets) → special: in viewData path resolves to `"$index"` string key

## Reactive State Maps

All state is stored in `RxMap` observables, keyed by `element_id`:

| Map | Key structure | Purpose |
|-----|--------------|---------|
| `viewData` | `viewData["query_name"]` → query result | Data from SQL queries |
| `inputData` | `inputData["element_id"]["$index"]` → value | User input state |
| `textControllers` | `textControllers["element_id"]["$index"]` → TextEditingController | Input controllers |
| `visibilityConditions` | `visibilityConditions["element_id"]["$index"]` → {value, expression} | Show/hide state |
| `disabledConditions` | Same structure | Enabled/disabled state |
| `requiredConditions` | Same structure | Required validation state |
| `selectedElements` | `selectedElements["element_id"]` → value | Selection state |
| `onclickResults` | `onclickResults["element_id"]` → query result | Results from execute_query actions |
| `colors` | `colors["element_id"]["$index"]` → hex string | Dynamic color state |
| `kanbanControllers` | Per kanban instance | Kanban board state |
| `calendarTaskControllers` | Per calendar instance | Calendar task state |

The `$index` is always a string representation of the integer loop index (e.g. `"0"`, `"1"`).

## API Layer

Located: `api.dart` — `API` class

All data operations go through a single PostgreSQL passthrough endpoint:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `getData(query)` | `POST /data/raw` | Execute SQL, return results |
| `getDataPaginated(query, page, limit)` | `POST /data/raw` | Auto-paginated SQL |
| `apiCall(method, url, headers, body)` | Any external URL | External API calls |
| `uploadFileWeb(file)` | `POST /file/upload` | File upload to CDN |
| `sendEmailNotification(...)` | `POST /apps/notifications/email` | Email sending |
| `sendAppNotification(...)` | `POST /notifications/send` | Push notifications |

## Auth System

### Dual token system
- **User token**: Full access — stored in GetStorage via `Functions().token`
- **App token**: Scoped access for public apps — stored via `Functions().appToken`
- `getEffectiveAuthToken()` resolves which to use (user token takes priority)

### Public app access
Apps with `"visibility": "public"` can be accessed via:
- URL flag: `?public=1`
- App token: `?app_token=TOKEN` (validated against configurable `auth_table`)

### Middleware
- `AuthMiddleware` — redirects unauthenticated users to `/login`, allows public bypass
- `GuestMiddleware` — redirects authenticated users away from `/login` to `/home`

## Navigation System

### Dual router (parallel)
- **GetX Router** (`routes/app_pages.dart`) — legacy routes, middleware
- **AutoRoute** (`router/app_router.dart`) — in-view navigation, deep links

### URL format
```
/{app_id}/{view_id}?param1=value1&param2=value2
```

### Argument passing priority
1. Constructor params (from navigation action)
2. `NavigationArgumentsService` (persisted to GetStorage, survives refresh)
3. URL query params
4. `Get.arguments`

## Database Tables

| Table | Purpose |
|-------|---------|
| `fp_apps` | App configs — `compiled_data` (JSON), `version`, `visibility` |
| `fp_users` | User accounts — email, password (bcrypt), org_id |
| `fp_user_logs` | User activity logs |
| `fp_organization_key_value` | Org-level settings (key-value store) |
| Custom tables | App-specific data (created per project) |

## Condition System

Visibility, disabled, and required conditions use SQL queries:

```
checkVisibilityCondition(element, index)
  → sanitizeStringAsync(condition expression)
  → if contains SELECT: getData(condition) → evaluate boolean result
  → else: evaluateExpression(sanitized string) → string comparison
```

Results are cached in the corresponding `RxMap` and throttled to prevent excessive API calls. Widgets wrap content in `Obx(() => Visibility(visible: condition_value, child: ...))`.

## Key Files Reference

| File | What it does |
|------|-------------|
| `view/controllers/view_controller.dart` | Core engine — ALL rendering + actions |
| `api.dart` | HTTP client — all backend calls |
| `constants/functions.dart` | GetStorage helpers, token management, utils |
| `constants/widgets.dart` | `calculateDynamicWidth()`, spacers, loaders |
| `router/app_router.dart` | AutoRoute config |
| `routes/app_pages.dart` | GetX route config |
| `middleware/auth_middleware.dart` | Auth + guest middleware |
| `services/navigation_arguments_service.dart` | Argument persistence |
| `view/screens/elements/` | All widget implementations |
| `view/screens/elements/inputs/` | All input type implementations |

## Widget Type Registry

The `getChildren()` method dispatches on `element["type"]`:

**Layout:** container, container_with_title, card, column, row, wrap, expandable_card, sidebar, tab_group, vertical_tab_group

**Display:** text, date_time, image, file, icon, pdf, webview

**Button:** button (with btn_type: filled_button, outlined_button, text_button, icon_button, dropdown_button)

**Input:** input (with input_type: text, number, email, phone, date, dropdown, checkbox, file, image, mcq, rating, signature, location, scan, step_counter, table, catalogue)

**Charts:** donut_chart, half_donut_chart, bar_chart, counter, grouped_counter_1

**Data:** list, table, pivot_table, form, kanban, calendar, calendar_full, calendar_task_widget

**Utility:** spacer, space_widget, gutter, combo_input, import_data, export_data, map_widget
