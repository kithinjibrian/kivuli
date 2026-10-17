# PRP-006 — Disabled text fields

Status: approved by the user (session 5); building on branch prp-006-disabled_text_fields
Discovery: session 4, 2026-10-17, from the user's brief (no interview: the brief settles the behaviour). *(you)*: from the brief; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `.disabled(true)` on `input(…)` and `text_area(…)`: the field shows its value greyed out, can't be edited, and sends no messages.

## OBJECTIVE

MEMORY decision 6, "disabled means no message", holds for text fields as it does for buttons. An app can show a field the user can't edit. Rapidkoi needs this to render contract inputs that have no `on_change` (rapidkoi DECISION-016, option A).

```
fun view(m: Model): View<Msg> = column([
    input(m.title, Msg.Title).placeholder("Title").disabled(m.locked),
    text_area(m.notes, Msg.Notes).name("notes").disabled(m.locked),
]);

@test
fun locked_fields_send_nothing() {
    let v = view(Model { locked: true, ..m });
    assert_eq(kivuli.testing.type_into(v, "Title", "x"), Err(Lookup.Silent("text field \"Title\"")));
    assert_eq(kivuli.testing.is_field_disabled(v, "notes"), Ok(true));
}
```

## CONTEXT

- Starting state: `View.Input(string, fun(string): Msg, Option<Msg>)` and `View.TextArea(string, fun(string): Msg)` in `src/views.la` (403 lines); `.disabled` handles `Button` only; `diff.la` (448) emits `SetOnChange`/`SetOnSubmit` for every field; `live_tree.Input(GtkWidget, Ref<fun(string): Msg>, Ref<Option<Msg>>)`, `TextArea(GtkWidget, GtkTextBuffer, Ref<fun(string): Msg>)`; `testing.fields`, `type_into`, `submit`, `is_disabled` (buttons).
- Ending state: those changed; tests in `test/lib.la`, `test/diff.la`, `test/testing.la`, `test/map.la`, and a new `test/live_disabled.la` (`test/live_forms.la` is already over 500 lines); README, docs/DESIGN.md, MEMORY.md decision 6 (its scope widened); kivuli 0.1.1.
- Related existing code: disabled buttons end to end: `Button(label, Option<Msg>)`, `SetMsg` setting both the handler's `Ref` and `gtk_widget_set_sensitive`, `is_disabled`, `Lookup.Silent`.
- GTK functions needed: none new. `gtk_widget_set_sensitive` / `get_sensitive` (lugha-gtk 0.1.11, checked against the header then) are already used by buttons.
- Open decisions that must be resolved first: none. (Rapidkoi's DECISION-016 is resolved by this PRP, in that repo.)

## IMPLEMENTATION REQUIREMENTS

### Representation (rule 6): the brief asks to choose and say why
- **`View.Input(string, Option<(fun(string): Msg, Option<Msg>)>)`**: the field's value and its messages, `None` when disabled. The edit handler and the Enter message travel together, so "disabled but Enter still sends" can't be written down at all; there is no flag to disagree with. *(proposed)*
- **`View.TextArea(string, Option<fun(string): Msg>)`**: `None` when disabled. *(proposed)*
- Why not `Input(string, Option<fun(string): Msg>, Option<Msg>)` (two separate options): it allows `None` edits with `Some` Enter, a state that means nothing; every helper would need a rule for it. Why not a `disabled: bool` style field: that is the separate flag rule 6 rules out.
- The public builders keep their signatures: `input(value, on_change)` and `text_area(value, on_change)` build `Some(…)`. *(you)*

### Must Do
- `.disabled(true)` on an input or text area takes its messages away (`None`), through `Styled` wrappers, as for buttons; `.disabled(false)` changes nothing. *(you)*
- `.on_submit(msg)` on a disabled input changes nothing (there are no messages to add it to), so `.disabled(true).on_submit(m)` and `.on_submit(m).disabled(true)` are the same. *(proposed)*
- On screen: a disabled field is insensitive (`gtk_widget_set_sensitive(false)`): greyed out by the theme, and the user can't type. For a text area, the editor (the text view) is made insensitive, not its scrolled window, so long text can still be scrolled. *(you; proposed: which widget for the text area, to be confirmed on real widgets)*
- It sends no messages: the `changed` and `activate` handlers read `None` and send nothing. *(you)*
- The value still updates while disabled (`SetValue`), keeping the rule that text already showing is never rewritten (risk R5). *(you)*
- Patches: `SetOnChange(path, Option<fun(string): Msg>)` sets the handler's `Ref` and the sensitivity, as `SetMsg` does for buttons; `SetOnSubmit` stays. Turning disabled on and off between renders restyles the same widget: handlers stay connected once per widget (risk R4). *(you; proposed: the patch shape)*
- `View.map` maps the optional messages.
- `kivuli.testing`:
  - `type_into` and `submit` on a disabled field return `Err(Lookup.Silent(…))`, like a disabled button. *(you)*
  - New **`is_field_disabled(v, hint): Result<bool, Lookup>`**, finding a field as `type_into` does (an input by placeholder or name, a text area by name). `is_disabled` stays for buttons: widening it to fields would let a field's placeholder and a button's label clash (`Ambiguous`) in existing tests. *(proposed)*
- MEMORY.md decision 6 now names text fields too.
- Docs: README modifier table; docs/DESIGN.md's Disabled section. *(you)*
- Release: `lugha.toml` 0.1.1, a CHANGELOG `[0.1.1]` entry, tag `v0.1.1` once every check passes; push `main` and the tag only on your go. *(you)*

### Must NOT Do
- No disabling of check boxes, drop-downs or sliders. *(you)*
- No read-only-but-selectable fields. *(you)*
- No change to how buttons behave. *(you)*
- No application flags (rapidkoi DECISION-014). *(you)*
- No change to lughac; no lugha-gtk release needed.

## ERROR HANDLING REQUIREMENTS

- `.disabled` can't fail.
- `type_into` / `submit` / `is_field_disabled` return `NotFound` / `Ambiguous` as today; `Silent` for a disabled field (`submit` also when an enabled input has no `.on_submit`, as today).
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- No new externs, closures or libraries. Handlers stay connected once per widget; a disabled field only stops reading a message.

## TESTS TO WRITE

Pure:
- [ ] `test/lib.la`: `input(…).disabled(true)` and `text_area(…).disabled(true)` are `None`; `disabled(false)` keeps `Some`; through a `Styled` wrapper; `.on_submit` on a disabled input stays `None`, in either order.
- [ ] `test/diff.la`: enabled → disabled gives `SetOnChange(path, None)` and `SetOnSubmit(path, None)`; disabled → enabled gives `Some`; a value change while disabled still gives `SetValue`; disabling isn't a `SetStyle` or a `Replace`.
- [ ] `test/testing.la`: `type_into` and `submit` on a disabled input → `Silent`; `type_into` on a disabled text area → `Silent`; `is_field_disabled` → `Ok(true)`, `Ok(false)`, `NotFound`, `Ambiguous`.
- [ ] `test/map.la`: a mapped enabled field sends mapped messages; a mapped disabled one stays `None`.

Live GTK (skipped without a display), `test/live_disabled.la`:
- [ ] an input disabled from the start is insensitive; text set in it outside a dispatch sends nothing; enabling it (a message) makes it sensitive and typing sends; disabling again; the value updates while disabled; the same widget throughout.
- [ ] a text area: the same, its editor insensitive and its scrolled window not.
- [ ] Enter (`activate`) on a disabled input sends nothing.

## ROLLBACK PLAN
- Branch `prp-006-disabled_text_fields` from `main`; abandon by leaving it unmerged. No lugha-gtk change.

## ACCEPTANCE CRITERIA
- [ ] `.disabled(true)` greys out inputs and text areas and stops their messages; toggling it follows the model on the same widget.
- [ ] `kivuli.testing` reports disabled fields as `Silent` and through `is_field_disabled`.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] Every example builds; fmt clean; public items documented; no file over 500 lines (new tests outside `live_forms.la`).
- [ ] README, docs/DESIGN.md, MEMORY.md, CHANGELOG updated; 0.1.1 tagged; merged into `main`. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built.
- For the user: in any example, `.disabled(true)` on a field: greyed out, no typing.
