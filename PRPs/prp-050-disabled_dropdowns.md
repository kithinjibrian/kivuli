# PRP-050 — Disabled drop-downs

Status: draft (session 107)
Discovery: session 107, 2026-10-20 (the reference's disabled select: the trigger at 50% opacity, not clickable). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `.disabled(on)` on a drop-down (native, `ui.dropdown`, `ui.dropdown_grouped`), as on check boxes, sliders and switches.

## OBJECTIVE

```
dropdown(choices, m.size).disabled(m.locked)
ui.dropdown(choices, m.size, "Size").disabled(m.locked)
assert_eq(kivuli.testing.choose(view(locked), "size", "Large"), Err(Lookup.Silent("size")));
```

*(you: A)*

## REQUIREMENTS

- **Disabled means no message (rule 6)**: a choice's message is an `Option`: `View.Dropdown` holds `(string, Option<Msg>)` choices in its groups (PRP-049's shape); `dropdown` and `dropdown_grouped` keep their signatures (their choices are wrapped in `Some`). `.disabled(true)` clears every choice's message; `.disabled(false)` leaves them.
- On GTK: the drop-down is sensitive while any choice sends, greyed and unclickable while none does; set at build and on every `SetChoices`, on the same widget. Its `notify::selected` handler already sends nothing for a `None`.
- `map` maps the `Some`s; `kivuli.testing.choose` on a disabled drop-down returns `Err(Lookup.Silent(name))`, as `toggle` and `slide` do for theirs.
- **`ui.dropdown`'s look** gains `When(State.Disabled, [Effects.opacity(0.5)])` on its trigger, no hover change while disabled.

### Must NOT Do
- No single disabled choices (B: later, its own PRP), no separate `disabled` flag (rule 6).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/dropdowns.la`: `.disabled(true)` clears every message, `false` keeps them; `map` of a disabled one; the diff's `SetChoices` when only the disabling changes.
- [ ] `test/testing.la`: `choose` on a disabled drop-down returns its error.
- [ ] `test/live_choices.la`: a disabled drop-down isn't sensitive; enabling it on update makes it sensitive on the same widget; it sends nothing while disabled.
- [ ] `test/dropdowns_ui.la`: the disabled look.
- [ ] `examples/components.la`'s Dropdown section gains a disabled one; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.45.0; merged, tagged; pushed only on your go.
