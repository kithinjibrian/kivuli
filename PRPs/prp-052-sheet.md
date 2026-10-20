# PRP-052 — kivuli.ui: Sheet

Status: approved (session 107, "build all")
Discovery: session 107, 2026-10-20 (the reference's sheet: a panel against one edge of the window over a 50% backdrop, full height and at most 384 px wide on the sides, a border on its inner edge, a large shadow, sliding in; a ✕ at its top end; a header, content and a footer at its bottom). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.sheet(close, edge, parts)`: a `Dialog` shown through the root's `.dialog(…)`, against an edge.

## OBJECTIVE

```
page(m).dialog(if m.editing { Some(edit_sheet()) } else { None })

fun edit_sheet(): Dialog<Msg> = ui.sheet(Msg.Close, Edge.Right, [
    dialog_header("Edit profile", "Make changes to your profile here."),
    fields(m),
    dialog_footer([ui.button("Save changes", Msg.Save)]),
])
```

*(you: A; the edges are motion's `Edge`: `Top`, `Bottom`, `Left`, `Right`, as it has no start or end)*

## REQUIREMENTS

- **Where the card sits moves to the card**: `ui.dialog` places its card `Center, Center` itself; `.dialog`'s layer takes the card as placed (no `centred` in `mapping`). `Dialog`'s fields don't change.
- **`ui.sheet<Msg>(close: Msg, edge: Edge, parts: View<Msg>[]): Dialog<Msg>`**:
  - `Left`/`Right`: the panel fills the height (`.place(Start or End, Fill)`), 384 px wide (min and max, a clamp; 3/4 of the window on a narrow one isn't possible in GTK CSS, so 384 *(proposed)*); `Top`/`Bottom`: the full width (`.place(Fill, Start or End)`), as tall as its content.
  - `Background` on `Foreground`, a 1 px `Border` border on the inner edge only, no radius, `Shadow.lg()`, padding 24, 16 between parts; sliding in from its edge (`Motion.slide_in(edge, 384 or 64, 300)`; instant under reduced motion, as motion is) *(proposed: 64 px for top and bottom, whose height isn't known)*.
  - The ✕ as the dialog's (ghost icon button, "Close", `.shortcut("Escape")`) at the top end; the backdrop and Escape close it.
  - **The footer at the bottom**: on `Left`/`Right`, the parts before the last `dialog_footer` take the room (an `.expand()` spacer before it) *(proposed: a footer is the last part when given)*.
- `dialog_header` and `dialog_footer` are reused, unchanged.

### Must NOT Do
- No dragging to close, no sheet and dialog at once (one `.dialog`), no sizes beyond 384 (an app styles the card with `.style` if it wants).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/dialogs.la`: `ui.dialog`'s card places itself centred and `.dialog`'s layers keep it; `ui.sheet` for each edge: its placement, width or none, the inner border's side, the slide's edge; the ✕ and Escape; the footer pushed to the bottom on the sides.
- [ ] `test/live_dialog.la`: a sheet open over the page: the page inert, the backdrop closes it.
- [ ] GTK parses the generated rules; `examples/components.la` gains a Sheet section (buttons opening it from each edge); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.47.0, CLAUDE.md; merged, tagged; pushed only on your go.
