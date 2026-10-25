# PRP-067 — A row that shows what fits

Status: approved and built (2026-10-25: fit_row, not window_size; lugha-gtk 0.1.56, pushed and tagged; patch.la split)
Request: Rapidkoi's DECISION-043, section 9 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `fit_row(children, more)` shows as many children as fit, from the start, and `more` after them when some don't; `.on_hidden(f)` sends how many are hidden.

## OBJECTIVE

```
fit_row(m.instances.map(pill), more_button(m.hidden)).style([Spacing.gap(4)]).on_hidden(Msg.StripHidden)
// StripHidden(n): the last n children don't fit; the ⋯ lists those
```

As the window narrows, pills leave one by one from the end, and the ⋯ appears. As it widens, they come back. The row asks for no more minimum width than its ⋯, so the window can shrink.

## CONTEXT
- Rapidkoi offers two options: the window's size as a subscription, or a row that overflows. This PRP takes the row: the real widths are GTK's, so the choice of what fits stays where they're known. With the window's width, Rapidkoi would have to guess each pill's width from its text and the font. `clip`'s `.on_overflow` (0.58) says how far content sticks out, but not how much room there is to bring a pill back.
- GTK has no such widget; libadwaita's `AdwSqueezer` is the nearest precedent: it hides children in `size_allocate` with `gtk_widget_set_child_visible`. lugha-gtk 0.1.56 adds `src/c/fit.c`, a widget with two children: a horizontal `GtkBox`, which kivuli builds and patches as any row, and `more`.
  - **Measuring:** the minimum is `more`'s; the natural size is the box's with every child showing.
  - **Allocating:** it keeps the box's children from the start while they and `more` fit, using each child's natural width plus the box's spacing. It hides the rest with `child_visible`, and places `more` after the last one kept. When everything fits, `more` is hidden.
  - **Signal:** `hidden-changed` is emitted from an idle (as the clip's is), never during layout.
- The node follows `clip`'s path through `views`, `mapping`, `walk`, `paint`, `diff`, `live_tree`, `paths`, `build`, `patch`, and `test/live.la`'s `shown()`. The box is the node's child at index 0 and `more` at index 1, so a pill added or removed is today's row patch. `views.la` (487), `patch.la` (494) and `diff.la` (470) are close to 500 lines: a split is proposed if one would pass it.

## REQUIREMENTS

### Must Do
- `fit_row(children, more)`; `.on_hidden(f: fun(i64): Msg)` (on a fit row, through its style, ignored elsewhere); `map` carries it.
- The style's gap is the box's spacing; other styles go on the fit row's outer widget.
- `hidden-changed` is connected once per widget (rule 5); its handler reads the message from a `Ref`, re-pointed by the diff (`SetOnHidden`, like `SetOnOverflow`).
- README, DESIGN.md, CHANGELOG 0.63.0.

### Must NOT Do
- No `Sub.window_size`: it isn't needed for this, and it can come in its own PRP when an app needs it.
- No hiding from the middle or the start, no vertical version: Rapidkoi's strip needs neither.
- `more` gets no content of kivuli's own: the app draws its ⋯ from the count.

## TESTS
- [ ] lugha-gtk `test/fit.la`: three 50 px children, spacing 4, `more` 20 px. At 300 px all show and `more` is hidden; at 100 px one shows and `more` sits at x = 54; the minimum is 20; `hidden-changed` comes once per change, with the count.
- [ ] `test/fit_rows.la` (pure): the node, `.on_hidden` set through a style and ignored elsewhere, `map`, and the diff (children patched as a row's, `more` at index 1, the message re-pointed).
- [ ] `test/live_fit.la` (GTK): the widgets built; after a narrow allocation, `on_hidden` sends `StripHidden(2)` once; a pill added through a patch lands in the box.

## ACCEPTANCE
- [ ] lugha-gtk tests pass; kivuli's full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] For the user, by eye: an example with a row of pills and a ⋯ listing the hidden ones. Narrow the window: pills move into ⋯ one by one, and the window keeps shrinking. Widen it: they come back.
- [ ] Once tagged, the user is told, so Rapidkoi can resolve DECISION-043
