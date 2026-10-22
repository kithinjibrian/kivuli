# PRP-057 — A scroll that fits its content

Status: approved (session 138, "build all")
Discovery: session 138, 2026-10-22 (rapidkoi's DECISION-037, part 1: its sidebar lists apps then a +, which should sit right after the last app until the list overflows, then stay at the bottom while the apps scroll, as Chrome's tab strip; GTK's `gtk_scrolled_window_set_propagate_natural_height`/`_width`). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `scroll_fit(child)`: a scroll as big as its content until its parent can't give that much, then scrolling.

## OBJECTIVE

```
column([scroll_fit(column(apps)), plus_button])     // + right after the apps; at the bottom once they scroll
```

*(you: A, a builder beside `scroll`)*

## REQUIREMENTS

- **`scroll_fit<Msg>(child: View<Msg>): View<Msg>`**: a `GtkScrolledWindow` asking for its child's natural height and width (`gtk_scrolled_window_set_propagate_natural_height`/`_width`, both *(proposed)*), so it's as big as its content; past what its parent gives, it scrolls. Unlike `scroll`, it doesn't take all the spare room (it doesn't expand unless `.expand()`ed).
- `View.Scroll` gains whether it fits: `Scroll(View<Msg>, bool)` *(proposed: one node kind, so the diff, paths and patches of scrolls are shared)*; `scroll(child)` is `Scroll(child, false)`, unchanged. A change from fitting to not patches in place (`SetFit`) on the same widget.
- **lugha-gtk 0.1.45**: `gtk_scrolled_window_set_propagate_natural_height`/`_width` and their getters (checked against `gtk/gtkscrolledwindow.h`). Pushed under the standing go.
- `views.la` (500 lines): the variant's field changes, no lines added.

### Must NOT Do
- No maximum height of its own (a parent's or `Sizing.max_height` does that), no horizontal-only or vertical-only fitting.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/scrolls.la` (new, pure): `scroll` and `scroll_fit`'s nodes; `map`; the diff: fitting changed is one patch in place; the child diffed inside as before.
- [ ] `test/live_scroll.la` (new, GTK): `scroll_fit` sets both propagations, `scroll` neither; changing it on update keeps the widget; a fitting scroll doesn't expand; in a column of a given height, a fitting scroll's measured natural height follows its content.
- [ ] lugha-gtk: both propagations set and read back.
- [ ] `examples/components.la` gains a "Fitting scroll" row (a list that grows by a button, a + after it, in a fixed-height box); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.53.0, CLAUDE.md; merged, tagged; pushed only on your go.
