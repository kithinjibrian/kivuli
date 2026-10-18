# PRP-013 — Overlay: layers on top of a view

Status: draft, for the user's approval
Discovery: session 14, 2026-10-18 (DECISION-014; Rapidkoi's DECISION-020, its info panel over the app). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `overlay(base, layers)` draws views on top of a base view; each layer sits where `.place(across, down)` says.

## OBJECTIVE

Rapidkoi shows its info panel as a column beside the app (a placeholder) instead of over it. An overlay puts the panel on top of the page, along the right edge, without the page re-laying out underneath; opening and closing it is just the list of layers changing. The same node serves toasts, badges and centred cards. *(you)*

```
overlay(page(m), if m.panel_open {
    [
        // click outside to close: a full-size layer under the panel (a pattern, not a feature)
        button("", Msg.ClosePanel).button_style(ButtonStyle.Subtle).tooltip("Close the panel"),
        info_panel(m).width(320).place(Align.End, Align.Fill),
    ]
} else {
    []
})
```

## CONTEXT

- Starting state: kivuli 0.7.0. `Align` (Fill, Start, Center, End) with `.align` placing a node across its parent's axis; `Live.Pack` (a header bar's views, patched by position without a box of their own, PRP-007); `src/views.la` 486 lines, `src/patch.la` 447; lugha-gtk 0.1.26.
- Ending state: `View.Overlay`, `Live.Overlay` and `Live.Layers`; `.place` (style fields); tests in `test/lib.la`, `test/diff_overlay.la` (new), `test/testing.la`, `test/map.la`, `test/live_overlay.la` (new); `examples/frame.la`'s info panel; lugha-gtk 0.1.27; kivuli 0.8.0.
- GTK functions needed (`gtkoverlay.h`, `gtkwidget.h`, checked when declared), new in lugha-gtk 0.1.27: `gtk_overlay_new`, `gtk_overlay_set_child` / `get_child`, `gtk_overlay_add_overlay` (on top of those already there), `gtk_overlay_remove_overlay`; for tests, reading the order: `gtk_widget_get_first_child`, `gtk_widget_get_next_sibling`.
- Open decisions that must be resolved first: none (this resolves DECISION-014).

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **`overlay(base: View<Msg>, layers: View<Msg>[])`**: the base fills the overlay; the layers are drawn on top in order, a later one above an earlier one. `[]` shows only the base. *(you)*
- **The base is never rebuilt by layers coming and going**: layers are their own list, diffed by position (`Append`, `Remove`, their own patches); the base is diffed as itself. An overlay that becomes something else is a `Replace`. *(you; proposed: positional layers)*
- **`.place(across: Align, down: Align)`** on a layer: `Fill` covers that whole way, `Start` / `Center` / `End` puts it there at its natural size (or its `.width` / `.height`). Without `.place`, a layer covers the whole base. Ignored on nodes that aren't layers. *(you)*
- **Sizing**: the overlay takes the base's size; layers never make the window bigger (GTK's default, `measure-overlay` off). *(proposed)*
- **Clicks**: a layer takes the clicks in its own area; the base gets the rest, as GTK does. No built-in "click outside to close": an app adds a full-size layer under its panel that sends a message, documented as a pattern in the README. *(you)*
- `View.map`, `walk` (tests find a layer's buttons), lazy resolution and `paint` handle overlays.
- Showcase: **`examples/frame.la`**: an "Info" button in the header opens a panel over the page's right edge (with a full-size layer behind it that closes it), as Rapidkoi's info panel will. *(proposed)*
- Docs: README (widgets, `.place`, the click-outside pattern), docs/DESIGN.md (the GtkOverlay mapping). *(proposed)*
- Release 0.8.0; tag once every check passes; push on your go.

### How it maps (proposed)
- `View.Overlay(base, layers)` ↦ a `GtkOverlay`: `set_child(base)`, `add_overlay(layer)` in order. In the live tree, `Live.Overlay(overlay, base, Live.Layers(overlay, layers))`: paths are 0 the base, 1 the layers (then each layer by index), as a header's parts are counted.
- `.place(across, down)` sets the layer's `halign` and `valign` (style fields `place_across`, `place_down`, default `Fill`); a layer is styled with a layer's own axis, so `.align` and `.expand` don't fight it.
- Patches on the layers: `Append` adds on top; `Remove` removes that layer; replacing a layer in the middle removes it and those above, then adds them back in order (a `GtkOverlay` only adds on top), as a header bar's packed views are repacked.

### Must NOT Do
- No built-in click-outside, no scrim or dimming, no animation, no keyed layers. *(you)*
- No `measure-overlay` / `clip-overlay` options.

## ERROR HANDLING REQUIREMENTS

- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- New externs match `gtkoverlay.h` / `gtkwidget.h`; no new libraries or closures handed to C.

## TESTS TO WRITE

Pure:
- [ ] `test/lib.la`: `overlay` makes its node; `.place` lands in the style; a layer without `.place` is `Fill`, `Fill`.
- [ ] `test/diff_overlay.la`: a layer added (`Append` at `[1]`), removed (`Remove [1, i]`), changed in place; the base changing alone touches only path `[0]`; an overlay that becomes a column is `Replace`; a change of `.place` is `SetStyle`.
- [ ] `test/testing.la`: `click` finds a button in a layer and in the base.
- [ ] `test/map.la`: an overlay maps its base's and layers' messages.

Live GTK (skipped without a display), `test/live_overlay.la`:
- [ ] an overlay is a `GtkOverlay` whose child is the base and whose layers follow it in order (first child, next siblings); a layer's `halign`/`valign` follow `.place`; a button in a layer sends its message.
- [ ] opening a panel (a message adds a layer) and closing it: the base is the same widget throughout; a removed layer is finalized.
- [ ] replacing a middle layer keeps the order of the others.
- [ ] lugha-gtk `test/overlay.la`: each new extern.
- [ ] How it looks on screen (a panel over the page) is checked by hand in `examples/frame.la`.

## ROLLBACK PLAN
- Branch `prp-013-overlay` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.27 is additions only.

## ACCEPTANCE CRITERIA
- [ ] Layers draw over a base where `.place` says; opening and closing them never rebuilds the base.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines (`views.la` ≈ 498 and `patch.la` ≈ 490 after this: a split is proposed first if either would pass 500).
- [ ] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md, DECISIONS (014 resolved) updated; 0.8.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built (benchmark included); lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/frame.la`: Info opens a panel over the page's right edge; clicking outside it closes it.
