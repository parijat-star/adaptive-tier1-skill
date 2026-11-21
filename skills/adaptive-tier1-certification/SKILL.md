---
name: adaptive-tier1-certification
description: >-
  Audit, implement, and verify Tier 1 Adaptive Apps Certification requirements
  for Android apps (large-screen, desktop windowing, ChromeOS/Googlebook,
  freeform resizing, keyboard/Tab navigation, touch targets, hover tooltips,
  scrollbars, context menus, and cross-app/touch drag-and-drop) before
  publishing or uploading APKs to GitHub.
---

# Adaptive Apps Tier 1 Certification Skill

Use this skill whenever building, auditing, or preparing an Android application for GitHub/Play release to ensure it meets **Tier 3 (Adaptive Ready)**, **Tier 2 (Adaptive Optimized)**, and **Tier 1 (Adaptive Differentiated)** certification requirements.

---

## Pre-Release Certification Checklist

### 1. Tier 3 — Adaptive Ready (Baseline)
- **Configuration & Continuity**: Use `enableEdgeToEdge()`, `LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS`, and state preservation across rotation, folding/unfolding, and freeform window resizing with no artificial letterboxing or pillarboxing.
- **Multi-Window & Multi-Resume**: Support split-screen, multi-window, freeform desktop windows, and concurrent multi-resume state without crashing or losing navigation/selection state.
- **Input Parity**: Support physical keyboard text input (and dynamic connect/disconnect without relaunch), mouse/trackpad pointer navigation/scroll, and stylus pointer + text input.

### 2. Tier 2 — Adaptive Optimized (Required)
- **Responsive Layouts & Dialogs**:
  - Adapt to compact (`< 600dp`), medium, and expanded width classes.
  - On `AlertDialog`s containing forms or multiple buttons, pass `DialogProperties(usePlatformDefaultWidth = false)` with `Modifier.padding(horizontal = 16.dp).widthIn(max = 560.dp).fillMaxWidth()` so Android's `280dp` platform dialog width doesn't conflict with `OutlinedTextField` (`MinWidth = 280.dp`) or squeeze buttons vertically.
  - Use `FlowRow` (with `maxLines = 1, softWrap = false`) or responsive `BoxWithConstraints` stacking for multi-button action bars so buttons never wrap character-by-character on `360dp` phones.
- **UI Secondary Elements & System Caption Bar**:
  - When drawing a custom tab strip or header inside `WindowInsets.captionBar`, attach a `View.OnLayoutChangeListener` to re-read `rootWindowInsets.getBoundingRects(WindowInsets.Type.captionBar())` on every freeform resize (do **not** rely on `remember(capTop)` alone, as horizontal rects change without `capTop` changing).
  - Always enforce a minimum reserve (`>= 144.dp`) on the window-controls side (`-`, `□`, `×`) whenever `capTop > 0`, even if `getBoundingRects` omits a rect during a narrow freeform resize.
  - Ignore/clamp oversized bounding rects (`> 45%` of window width) so tabs/headers never shift to the center when maximized.
- **Touch Targets (`>= 48×48dp`)**:
  - Audit all interactive icon buttons on Home/main screens (toolbar, titlebar `+`, sidebar actions, statusbar cancel, list/grid rows) to ensure `>= 48.dp × 48.dp` hit targets (or `>= 48.dp` width in `44.dp` caption strips).
- **Drawable Focus States**:
  - Every focusable navigation and content item (sidebar items, header actions, list/grid rows) must use `.focusable(interactionSource)` + `collectIsFocusedAsState()` to render a visible focus indicator (e.g., `2.dp` `primary` border or highlight) when navigated with `Tab` or arrow keys.
- **Keyboard Navigation & Focus Traversal**:
  - Users must be able to `Tab` / `Shift+Tab` between Toolbar, Left Navigation Pane, and Main Content Area, and open/activate focused items with `Enter` / `Space` (plus `ArrowLeft`/`ArrowRight` to collapse/expand tree nodes).
  - Do **not** intercept `Enter`/`Space`/Arrow keys in a root `onPreviewKeyEvent` before focused children see them; reserve `onPreviewKeyEvent` for global `Ctrl`/`Alt`/`Meta` and `F1`–`F12` shortcuts, and use `onKeyEvent` for fallback navigation.
- **Context Menus (Mouse/Trackpad + Touch Parity)**:
  - Both mouse/trackpad right-click (`isSecondaryPressed`) **and** touchscreen stationary long-press (`onLongPress`) must open the context menu at accurate root coordinates. Do not hijack touch long-press solely for selection if it prevents opening the context menu.
- **Hover States**:
  - All interactive controls—especially tab close (`×`), tab add (`+`), and icon buttons—must expose visual hover feedback (`hoverable(interactionSource)` + `collectIsHoveredAsState()`).

### 3. Tier 1 — Adaptive Differentiated (Required Differentiators)
- **`Scrollbar_Display`**:
  - Render visible, mouse/trackpad-scrubbable scrollbars (`LazyListState`, `LazyGridState`, `ScrollState`) on every scrollable list, grid, sidebar, and detail/preview pane whenever content overflows.
- **`Hover_Parity`**:
  - Wrap every icon-only interactive button in Material 3 `TooltipBox` + `PlainTooltip` (`TooltipDefaults.rememberTooltipPositionProvider(TooltipAnchorPosition.Below)`) so hovering with mouse/trackpad displays informative labels and keyboard shortcuts.
- **`Keyboard_Navigation` (Desktop)**:
  - Ensure seamless `Tab` traversal from the main content area into the left sidebar and toolbar, `Enter`/`Space` activation on focused files/folders, and `Menu` / `Shift+F10` for keyboard context menus.
- **`Drag_Drop_Support` (Cross-App + Touch)**:
  - Support **cross-app** drag-and-drop by building a `FileProvider` `content://` `ClipData` and calling `view.startDragAndDrop(clipData, shadowBuilder, localState, View.DRAG_FLAG_GLOBAL or View.DRAG_FLAG_GLOBAL_URI_READ)` when dragging past window bounds with a mouse/trackpad.
  - Support **touch** drag-and-drop in both clamshell and touchview (e.g., horizontal touch drag past touch slop calling `startDragAndDrop`) without breaking vertical touch scrolling or stationary touch long-press context menus.
  - Accept incoming drops via `Modifier.dragAndDropTarget` + `activity.requestDragAndDropPermissions(dragEvent)` and also handle `localState` for in-app touch drops.

---

## Automated Verification Gate (Run Before GitHub Push)

Before committing or pushing an APK/release to GitHub, add/run Robolectric Compose UI tests covering:
1. **Freeform & Maximized Caption Insets**: Verify narrow freeform (`360dp`) reserves `>= 144.dp` for window controls and maximized (`1280dp`) clamps oversized rects so tabs stay left-aligned.
2. **Compact Phone Portrait (`360dp`)**: Verify dialog and banner buttons render horizontally (`width > height`, `height <= 56.dp`) with no vertical character-by-character wrapping.
3. **Touch Targets**: Assert `assertWidthIsAtLeast(48.dp)` and `assertHeightIsAtLeast(48.dp)` on Home/toolbar icon buttons.
4. **Keyboard Focus & Navigation**: Verify sidebar and main content items are `isFocusable()` and support `Tab` traversal.
5. **Touch Long-Press Context Menu**: Assert `performTouchInput { longClick() }` on content items opens the context menu.
6. **Cross-App Drag ClipData**: Verify `FileProvider` `content://` URIs are generated for local files and virtual paths are skipped.

Run the full build and test suite:
```bash
./gradlew :app:assembleDebug :app:testDebugUnitTest
```
Only push to GitHub once `:app:assembleDebug` and all unit/UI/fuzzing tests pass with 0 failures.
