# PRP-004 — Dropdown and slider

Status: draft, for the user's approval
Discovery: session 3, 2026-10-17. *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `dropdown(choices, selected)`, a list of labelled choices each sending its own message, and `slider(min, max, value, on_change)`, a whole-number slider.

## OBJECTIVE

An app can offer a fixed set of choices and a number in a range, the two controls settings screens are made of, with the same rules as every kivuli widget: the model holds the state, the widget shows it, user changes arrive as messages, kivuli's own updates don't echo. Tests drive both without GTK.

```
fun view(m: Model): View<Msg> = column([
    dropdown([("Small", Msg.Size(Size.Small)), ("Medium", Msg.Size(Size.Medium)), ("Large", Msg.Size(Size.Large))], size_label(m.size))
        .name("size"),
    slider(0, 100, m.volume, Msg.Volume).name("volume"),
    slider(8, 72, m.font, Msg.Font).step(2).name("font"),
]);

@test
fun picking_and_sliding() {
    assert_eq(kivuli.testing.choose(view(m), "size", "Large"), Ok(Msg.Size(Size.Large)));
    assert_eq(kivuli.testing.slide(view(m), "volume", 42), Ok(Msg.Volume(42)));
}
```

## CONTEXT

- Starting state: `src/views.la` (441 lines), `src/widgets.la`, `src/diff.la`, `src/live_tree.la`, `src/build.la` (432 lines), `src/patch.la`, `src/testing.la`, lugha-gtk 0.1.18.
- Ending state: those changed; `src/build.la` split (see Must Do); tests in `test/lib.la`, `test/diff.la`, `test/testing.la`, `test/map.la`, `test/live_forms.la`; lugha-gtk 0.1.19; a new `examples/settings.la`.
- Related existing code: `checkbox` end to end (a value, a message on change, re-pointed handler, no echo); `text_area` (a widget with its own GTK model object, the buffer); `.name` lookups in `testing.la`.
- Related source files: none.
- Lugha spec sections relied on: nothing new.
- GTK functions needed (all in GTK 4.22), new externs: `GtkStringList` and `GListModel`-taking calls: `gtk_string_list_new` (`None`), `gtk_string_list_append`, `gtk_drop_down_new` (model, `None` expression), `gtk_drop_down_set_model`, `gtk_drop_down_set_selected` / `get_selected` (`u32`; none = `0xffffffff`), `connect_notify` (`g_signal_connect_data` for a `notify::…` handler taking the object and a `GParamSpec`); `gtk_scale_new_with_range`, `gtk_scale_set_digits`, `gtk_scale_set_draw_value`, `gtk_range_set_value` / `get_value` (`f64`), `gtk_range_set_range`, `gtk_range_set_increments`, `gtk_range_set_round_digits`.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **`dropdown(choices: (string, Msg)[], selected: string)`**: shows the labels; picking one sends its message; `selected` is the label shown as picked — one not among the labels shows nothing picked. *(you)* Duplicate labels: the first is the one `selected` picks. *(proposed)*
- **`slider(min: i64, max: i64, value: i64, on_change: fun(i64): Msg)`**, horizontal, and **`.step(n)`** (default 1; `n <= 0` is 1): the handle moves in whole steps from `min`; `value` is shown clamped to the range. *(you)* The current number is shown beside the handle (GTK's drawn value, no decimals). *(proposed)*
- Slider messages: each new step during a drag sends `on_change(value)`; a repeat of the value last sent is not sent again. *(you)*
- Diff: a dropdown gives `SetChoices(path, choices, selected)` every time (its messages are re-pointed, as `SetMsg` is for buttons; GTK's list is rebuilt only when the labels change, its selection only when it differs). A slider gives `SetRange(path, min, max, step)` when those change, `SetNumber(path, value)` when the value changes, and `SetOnSlide(path, on_change)` always. *(proposed: the patch shapes)*
- No echo: setting a dropdown's selection or a slider's value from a patch emits GTK's signal; the existing `patching` flag drops it. *(proposed: as for the others)*
- `View.map` maps both: a dropdown's choice messages, a slider's `on_change`. *(proposed)*
- `.name(id)` finds them in tests. `kivuli.testing.choose(v, name, label)`: the message of that choice (`NotFound` for an unknown name or label, `Ambiguous` for two widgets with that name). `kivuli.testing.slide(v, name, value)`: `on_change` of the value clamped to `min..max` and rounded to the nearest step, as the widget would. *(you)*
- Split `src/build.la` before adding to it: the styling functions (`apply_style`, `style_text`, `style_long_text`, `style_button`, `style_shortcut`, `gtk_align`) move to a new `src/styling.la` (not `style.la`: `style` is a common local name), as their own commit with no behaviour change. *(proposed: file size rule)*
- Showcase: a new `examples/settings.la` — a text preview whose size follows a font slider and whose weight follows a dropdown (Normal / Bold), plus a volume slider showing its number; with its own tests. *(proposed)*

### Must NOT Do
- No decimals in sliders (`f64`) and no vertical sliders: not chosen.
- No searchable or editable dropdowns, no icons in choices.
- No change to existing widgets' behaviour.
- No change to lughac; lugha-gtk additions only (0.1.19).

## ERROR HANDLING REQUIREMENTS

- The builders can't fail. A slider with `min > max` is a programmer error: a panic naming both, when the view is built. *(proposed)*
- `choose` / `slide` return `Lookup` errors as the other helpers do.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- Each new extern matches its header; GTK's "no selection" (`0xffffffff`) is a `u32` constant in kivuli, not a magic `-1`.
- The `notify::selected` and `value-changed` handlers are closures pinned with `data`/`release`, connected once per widget (the existing finalize check covers their release).
- No new libraries.

## TESTS TO WRITE

Pure:
- [ ] `test/lib.la`: both builders make their nodes; `.step` lands in the style (or the node, if that's where it lives).
- [ ] `test/diff.la`: a dropdown always gives `SetChoices`; a slider gives `SetNumber` only when the value changes, `SetRange` only when the range or step does, `SetOnSlide` always; a dropdown that becomes a slider is `Replace`.
- [ ] `test/testing.la`: `choose` by name and label, unknown name, unknown label, two of a name; `slide` in range, clamped below and above, rounded to the step.
- [ ] `test/map.la`: mapped choices and a mapped slider send mapped messages.

Live GTK (skipped without a display), `test/live_forms.la`:
- [ ] dropdown: labels and selection read back; picking (setting the selection outside a dispatch) sends that choice's message; an update changing the labels rebuilds them; an update changing `selected` moves the selection without an echo; an unknown `selected` shows none.
- [ ] slider: range, step and value read back; a user change sends the rounded value once (a repeat isn't resent); an update moves the handle without an echo; `.step` rounding.
- [ ] lugha-gtk `test/choices.la`: string list, drop-down selection and `notify::selected`; scale range, value, `value-changed`.
- [ ] `examples/settings.la`'s own tests.

## ROLLBACK PLAN
- Branch `prp-004-dropdown_and_slider` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.19 is additions only.

## ACCEPTANCE CRITERIA
- [ ] `dropdown` and `slider` work in an app, send their messages, and follow the model without echoes.
- [ ] `choose` and `slide` work without GTK.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] Every example builds; fmt clean; public items documented; no file over 500 lines.
- [ ] CHANGELOG.md and README.md updated. Merged into `main`.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/settings.la`: drag the font slider and watch the preview grow; pick Bold; the volume shows its number.
