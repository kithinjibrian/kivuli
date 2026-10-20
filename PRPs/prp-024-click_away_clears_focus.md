# PRP-024 — Clicking away clears keyboard focus

Status: built (session 29) on branch prp-024-click_away; merged into main
Discovery: session 29, 2026-10-20 (the user's `examples/search_bar.la`). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: in every kivuli app, a click on anything that doesn't take focus itself clears the window's keyboard focus, as a web page blurs a field.

## OBJECTIVE

In GTK a window always keeps a focused widget, and a click only moves focus onto something that can take it, so clicking beside a field (empty space, padding, text, a picture) leaves the field focused. `examples/search_bar.la` shows it: its one field keeps focus whatever you click. After this, clicking away leaves the field, with nothing to write. *(you: always on)*

```
fun view(m: Model): View<Msg> = column([
    input(m.address, Msg.Draft).placeholder("Enter Address").autofocus(),
]).style([pad(24)])
// click beside the field: it loses focus
```

## REQUIREMENTS

- **lugha-gtk 0.1.31**: `gtk_window_set_focus(window, Option<GtkWidget>)` (`gtkwindow.h`: `void gtk_window_set_focus (GtkWindow *window, GtkWidget *focus)`, `NULL` clears it). Pushed under the standing go.
- **kivuli**: when the window opens, a click gesture on the window (primary button, the bubble phase, so it sees only clicks no widget under the pointer took). On a press, it clears the window's focus. *(proposed)*
- **What takes its own clicks keeps working as now**: buttons, fields, text areas, check boxes, drop-downs, sliders, selectable text, context menus (their right-click gesture is on their own node). Tab and `Cmd.focus` are untouched; `.autofocus` still focuses once.
- No new public API.

## TESTS (narrow, per the user's preference)
- [x] lugha-gtk `test/focus.la` (or the existing focus test): `gtk_window_set_focus` with a widget focuses it, with `None` clears it.
- [x] `test/live_forms.la` is over 500 lines, so a new `test/live_focus.la`: a focused field loses focus when the window's gesture is pressed (`lgtk_emit_pressed`) at a spot no widget takes; a click on a button still reaches the button; `Cmd.focus` still focuses afterwards.
- [ ] Every example builds; how a real click feels: by hand in `examples/search_bar.la`.

## ACCEPTANCE
- [x] fmt clean; no file over 500 lines; README (a line on focus), DESIGN, CHANGELOG 0.19.0; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **lugha-gtk 0.1.31**: `gtk_window_set_focus` with `test/focus.la` (set, then cleared); pushed under the standing go (its README line in a follow-up commit).
- **The gesture lives on `Running`** (`blur`), so the live test presses it with `lgtk_emit_pressed`; mutation-checked (without the clearing, the test fails). A press can't be routed through real widgets in a test, so "a button still gets its click" rests on GTK's bubble phase and is the hand check.
- **Narrow runs** (the user's preference): live_focus (new), live_forms, live_menu, live; every example built.
