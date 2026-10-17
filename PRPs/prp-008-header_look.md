# PRP-008 — Header look

Status: built (session 8) on branch prp-008-header_look; merged into main
Discovery: session 8, 2026-10-17, from the user's brief, Rapidkoi's screenshot (`screenshot-3-after-amendment-2.png`) and its inspiration (`inspiration-opera-frame.png`). *(you)*: from the brief; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: The root's header bar takes the app's looks (`.header_style([…])`), and a `Surface` can turn a node's theme edges off (`flat`), so a header bar and a rail read as one frame.

## OBJECTIVE

Rapidkoi paints its frame #242424 with the tab rail on it. Its header bar should be the same surface: the same colour, no line under it, so the top and the side read as one L-shaped frame, as in the inspiration. Today the bar keeps the theme's background and a dark line along its bottom edge. *(you)*

```
enum Look { Frame, Selected }

fun surface(l: Look): Surface = match l {
    Frame => Surface { background: Some(hex("#242424")), flat: Some(true), ..Surface.theme() },
    ...
};

row([rail(m).style([Look.Frame]), home(m)])
    .theme(theme([Look.Frame, Look.Selected], surface))
    .header([back, forward], address(m).expand(), [menu])
    .header_style([Look.Frame])
```

## WHAT DRAWS THE LINE (from the themes' CSS)

- **GTK 4.22's Default theme** (dark: `Default-dark.css` in libgtk's resources) styles `headerbar` with `border-width: 0 0 1px; border-style: solid; border-color: #070707` and `background: #1b1b1b linear-gradient(to top, #262626, #2b2b2b)`; when the window isn't focused (`:backdrop`), `border-color: #202020; background-color: #353535; background-image: none`.
- **Yaru-dark** (`/usr/share/themes/Yaru-dark/gtk-4.0/gtk.gresource`, `gtk-dark.css`) does the same with `border-color: black` and `background: #131313 linear-gradient(to top, #222222, #222222)`; backdrop `#181818` / `#2c2c2c`.
- **Neither gives the header bar a `box-shadow`** (the only one is `box-shadow: none` under `.solid-csd`). **The line is a 1 px bottom border**; the screenshot's 2 px is that logical pixel on a scaled display (the panel is 3072×1920; the screenshot 2000 px wide).
- So the header needs `border-width: 0` (the line) and the existing `background` rule (which already clears the theme's `background-image`, the gradient). kivuli's rules are at application priority (600), over the theme's (200), so they hold in every state, backdrop included. Many other themed widgets (buttons, entries) draw a `box-shadow` as well, so the new field clears both.

## CONTEXT

- Starting state: kivuli 0.2.0; `src/theming.la` (`Surface`: background, text, border, radius; atomic rules), `src/paint.la` (classes from the root's theme), `src/views.la` (487 lines), `src/looks.la` (Style), `src/styling.la` (`apply_style`: a framed root styles its body), `Live.Framed(bar, …)`; lugha-gtk 0.1.23.
- Ending state: those changed; tests in `test/theming.la`, `test/lib.la`, `test/live_look.la`; `examples/frame.la`'s header takes the rail's look; kivuli 0.3.0.
- GTK functions needed: none new (classes go on the bar with `gtk_widget_add_css_class` and `lgtk_remove_css_classes_with_prefix`, as on any node; `lgtk_widget_rgba` reads a colour back in tests). No lugha-gtk release.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **`.header_style(looks: L[])`** on the root, beside `.header(…)`: the header bar takes those looks, merged left to right as `.style` does. A root style field (`header_looks`, its classes filled by `paint` as `header_classes`), so changing it between views is a `SetStyle` that restyles the bar in place. Ignored without a `.header` (as `.theme` is below the root). *(proposed: a separate modifier, not a fourth `.header` argument: `.header`'s signature stays as it is (no break), it reads like `.style`, and every look stays a style field kivuli already diffs)*
- **`flat: Option<bool>` in `Surface`**: `Some(true)` turns the theme's edges off, `box-shadow: none; border-width: 0;`, on any node. `None` (unset) leaves the theme's, as now; `Some(false)` is the same as unset. A look that sets both `border` and `flat` keeps its `border`: merged in kivuli's code (the border rule is emitted, the flat rule's `border-width` isn't), so the result never depends on stylesheet order. *(proposed: the name `flat` (the edges a theme draws: shadow and border line), and one field for both, since the header needs the border and other widgets the shadow)*
- The generated CSS: `.kv-flat { box-shadow: none; border-width: 0; }` (or `box-shadow: none;` alone when the merged surface also has a `border`, as `.kv-flat-shadow`). *(proposed: class names)*
- **The header bar's window controls stay GTK's**: kivuli's classes go on the `headerbar` node only, and a `Surface` has no field that reaches its children's controls. *(you)*
- `examples/frame.la`: one `Frame` look for the rail and the header (`#242424`, flat), so they read as one frame. *(you)*
- Docs: README (`.header_style`, `flat`), docs/DESIGN.md (the header bar's look; what draws the line in the themes). *(you)*
- Release: 0.3.0, a CHANGELOG entry, tag `v0.3.0` once every check passes; push on your go. *(you)*

### Must NOT Do
- No custom window controls; no change to the header bar's layout. *(you)*
- No per-node inline CSS: looks only. *(you)*
- No `.dark()` (DECISION-017 stays deferred).

## ERROR HANDLING REQUIREMENTS

- As for `.style`: a header look missing from the theme, or `.header_style` without a `.theme`, stops the app with a message, found by `paint` before GTK.
- `flat` can't be wrong (a bool).

## SECURITY CONSIDERATIONS

- No new externs or libraries; the CSS is still generated only from checked values.

## TESTS TO WRITE

Pure:
- [x] `test/theming.la`: a flat surface's CSS (`kv-flat`) and classes; flat with a border (the border wins: `kv-flat-shadow`); a theme using both; `flat: Some(false)` is unset.
- [x] `test/theming.la` (paint): `.header_style` fills `header_classes` from the theme; a missing header look and `.header_style` without `.theme` are errors.
- [x] `test/lib.la`: `.header_style` lands in the root style (and stays outside the frame, as `.header` keeps modifiers).
- [x] `test/diff.la`: a change of header looks is a `SetStyle` on the root.

Live GTK (skipped without a display), `test/live_look.la`:
- [x] the header bar gets the look's classes; changing the header looks between views swaps them in place; GTK's own classes on the bar (`titlebar`) stay.
- [x] the app's CSS wins over the theme's on the bar: a header look's `text` colour is the bar's colour (`gtk_widget_get_color` through `lgtk_widget_rgba`). Its background and border can't be read back through GTK's API; they come from the same rule and priority, and are checked by hand in `examples/frame.la`.

## ROLLBACK PLAN
- Branch `prp-008-header_look` from `main`; abandon by leaving it unmerged. No lugha-gtk change.

## ACCEPTANCE CRITERIA
- [x] A header bar can take the app's looks; `flat` turns a node's theme edges off; apps without them look as before.
- [x] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG updated; 0.3.0 tagged; merged into `main`. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built.
- For the user: `lughac run examples/frame.la`: the header bar and the rail are one #242424 surface with no line between them, focused or not; the window controls look as in other apps.

## NOTE: views.la is at the limit

`src/views.la` is 487 lines and this adds about 12 (≈499). Its modifiers can't move out (an `impl` must sit with its type, PRP-002), and the one large body that could, `map`, would need `views` and a new module to import each other, against CLAUDE.md's rule that imports run one way. Recorded as an open decision (DECISION-018) for the *next* modifier; it doesn't block this PRP.

## NOTES FROM THE BUILD

- **As proposed.** `flat` merges like the other fields (a later look's `Some(false)` undoes an earlier `Some(true)`); the border-or-not choice is made in `theming.rules`, so the generated rules never compete in the cascade.
- **The diff test lives in `test/diff_header.la`**: `test/diff.la` is at 498 lines.
- **GTK keeps its own `titlebar` class on the bar** (checked in the live test); kivuli replaces only its `kv-` classes there.
- **Mutation-checked:** without the bar's classes in `apply_style`, the header never takes its looks.
- **By hand:** the bar's background and the missing line can't be read back through GTK's API; `examples/frame.la` shows them.
- `src/views.la` is now 499 lines: DECISION-018 must be settled before the next modifier.
