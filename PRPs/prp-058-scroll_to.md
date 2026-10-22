# PRP-058 — Scrolling a node into view

Status: built (session 140); merged into main in session 140 (its own tests run and pass)
Discovery: session 138, 2026-10-22 (rapidkoi's DECISION-037, part 2: after an app opens or is selected, its sidebar scrolls so the app's button shows, without taking the focus from the address box; `Cmd.focus` scrolls but takes the focus; GTK 4.12's `gtk_viewport_scroll_to`). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Cmd.scroll_to(name)`: the nearest scroll around the node named `name` scrolls as little as it can to show it; the focus doesn't move.

## OBJECTIVE

```
Opened(id) => (Model { current: id, ..m }, Cmd.scroll_to("app-{id}"))
```

*(you: A, as little as possible: nothing if it's already fully visible)*

## REQUIREMENTS

- **`Cmd.scroll_to(name: string): Cmd<Msg>`** (`Cmd.ScrollTo(string)`): after the update is shown, as `Cmd.focus` (the node exists by then); the nearest `scroll`/`scroll_fit` around the node named `name` (`.name(…)`) scrolls so it's fully visible, at the nearest edge, nothing if it already is (`gtk_viewport_scroll_to` with GTK's default scroll info). No node by that name, or none inside a scroll: nothing happens. The keyboard focus doesn't move.
- `Cmd.map` and `Cmd.batch` carry it, as `Focus`.
- **lugha-gtk 0.1.45 or 0.1.46**: a C helper `lgtk_scroll_to(widget)` finding the nearest `GtkViewport` ancestor and scrolling it to the widget (Lugha can't test a widget's type), returning whether it found one; checked against `gtk/gtkviewport.h`. Pushed under the standing go.
- **Timing, probed first**: GTK applies the scroll when the viewport is next laid out; a node added in this very update is scrolled to once it has its size *(proposed: verified by a C probe; if GTK needs it later, kivuli defers to the next idle turn)*.

### Must NOT Do
- No centring (C, later if wanted), no smooth animation of our own (GTK's), no scrolling the window itself.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] `test/effects.la`: `Cmd.scroll_to`'s value, `map` and `batch` keep it.
- [x] `test/live_scroll.la` (GTK): the focus stays where it was after `Cmd.scroll_to`; an unknown name does nothing. *(The scroll position itself: a hidden window isn't laid out, so it's by hand; the C probe showed GTK's behaviour.)*
- [x] lugha-gtk: the helper scrolls a viewport to a child, and says no without a viewport.
- [ ] *(not done)* An example: every example with a list is a `sandbox` app, which can't send commands; rapidkoi is the first user.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.54.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **Probed first** (C, `scratchpad/probe/st2.c`): on a shown window `gtk_viewport_scroll_to` scrolls at the next layout (the last of 50 rows: 737 px), and a row added in the same turn is reached; on a hidden window nothing is laid out, so nothing moves. No deferral needed in kivuli.
- **lugha-gtk 0.1.46**: `lgtk_scroll_to` (`src/c/scrolling.c`), finding the nearest `GtkViewport` (Lugha can't test a widget's type). Pushed, tagged.
- **No example**: the gallery and every list example are `sandbox` apps, without commands.
- **Tests run and pass**: effects (9, pure); live_scroll (2), live_shortcuts (2). Every test and example type-checks.
