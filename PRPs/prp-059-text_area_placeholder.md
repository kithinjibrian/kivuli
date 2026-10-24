# PRP-059 — A placeholder in a text area

Status: approved and built (2026-10-24)
Request: Rapidkoi's DECISION-055, section 1 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `.placeholder(hint)` on a `text_area` shows `hint`, dimmed, while its text is empty.

## OBJECTIVE
An app gives a text area of several lines the same hint a one-line field takes, with no new API:

```
text_area(m.body, Msg.Body).placeholder("Write something")
```

While the text is empty, "Write something" shows dimmed at the top start, where the first character would go. The first keystroke (or a paste, or a model update) hides it; emptying the text brings it back. Clicking on it places the cursor. Tests find the text area by its placeholder: `kivuli.testing.type_into(view(m), "Write something", "Hi")`.

## CONTEXT
- Read first: `src/render/build.la` (the `TextArea` arm, `editor`), `src/render/styling.la` (the `Input` placeholder arm, the 80 px default height), `src/render/patch.la` (`SetValue`, `SetOnChange`), `src/core/live_tree.la` (`Live.TextArea`, `widget()`), `src/testing.la` (`fields`), `test/live.la` (`shown()`).
- Changes: those files, plus `README.md`, `docs/DESIGN.md`, `CHANGELOG.md` (0.55.0, minor: no API change). New tests: `test/diff_text_area.la`, `test/live_text_area.la`; cases added to `test/testing.la`.
- Unchanged: `src/view/*` (`.placeholder` already stores the hint in `Style` for any node), `src/core/diff.la` (a changed style is already a `SetStyle`).
- GTK, all already in lugha-gtk 0.1.46, checked against `/usr/include/gtk-4.0/gtk/` (`gtkoverlay.h`, `gtklabel.h`, `gtkwidget.h`, `gtkenums.h`): `gtk_overlay_new`, `gtk_overlay_set_child`, `gtk_overlay_add_overlay`, `gtk_label_new`, `gtk_label_set_text`, `gtk_label_set_wrap`, `gtk_label_set_wrap_mode`, `gtk_label_set_xalign`, `gtk_widget_set_halign`/`valign`, `gtk_widget_set_can_target`, `gtk_widget_set_visible`, `gtk_widget_add_css_class`, `gtk_widget_set_overflow` (`GTK_OVERFLOW_HIDDEN` = 1). No new externs.
- Decisions: DECISION-041 (open) is read as blocking work towards a new renderer, not features in today's GTK renderer (the user, 2026-10-24); recorded there. No other open entry applies.

## REQUIREMENTS

### Must Do
- **Every text area** is built as overlay → scrolled window → text view, with a hint label as the overlay's overlay, whether or not it has a placeholder: one live shape, so adding or changing a hint later is a style patch.
- **The hint label**: plain text (never markup); the theme's `dim-label` class; `halign`/`valign` start, so it mirrors under right-to-left; wraps at words (`PANGO_WRAP_WORD_CHAR`, as the text view); doesn't take the pointer (`can_target` false); the overlay clips (`overflow` hidden), so a long hint never grows the area.
- **Shown exactly while the buffer is empty and the hint isn't `""`.** Any text, even one space, hides it. The buffer's one existing `changed` handler also updates the label; no second handler, nothing connected per render.
- **`SetStyle`** on a text area sets the label's text, or hides it for `""`.
- **The node's outer widget is the overlay**: its style classes and the 80 px default height go there (the scrolled window fills it).
- **A disabled text area** still shows its hint.
- **`kivuli.testing`**: `fields` matches a text area by placeholder as well as by name, as for one-line fields, so `type_into` and `is_field_disabled` find it; a one-line field and a text area with the same hint are `Ambiguous`.

### Must NOT Do
- No change to `View`, its variants or `.placeholder`'s signature; no change to one-line fields (they keep `gtk_entry_set_placeholder_text`).
- No state in the model or messages for the hint; no colour, font or pixel value in kivuli's CSS.
- Not inside the scrolled window: wrapping the text view would lose its native scrolling (the cursor kept in view).
- No accessibility work: lugha-gtk has no `gtk_accessible_*` functions. While shown, the hint is read as a plain label; hidden, it's out of the accessibility tree. Announcing it as the field's placeholder belongs to request 3 (Rapidkoi's DECISION-050).
- No edits to `../lugha-gtk`, `../lugha` or `../rapidkoi`.

## DESIGN
- `Live.TextArea(overlay, view, buffer, current, hint)`: keeps the text view, so `editor(scrolled)`'s lookup goes; `widget()` returns the overlay.
- Show/hide is one private function in `build.la` (`show_hint(hint, buffer)`), called by the `changed` handler and by styling. The label's text is its own state: the handler reads it, so a `SetStyle` changing the hint needs no `Ref`.
- No new error or panic beyond the existing ones: a `SetStyle` reaching a text area that has no hint label can't happen by construction.
- `src/render/build.la` is at 479 lines: if the change takes it past 500, stop and propose a split before going on.

## TESTS
Pure, no display:
- [ ] `test/testing.la`: `type_into` finds a text area by placeholder and sends `on_change`; `is_field_disabled` too; a field and a text area with the same hint → `Err(Ambiguous)`; no match → `Err(NotFound)`; disabled → `Err(Silent)` from `type_into`.
- [ ] `test/diff_text_area.la`: a changed hint → one `SetStyle`, no rebuild; adding and removing a hint → `SetStyle`; an unchanged hint → no patch.

GTK, skipped without a display (`test/live_text_area.la`):
- [ ] An empty text area with a hint shows the label, with that text and `dim-label`; one with text doesn't.
- [ ] Setting the buffer's text hides it; clearing it shows it; a model update emptying the text shows it.
- [ ] A changed hint patches the same label's text; `""` hides it.
- [ ] The label doesn't take the pointer; the node's outer widget is the overlay, with the node's classes and the 80 px default height.
- [ ] A disabled text area shows its hint.
- [ ] Existing: `test/live_forms.la`, `test/live_health.la` (echo guard, kept cursor) still pass; `test/live.la`'s `shown()` covers the new shape.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles and its tests pass
- [ ] `lughac test test/`, the examples' tests (one at a time) and `lughac fmt --check src examples test` pass; every example builds
- [ ] README, `docs/DESIGN.md`, CHANGELOG 0.55.0; DECISION-041's reading recorded
- [ ] For the user, by eye: in `examples/controls.la` (its text area given a placeholder), the hint shows dimmed, lined up with where the text starts, goes on the first keystroke and comes back when emptied; clicking it places the cursor; styled text areas in `components.la` and `controls.la` look as before
- [ ] Once merged, the user is told, so Rapidkoi can resolve its DECISION-055

## NOTES FROM THE BUILD

- **Split (the user's approval):** `src/render/build.la` reached 507 lines after `fmt`. The text fields' widgets moved to `src/render/text_fields.la` (`create_input`, `create_text_area`, `text_of`, `show_hint`); `build.la` is 390. Named `text_fields`, not `fields`: importing from `render.fields` binds `fields`, which clashed with test helpers of that name (E0314). Imports run live_tree ← text_fields ← styling ← build ← patch ← live.
- **`show_hint` is in `text_fields.la`**, not `build.la` as DESIGN said: styling needs it, and styling can't import build.
- **Not tested:** the overlay's 80 px default height: lugha-gtk has no `gtk_widget_get_size_request`, and a hidden window isn't laid out. It's the same call as before, on the overlay.
- **Mutation check:** without the hint update in the `changed` handler, `test/live_text_area.la` fails.
- `examples/controls.la`'s text area has `.placeholder("Notes")`, for the check by eye.
