# PRP-047 — Dialog: a modal over the page

Status: built (session 105) on branch prp-047-dialog
Discovery: session 104, 2026-10-20 (the reference's dialog: a backdrop of black at 50%, a centred card (`Background`, a border, `rounded-lg`, 24 px padding, a large shadow, up to 512 px wide) with a title, a muted description, content and a footer of buttons, a close ✕ at its top right; Escape or a click on the backdrop closes it; it fades in). One window only (CLAUDE.md anti-pattern 9). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `.dialog(Option<Dialog<Msg>>)` on the root, `ui.dialog(close, parts)`, `dialog_header`, `dialog_footer`; and the native modifiers `.inert(bool)` and `.focusable(bool)` that make it modal.

## OBJECTIVE

```
fun view(m: Model): View<Msg> = page(m).dialog(if m.confirming { Some(confirm()) } else { None });

fun confirm(): Dialog<Msg> = ui.dialog(Msg.Cancel, [
    dialog_header("Delete project?", "This can't be undone. The project and its files go."),
    dialog_footer([
        ui.button("Cancel", Msg.Cancel).style([Button.outline()]),
        ui.button("Delete", Msg.Delete).style([Button.destructive()]),
    ]),
])
```

*(you: A)*

## REQUIREMENTS

### Native: `.inert` and `.focusable`
- **`.inert(on)`**: while on, nothing in the node takes the pointer or keyboard focus (`gtk_widget_set_can_target` and `gtk_widget_set_can_focus` off: GTK applies both to the node's descendants). Set every restyle, on the same widget.
- **`.focusable(on)`**: whether the node itself takes keyboard focus (`gtk_widget_set_focusable`); it still takes clicks. Default on.
- lugha-gtk: `gtk_widget_set_can_target`/`get_can_target`, `gtk_widget_set_can_focus`/`get_can_focus` (0.1.40; `set_focusable` exists). Pushed under the standing go.

### The dialog
- `public struct Dialog<Msg>`: what `ui.dialog` makes (its close message and its card).
- **`.dialog(d: Option<Dialog<Msg>>)`** on the root: the root becomes an overlay over itself with the dialog's layers while `Some`, none while `None`. The overlay is there either way *(proposed: so opening and closing the dialog adds and removes layers and never rebuilds the page: its fields keep their text, its scroll its place)*; the page is `.inert(true)` while a dialog is open. Below the root it stops the app (`.dialog(…) belongs on the root of the view only`).
- **The layers**: the backdrop (black at 50%, the whole window, a click sends the close message: a button with no look, `.focusable(false)`, fading in 150 ms), then the card centred (`.place(Align.Center, Align.Center)`).
- **`ui.dialog<Msg>(close: Msg, parts: View<Msg>[]): Dialog<Msg>`**: the card: a column of `parts`, `Background` on `Foreground`, a 1 px `Border` border, `Border.rounded_lg()`, padding 24, 16 between parts, `Shadow.lg()`, at most 512 px wide (480 at least) *(proposed)*, fading in 150 ms; a close ✕ at its top end (a ghost icon button, `window-close-symbolic`, its tooltip "Close", `.shortcut("Escape")`: Escape closes the dialog from anywhere while it's open).
- **`dialog_header(title, description)`**: the title (`Font.lg()`, semibold) over the description (`Font.sm()`, `MutedForeground`, wrapping), 8 apart; an empty description left out.
- **`dialog_footer(buttons)`**: a row at the card's end, 8 apart.

### Must NOT Do
- No second window (anti-pattern 9), no nested dialogs, no dialog anywhere but the root, no zoom (a scale would move the text); no sheet or drawer (later, from the same pieces).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] `test/dialogs.la` (new, pure): `.dialog(None)` and `.dialog(Some(…))` are both an overlay over the page (the page `.inert` only while open); the backdrop and card layers; the card's look; the ✕'s message and Escape shortcut; header and footer; `.dialog` below the root refused; the diff between closed and open adds layers and keeps the page.
- [x] `test/live_dialog.la` (new, GTK): opening keeps the page's widgets (same widgets, inert), the backdrop and ✕ send the close message, closing makes the page take focus and clicks again.
- [x] lugha-gtk: can-target and can-focus set and read back.
- [x] `test/lib.la` or `test/live_style.la`: `.inert` and `.focusable` land on the widget.
- [x] GTK parses the generated rules; `examples/components.la` gains a Dialog section (a button opening the delete dialog; Escape, the backdrop and Cancel close it); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.42.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **`Dialog` and `.dialog`'s body live in `view/mapping.la`** (views.la is at 491 lines; `views` may import `mapping`); `ui.dialog` builds the card in `ui/dialogs.la`. The backdrop is built as view nodes in `mapping` (it can't import `widgets`, which imports `views`).
- **`.focusable` is set on buttons only**: making a box or label focusable would add it to the Tab order. `.inert` sets can-target and can-focus on every widget each restyle (both default on, so nothing else changes): the GTK tests that cover styling, forms, focus, overlays, menus and button content still pass (16).
- **`.inert` and `.focusable` are covered by `test/live_dialog.la`** (the page's can-target and can-focus, the backdrop's focusable), not a separate live_style test.
- **Tests run and pass**: dialogs (5), live_dialog (1). Every test and example type-checks; only components built; GTK parsed the CSS (96 lines).
