# PRP-061 — Scrolling one way only

Status: approved (2026-10-24: the `.style` design, lugha-gtk 0.1.49) and built
Request: Rapidkoi's DECISION-057, section 3 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: an `Overflow` style group: `scroll(…).style([Overflow.scroll_y()])` scrolls up and down only, `Overflow.scroll_x()` sideways only.

## OBJECTIVE

```
scroll(column(rows)).style([Overflow.scroll_y()])     // up and down only: text wraps; a row that can't shrink widens it
scroll_fit(row(chips)).style([Overflow.scroll_x()])   // sideways only
scroll(big_table)                                     // both, as today
```

## CONTEXT
- Compared with other toolkits (SwiftUI's `ScrollView(.horizontal)`, Compose's `verticalScroll`, iced's `.direction(…)`, CSS/Tailwind `overflow-y`), then chosen as a style value, not a modifier (the user, 2026-10-24): layout already went into styles (`Spacing`, `Sizing`), `Group.mark()` is a non-visual style refused under states, and Rapidkoi's request 12 (clipping, CSS `overflow: hidden`, the same DECISION-057) can later join this group.
- Read first: `src/styles.la` (`Sheet`, `Styling`), `src/styles/groups.la` (the smallest group), `src/styles/checks.la` (refusals), `src/render/styling.la` (`apply_style`), `src/lib.la` (re-exports).
- GTK (`gtk/gtkscrolledwindow.h`): `gtk_scrolled_window_set_policy(sw, h, v)`, `GtkPolicyType` 0 always, 1 automatic, 2 never. lugha-gtk 0.1.49 adds it and `lgtk_scrolled_window_policy` (reading one back; `get_policy` returns through pointers).
- Decisions: none open applies (DECISION-041 doesn't block features in today's renderer).

## REQUIREMENTS

### Must Do
- `src/styles/overflow.la`: `Overflow { x: Scrolling, y: Scrolling }`, `enum Scrolling { Theme, Auto, Never }` (`Theme`: not set, GTK's automatic); `Overflow.theme()`, `Overflow.scroll_y()` (`x: Never, y: Auto`), `Overflow.scroll_x()` (the reverse); `apply` merges each axis, the last value setting it wins.
- `Sheet` gains `overflow: Overflow` (a breaking change for a `Sheet { … }` literal with every field, as with `group` in 0.52).
- `apply_style` on a scroll sets both policies from the sheet every time (defaults included, so a dropped value is undone); a change is an ordinary `SetStyle`. On other nodes it does nothing.
- Refused, with a message, under `When(…)` and inside `Inside(…)` (`Overflow belongs on a node's own style`): a scrollbar policy can't follow a state or a part.
- Re-exported from the entry; README, DESIGN.md, CHANGELOG 0.57.0.

### Must NOT Do
- No clipping (request 12: its own PRP); no change to `scroll`/`scroll_fit` or `View.Scroll`; no CSS rule (a policy isn't CSS).

## DESIGN
- `x`/`y` are across/down, as CSS's `overflow-x`/`overflow-y`. GTK's `NEVER` makes the area as wide (or tall) as its content's minimum: text that wraps wraps, and a row that can't shrink widens it ("the content determines the size", `gtkscrolledwindow.h`). Not cut: Rapidkoi's request expected "wraps or is cut"; cutting is request 12's clip, and the two combine. `EXTERNAL` would cut but still scrolls sideways by touchpad or Shift+wheel (`may_hscroll` in `gtkscrolledwindow.c`), so it isn't one way only.
- No new patch, live node or extern beyond lugha-gtk 0.1.49.

## TESTS
- [ ] `test/overflow.la` (pure): `scroll_y`/`scroll_x`'s values; merging (a later value wins per axis; `Theme` sets nothing); refused under `When` and `Inside`; accepted on a node; a changed overflow is one `SetStyle`, no rebuild.
- [ ] `test/live_scroll.la` (GTK): a scroll styled `scroll_y` has policies never across, automatic down; restyled to `scroll_x`, the reverse on the same widget; the style dropped, automatic both ways.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles and its tests pass
- [ ] The full suite (in batches), the examples' tests and `fmt --check` pass; every example type-checks
- [ ] For the user, by eye: in `examples/tasks.la` (its list `Overflow.scroll_y()`), a very long task title never brings a sideways scrollbar: the list (and window) widens instead
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve this part of DECISION-057
