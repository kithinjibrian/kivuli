# PRP-062 — A clip that says how far its content overflows

Status: approved (2026-10-24: a node, `clip`, not an `Overflow` style; lugha-gtk 0.1.50)
Request: Rapidkoi's DECISION-057, section 12 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `clip(child)` cuts off what doesn't fit, never pushing its parent bigger; `.on_overflow(f)` sends how far the child sticks out.

## OBJECTIVE

```
clip(app_view).on_overflow(Msg.Overflowed)   // Overflowed(Overflowing): across and down, in pixels, 0 when it fits
scroll(clip(list)).style([Overflow.scroll_y()])   // scrolls down; what's too wide is cut
```

## CONTEXT
- GTK has no such widget. A scrolled window with `GTK_POLICY_EXTERNAL` (Rapidkoi's suggestion) still scrolls by touchpad or wheel (`may_hscroll`, `gtkscrolledwindow.c`); GTK's `overflow: hidden` clips drawing but the child's minimum still pushes the window. lugha-gtk 0.1.50's clip (`src/c/clip.c`): no minimum size; the child gets at least its minimum, at the top start, the rest cut off; `lgtk_clip_overflow` across and down; `overflow-changed` from an idle when it changes, never during layout.
- A node, not a style (the user, 2026-10-24): the cut and its message stay on one node, and styles stay free of messages.
- Two numbers, not four sides: the child sits at the top start, so it can only stick out at the end side and the bottom. Rapidkoi's `Overflow` name is taken by PRP-061's style: `Overflowing { across, down }`.
- Read first: how `scroll` flows through `views`, `mapping`, `widgets`, `walk`, `lazies`, `paint`, `diff`, `live_tree`, `paths`, `build`, `patch`; `test/live.la`'s `shown()`.
- `views.la` was split first (`Bar` to `mapping.la`, approved); `diff.la` (492) and `patch.la` (480) take a few lines each: a split is proposed if one passes 500.

## REQUIREMENTS

### Must Do
- `View.Clip(View<Msg>, Option<fun(Overflowing): Msg>)`; `clip(child)`; `.on_overflow(f)` (on a clip, through its style; ignored elsewhere); `map` carries it.
- The diff: a clip's child diffed at index 0; its message re-pointed every time (`SetOnOverflow`), as `SetOnChange`.
- `overflow-changed` connected once per widget (rule 5); its handler reads the message from a `Ref`; no message while there's none.
- `Overflowing` re-exported from the entry; README, DESIGN.md, CHANGELOG 0.58.0.

### Must NOT Do
- No scrolling, no development band (Rapidkoi draws it), no default `.expand()` (as `scroll_fit`: `.expand()` when it should fill).

## TESTS
- [ ] `test/clips.la` (pure): `clip`'s node; `.on_overflow` sets the message, through a style, ignored on other nodes; `map`; the diff (child diffed inside, message re-pointed, no rebuild).
- [ ] `test/live_clip.la` (GTK): a clip's widget asks for no minimum and holds the child's widget; after an allocation smaller than the child, `overflow-changed` sends `Overflowed({across, down})` once; a changed message is the one sent; `None` sends nothing.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles and its tests pass; the full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] For the user, by eye: a window with a clipped wide row shrinks below the row's width, the row cut off, and the overflow shown
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve DECISION-057
