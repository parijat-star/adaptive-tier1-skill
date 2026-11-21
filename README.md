# Adaptive Apps Tier 1 Certification Skill for Antigravity

An [Antigravity](https://antigravity.google) agent skill for auditing, implementing, and verifying **Tier 1 Adaptive Apps Certification** requirements on Android applications (large-screen, desktop windowing, ChromeOS/Googlebook, freeform resizing, keyboard/Tab navigation, touch targets, hover tooltips, scrollbars, context menus, and cross-app/touch drag-and-drop) before pushing releases or uploading APKs.

Distilled from real-world Googlebook / Desktop Windowing certification audits and implemented in [Wayfinder](https://github.com/parijat-star/wayfinder).

---

## What This Skill Covers

| Tier | Areas Checked & Enforced |
| :--- | :--- |
| **Tier 3 — Adaptive Ready** | Edge-to-edge (`enableEdgeToEdge()`), cutout mode `ALWAYS`, multi-window & multi-resume state continuity, physical keyboard, mouse/trackpad, and stylus parity |
| **Tier 2 — Adaptive Optimized** | Responsive layouts (`<600dp` / medium / expanded), `DialogProperties(usePlatformDefaultWidth = false)` + `FlowRow` for compact forms, dynamic `WindowInsets.Type.captionBar` tracking with `>= 144.dp` window-controls reserve & oversized-rect clamping, `>= 48×48dp` touch targets, visible `.focusable()` focus indicators, child-first `Tab`/`Enter`/`Arrow` keyboard navigation, touch long-press context menus, and pointer hover states |
| **Tier 1 — Adaptive Differentiated** | Scrubbable scrollbars (`Scrollbar_Display`), Material 3 tooltips on icon-only buttons (`Hover_Parity`), desktop `Tab` traversal between content/sidebar/toolbar (`Keyboard_Navigation`), and `FileProvider` `startDragAndDrop` for cross-app mouse drags & horizontal touch drags (`Drag_Drop_Support`) |

It also defines an **Automated Verification Gate** using Robolectric + Compose UI tests before pushing to GitHub.

---

## How to Use It in Antigravity

### Option A: Install Globally (Recommended — applies to all your Android apps)

Copy `skills/adaptive-tier1-certification` into your global Antigravity skills directory:

```bash
git clone https://github.com/parijat-star/adaptive-tier1-skill.git /tmp/adaptive-tier1-skill
mkdir -p ~/.gemini/config/skills
cp -r /tmp/adaptive-tier1-skill/skills/adaptive-tier1-certification ~/.gemini/config/skills/
rm -rf /tmp/adaptive-tier1-skill
```

Antigravity automatically discovers skills under `~/.gemini/config/skills/<name>/SKILL.md` across every workspace.

### Option B: Install Per-Repository (Shared with your team)

Copy `skills/adaptive-tier1-certification` into `.agents/skills/` at the root of your Android repository:

```bash
git clone https://github.com/parijat-star/adaptive-tier1-skill.git /tmp/adaptive-tier1-skill
mkdir -p .agents/skills
cp -r /tmp/adaptive-tier1-skill/skills/adaptive-tier1-certification .agents/skills/
rm -rf /tmp/adaptive-tier1-skill
```

Commit `.agents/skills/adaptive-tier1-certification/SKILL.md` so anyone opening the repo in Antigravity discovers it automatically.

---

## Triggering the Skill

Once installed, ask Antigravity things like:
- *"Audit this app for Adaptive Tier 1 certification and fix any gaps."*
- *"Verify caption bar resizing, keyboard Tab navigation, touch targets, tooltips, scrollbars, and drag-and-drop before I push the APK."*

## License

Apache License 2.0
