# PRP-064 — Writing text to the clipboard

Status: awaiting approval
Request: Rapidkoi's DECISION-047, section 5 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `Cmd.copy_text(text)` puts `text` on the clipboard.

## OBJECTIVE

```
Msg.Share => (m, Cmd.copy_text(link)),
```

Rapidkoi's share button copies an app's link.

## CONTEXT
- GTK 4 (`gdk/gdkdisplay.h`, `gdk/gdkclipboard.h`): `gdk_display_get_clipboard(display)`, `gdk_clipboard_set_text(clipboard, text)`; the display from `gtk_widget_get_display(window)`. lugha-gtk 0.1.52 adds `GdkClipboard`, those two, and `lgtk_clipboard_local_text` (a test read-back: GTK keeps the app's own text in the process).
- `src/render/live.la` is at 489 lines: `perform` gains one arm calling `copy_text` in `windows.la`.
- Decisions: none open applies.

## REQUIREMENTS

### Must Do
- `Cmd.Copy(string)`, built with `Cmd.copy_text(text)`; `Cmd.map` and `Cmd.batch` carry it; carried out after the update is shown, as the other commands.
- README (commands), CHANGELOG 0.60.0.

### Must NOT Do
- No reading the clipboard (not asked: pasting into fields works), no images or other formats, no primary selection.

## TESTS
- [ ] `test/effects.la` (pure): the value; `map` and `batch` keep it.
- [ ] `test/live_clipboard.la` (GTK): an update returning `Cmd.copy_text("…")` leaves that text on the display's clipboard; a second replaces it.
- [ ] lugha-gtk: text set reads back.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; tests pass; the full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] For the user, by eye: an example's button copies text, pasted elsewhere
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve DECISION-047
