# PRP-020 — Sizing through `.style`: width, min and max

Status: draft, for the user's approval (session 22)
Discovery: session 22, 2026-10-19 (DECISION-023's next group). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: a `Sizing` value in `.style([...])`: an exact width and height, a minimum and a maximum of each; `.width(n)` and `.height(n)` (which were minimums) go.

## OBJECTIVE

GTK has minimum sizes only: its CSS has `min-width` and `min-height` but no `width`, `max-width` or `max-height` (session 22: "No property named width"), and `gtk_widget_set_size_request` is a minimum too. kivuli's `.width(n)` has always been a minimum under a misleading name. This gives every node an exact size, a minimum and a maximum, honestly, through the one door: minimums as GTK's CSS, and an exact size or a maximum through a small clamp widget added to lugha-gtk. *(you)*

This is the target. Each example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Sizing, Width, column, row, text };

fun view(m: Model): View<Msg> = column([
    // Exactly 320 wide.
    column(rows).style([Sizing { width: Width.Px(320), ..Sizing.theme() }]),
    // A readable column: never wider than 560, however wide the window.
    text(m.body).wrap().style([Sizing { max_width: Width.Px(560), ..Sizing.theme() }]),
    // Between the two.
    column(cards).style([Sizing { min_width: Width.Px(200), max_width: Width.Px(400), ..Sizing.theme() }]),
    // What `.width(70).height(40)` meant: at least.
    text("Size").style([Sizing { min_width: Width.Px(70), min_height: Width.Px(40), ..Sizing.theme() }]),
    // Heights the same way.
    column(log).style([Sizing { max_height: Width.Px(200), ..Sizing.theme() }]),
])
```

## CONTEXT

- Starting state: kivuli 0.14.0. `.width(n)` / `.height(n)` set `Style.width` / `Style.height` and `gtk_widget_set_size_request` (minimums). `.menu` already wraps a node in the live tree without taking a place in its path (`Live.Menu`, PRP-014). lugha-gtk 0.1.29 has C helpers in `src/c/`.
- Ending state: `Sizing` beside the other style values; `Sheet.sizing`; minimums as CSS; a clamp widget in lugha-gtk 0.1.30 (new C helper and externs); `Live.Clamp`; `.width`, `.height`, `Style.width`, `Style.height` gone; every example, test and doc migrated; kivuli 0.15.0.
- GTK used: CSS `min-width`, `min-height` (parsed in session 22). The clamp is a `GtkWidget` subclass written in C (`G_DEFINE_TYPE`), with `measure` and `size_allocate` over one child; tests read sizes with `gtk_widget_measure` (in `gtkwidget.h`, a new extern).
- Open decisions: none (this continues DECISION-023).

## IMPLEMENTATION REQUIREMENTS

### The value *(you)*
```
public struct Sizing {
    width: Width,       // exactly
    min_width: Width,
    max_width: Width,
    height: Width,      // exactly
    min_height: Width,
    max_height: Width,
}
Sizing.theme()          // every field the theme's
```
- `Sizing` implements `Styling`; `Sheet` gains `sizing`; fields merge one by one, the last that sets each winning.
- **How the three combine**, as CSS: an exact size is kept between the minimum and the maximum, and the minimum wins over the maximum. *(proposed)*
- **Never smaller than the content can be**: a node is never given less than its content's own minimum (GTK can't draw a widget smaller than that); a maximum below it gives the content's minimum. Wrapping text has a small minimum, so `max_width` on a wrapping text works as a readable column. *(proposed)*

### The clamp *(you: a widget in lugha-gtk)*
- **lugha-gtk 0.1.30**: `lgtk_clamp_new()`, `lgtk_clamp_set_child(clamp, child)`, `lgtk_clamp_set_limits(clamp, min_width, max_width, min_height, max_height)` (`-1`: no limit), in `src/c/clamp.c`, a `GtkWidget` subclass with one child. Its size is the child's, kept within the limits (and never below the child's minimum); given more room than its maximum, it gives its child the maximum, at the start (as CSS does; `.align` on the node places the whole thing). Plus `gtk_widget_measure` for tests. Pushed and tagged under the standing go.
- **In kivuli**, a node is put in a clamp only when its sizing has an exact size or a maximum (a minimum alone is CSS, no clamp). `Live.Clamp(node, clamp)` wraps it and takes no place in a path, as `Live.Menu` does.
- **Where styles go**: the node's own classes (its font, background, border, padding, margin) stay on the node; how it sits in its parent (`.expand`, `.align`, `.place`) goes on the clamp, which is what the parent holds. *(proposed)*
- **Gaining or losing the clamp rebuilds the node**: the diff turns a style change that adds the first exact size or maximum, or removes the last, into a `Replace` of that node, so a widget is never re-parented into a clamp while live. Changing the numbers is a `SetStyle` that resets the clamp's limits. *(proposed)*

### CSS *(proposed classes)*
- `kv-minw-{n}` → `min-width: {n}px`; `kv-minh-{n}` → `min-height: {n}px`.

### Checks (stop the app with a message) *(proposed wording)*
- `Sizing.max_width: Width.Px(-1): a size can't be negative` (any field); `Sizing: min_width 300 is more than max_width 200` (and heights).

### What goes *(you)*
- The `.width(n)` and `.height(n)` modifiers, `Style.width`, `Style.height`, and kivuli's `gtk_widget_set_size_request` calls.
- `.size(n)` (an icon's or picture's box), `.expand()`, `.align(…)` and `.place(…)` stay modifiers.

### Migration *(proposed)*
- `.width(n)` → `Sizing { min_width: Width.Px(n), .. }` and `.height(n)` → `min_height`, keeping today's looks. Examples get a helper where it repeats (`at_least(w, h)`), and `examples/borders.la` gains a "Sizing" section: an exact width, a maximum on wrapping text, a minimum, a clamp between the two.
- Tests that used `.width(n)` as an arbitrary modifier (PRP-019 moved several there) switch to `.tooltip(…)`, same intent.
- Rapidkoi is not touched; the CHANGELOG lists the change.

### Must NOT Do
- No percentages, no `auto`, no aspect ratio (pictures have `.fit`).

## ERROR HANDLING REQUIREMENTS

- Bad values are programmer errors: `paint` returns `Err(message)` and the app stops (`kivuli: …`).

## SECURITY CONSIDERATIONS

- New externs (`lgtk_clamp_*`, `gtk_widget_measure`) match their C prototypes (`gtkwidget.h`; the clamp's own header in lugha-gtk). The clamp holds its child with GTK's own parenting (`gtk_widget_set_parent`), unparented in `dispose`. No new libraries.

## TESTS TO WRITE

lugha-gtk `test/clamp.la` (needs a display):
- [ ] a clamp's measured size is its child's, kept within its limits; a maximum below the child's minimum gives the child's minimum; `-1` is no limit; changing limits re-measures; the child is freed with the clamp.

Pure:
- [ ] `test/sizing.la` (new): `Sizing.theme()` sets nothing; fields merge one by one; the OBJECTIVE's `view` reads as written.
- [ ] `test/sheets_border.la`: CSS for `min_width` and `min_height`; an exact size and a maximum give no CSS (the clamp takes them); each check message.
- [ ] `test/diff.la` (or a new `test/diff_sizing.la`): adding the first maximum is a `Replace`; changing a maximum is a `SetStyle`; removing the last is a `Replace`; a minimum alone never is.
- [ ] Every test that used `.width` or `.height`: rewritten, same intent.

Live GTK (skipped without a display), `test/live_style.la` (or a new `test/live_sizing.la`):
- [ ] a node with a maximum sits in a clamp with those limits and measures no wider; an exact width measures exactly that; a minimum alone is a class, no clamp; `.align` lands on the clamp; a `SetStyle` of new numbers resets the limits on the same widgets.
- [ ] How sizes look in a window: checked by hand in `examples/borders.la`.

## ROLLBACK PLAN
- Branch `prp-020-sizing` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.30 is additions only.

## ACCEPTANCE CRITERIA
- [ ] The OBJECTIVE's code compiles and its tests pass unchanged.
- [ ] No `.width(` or `.height(` modifier left in kivuli, its examples, tests or docs.
- [ ] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines (a split proposed first if one would pass it).
- [ ] README, docs/DESIGN.md, CHANGELOG (Breaking), CLAUDE.md, DECISIONS updated; the tutorial page; 0.15.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built with `-o examples/<name>`; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/borders.la`'s "Sizing" section: resize the window and the readable column stops at 560 px.
