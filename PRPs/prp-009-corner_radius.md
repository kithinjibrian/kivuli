# PRP-009 — Corner radius

Status: built (session 9) on branch prp-009-corner_radius; merged into main
Discovery: session 9, 2026-10-17, from the user's brief and Rapidkoi's `screenshot-4-after-header-look.png`. *(you)*: from the brief; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: A `Surface` can round each corner alone (`radius_top_left` … `radius_bottom_left`), and a rounded node's content is clipped to its curve.

## OBJECTIVE

Rapidkoi's page sits flush in its frame, the header bar above and the tab rail on its right, and should curve only at its top-right corner, where the frame wraps it. Today `Surface.radius` rounds all four corners or none (the screenshot shows the page rounded all round). *(you)*

```
Page => Surface { background: Some(hex("#161616")), radius_top_right: Some(12), ..Surface.theme() },
```

## WHAT GTK DOES WITH A RADIUS (probed, session 9)

A probe (C, GTK 4.22, a window on a private broadway display, so nothing on screen) drew a box with `border-top-right-radius: 24px` and a child with its own background, and read GTK's render tree:

- **Overflow visible (GTK's default):** the box's own background is drawn inside a rounded clip (corners 0 24 0 0), but **the child isn't clipped**: content reaching the corner would show square past the curve.
- **Overflow hidden (`gtk_widget_set_overflow(…, GTK_OVERFLOW_HIDDEN)`):** the child is drawn inside a second rounded clip with the same per-corner shape: clipped to the curve.
- GTK's docs say `set_overflow` is "provided for widget implementations and should not be used by application code". kivuli is the widget layer here, managing widgets for apps, so it qualifies; apps never call it.

So kivuli turns overflow hidden on for a node whose looks round any corner, and back to visible when they no longer do. *(proposed)*

## CONTEXT

- Starting state: kivuli 0.3.0; `src/theming.la` (`Surface` with `radius: Option<i64>`, the atomic rules, `merged`, the size check), `src/styling.la` (`apply_style`, classes), `src/looks.la` (`Style.classes`); lugha-gtk 0.1.23.
- Ending state: those changed; tests in `test/theming.la`, `test/live_look.la`; `examples/frame.la`; lugha-gtk 0.1.24; kivuli 0.4.0.
- GTK functions needed, checked against `gtkwidget.h` / `gtkenums.h`: `gtk_widget_set_overflow(GtkWidget *, GtkOverflow)` and `gtk_widget_get_overflow` (tests), `GtkOverflow`: `VISIBLE` 0, `HIDDEN` 1. New in lugha-gtk 0.1.24.
- `src/views.la` (499 lines, DECISION-018) is untouched: this adds no modifier.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **Four new optional fields in `Surface`**: `radius_top_left`, `radius_top_right`, `radius_bottom_right`, `radius_bottom_left` (`Option<i64>`, pixels). `radius: Some(n)` keeps meaning all four, so existing apps don't change. *(you; proposed: separate fields, not a `Corners` struct)*
  - Why separate fields: each corner is then a field like any other, so "later looks win field by field" needs no new rule; an app writes `radius_top_right: Some(12)` instead of `corners: Some(Corners { top_right: Some(12), ..Corners.none() })`; no nested `Option` to get wrong. The cost: `Surface` grows from 6 to 10 fields, which `..Surface.theme()` hides.
- **Merging**: `radius` is shorthand for all four corners. When looks merge, each look's `radius` first stands for its four corners, then the corners merge field by field: a later look's `radius` replaces every corner an earlier look set, and a later look's one corner replaces only that corner. *(proposed)*
- **Generated CSS: only the corners that are set.** All four set and equal: `border-radius: Npx` (class `kv-radius-N`, as today, so existing apps' classes don't change). Otherwise one rule per set corner: `.kv-radius-tr-12 { border-top-right-radius: 12px; }` (and `tl`, `br`, `bl`); unset corners stay the theme's. *(you; proposed: class names)*
- **Checked as `radius` is**: a negative corner is "the look … has a negative size" when the theme is built. *(you)*
- **Clipping**: a node whose merged surface rounds any corner gets `gtk_widget_set_overflow(HIDDEN)`, so its content follows the curve; one that doesn't is set back to `VISIBLE`. Done in `apply_style`, from a style field `paint` fills (`clips: bool`), like the classes. *(proposed)*
- **`examples/frame.la`**: the page curves only at the corner the frame wraps. To match Rapidkoi, its tab rail moves to the right, and the row behind the page takes the frame's look so the curve shows the frame's colour; the page sets `radius_top_right`. *(you; proposed: moving the rail right, and the frame look behind the page)*
- Docs: README (the `Surface` fields), docs/DESIGN.md (corner radius and the clip, from the probe). *(you)*
- Release: 0.4.0, a CHANGELOG entry, tag `v0.4.0` once every check passes; push on your go. *(you)*

### Must NOT Do
- No elliptical radii, no per-side borders, nothing beyond corner radius. *(you)*
- No new modifier (DECISION-018 stays open).

## ERROR HANDLING REQUIREMENTS

- A negative corner stops the app with a message when the theme is built, as `radius` does today.

## SECURITY CONSIDERATIONS

- Two new externs, each matching `gtkwidget.h`; the CSS is still generated only from checked numbers.

## TESTS TO WRITE

Pure (`test/theming.la`):
- [x] one corner: its rule and class only (`kv-radius-tr-12`).
- [x] all four equal: `border-radius` as today (`kv-radius-6`); all four set but different: four corner rules.
- [x] `radius` with one corner: that corner overrides, the other three follow `radius`.
- [x] merging: a later look's corner replaces only that corner; a later `radius` replaces every corner.
- [x] a negative corner is refused.
- [x] paint: `clips` is true for a node whose looks round a corner, false otherwise.

Live GTK (skipped without a display), `test/live_look.la`:
- [x] a node whose look rounds one corner gets that corner's class and overflow hidden; when the look goes (a message), the class goes and overflow is visible again.
- [x] lugha-gtk `test/look.la`: `gtk_widget_set_overflow` / `get_overflow` set and read back.
- [ ] The curve itself (what's drawn) is checked by hand in `examples/frame.la`: GTK's API doesn't read a radius back.

## ROLLBACK PLAN
- Branch `prp-009-corner_radius` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.24 is additions only.

## ACCEPTANCE CRITERIA
- [x] A look can round any corner alone; `radius` behaves as before; a rounded node's content is clipped to its curve.
- [x] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG updated; 0.4.0 tagged; merged into `main`. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/frame.la`: the page meets the header and the rail with one curved corner, top right; the other three are square.

## NOTES FROM THE BUILD

- **Every corner rule for each radius a theme uses.** A merged node can mix corners from different looks (`Round` all 6, then `Corner` top right 12 gives `kv-radius-tl-6` …), and kivuli's stylesheet is built once from the looks alone. So `css` writes, for every radius value any look uses, the `border-radius` rule and the four corner rules: 5 small rules per distinct value, and every class a node can get exists. Classes on nodes are as proposed (`kv-radius-6` when all four are equal).
- **Overflow is undone only where kivuli set it.** Some GTK widgets clip on their own; kivuli turns `HIDDEN` on only where the widget was `VISIBLE`, marks it with a `kv-clip` class, and sets `VISIBLE` back only where that mark is. Mutation-checked: without the undo, a node keeps clipping after its rounded look goes.
- **The probe** (C, on a private `gtk4-broadwayd` display, nothing on screen) is described in the PRP; its source stayed in the session scratchpad.
- **`examples/frame.la`**: the rail moved to the right (as in Rapidkoi), the row behind the page takes the frame's look so the curve shows the frame colour, and the page no longer aligns to the top (it fills the row's height).
- `src/views.la` untouched (499 lines): DECISION-018 still open.
