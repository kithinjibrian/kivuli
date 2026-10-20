# PRP-040 — Switch: a native switch, and kivuli.ui's

Status: draft, for the user's approval (session 81)
Discovery: session 81, 2026-10-20 (the user's next component; the reference is shadcn's current switch). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: a native `switch(on, on_toggle)` on GtkSwitch (disableable from the start, rule 6), `kivuli.testing.flip`, and `ui.switch` in the palette's colours.

## OBJECTIVE

```
import kivuli { switch, field_horizontal … };
import kivuli.ui;

field_horizontal([ui.switch(m.airplane, Msg.Airplane), field_label("Airplane mode")])
ui.switch(m.wifi, Msg.Wifi).disabled(m.airplane)
switch(m.on, Msg.On).name("power")                              // GTK's own look
assert_eq(kivuli.testing.flip(view(m), "power"), Ok(Msg.On(true)));
```

*(you: A)*

## REQUIREMENTS

### lugha-gtk 0.1.36
- `gtk_switch_new`, `gtk_switch_set_active`/`get_active` (`gtkswitch.h`), and the `notify::active` signal through the existing `connect_notify`. Pushed under the standing go.

### The native switch
- `switch<Msg>(on: bool, on_toggle: fun(bool): Msg): View<Msg>`: `View.Switch(bool, Option<fun(bool): Msg>)`, `None` when `.disabled(true)` (rule 6), mapped by `map`.
- Built as a GtkSwitch, set before its handler is connected (so the first state isn't sent); `notify::active` sends `on_toggle(active)` through a `Ref` (connected once, rule 5), nothing while kivuli sets it or when disabled; insensitive when disabled.
- Diffed and patched like a check box without a label: `SetChecked` when `on` changes (no echo), `SetOnToggle` re-points the handler and sets the sensitivity; the widget is kept. A switch has no label: `.name(…)` finds it (`walk` names it "a switch").
- `Live.Switch`, with its arm in `test/live.la`'s `shown()` (memory: a new `Live` variant breaks it).
- **`kivuli.testing.flip(v, name): Result<Msg, Lookup>`**: the message flipping the switch named `name` sends; `NotFound`, `Ambiguous`, or `Silent` when disabled.

### `ui.switch`
- `kivuli.ui`'s `switch<Msg>(on: bool, on_toggle: fun(bool): Msg): View<Msg>`: the native switch with:
  - the track (the switch itself): 32 × 18 (minimums), fully rounded, a 1 px transparent border, the faint shadow, `Input` when off and `Primary` when on (`When(State.Checked, …)`: GTK's switch is `:checked` when on);
  - the thumb (`Part.Knob`, GTK's `slider` inside the switch): 16 × 16, fully rounded, `Background`; in dark, `Foreground` off and `PrimaryForeground` on *(proposed: the reference's dark thumb; tokens give it by scheme: `Background` off and `PrimaryForeground` on in both schemes reads right, so no scheme rule is needed)*;
  - keyboard focus: the `Ring` border and the 3 px halo; disabled: half opacity; its colours easing 150 ms.
- GTK slides the thumb itself.

### Must NOT Do
- No label inside the switch (a `field_horizontal` beside a `field_label`), no sizes, no "on/off" text.

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [ ] lugha-gtk `test/switch.la`: set and read back the state; `notify::active` fires.
- [ ] `test/lib.la` or a new `test/switches.la` (pure): `switch(…)`'s node, `.disabled`, `map`; `test/diff_switch.la` (new): on/off and disabled patches.
- [ ] `test/testing.la`: `flip`'s message and each error.
- [ ] `test/live_switch.la` (new, GTK): a flip reaches the model, an update moves it with no echo, disabled sends nothing and is insensitive, the widget is kept; `test/live.la`'s `shown()` covers it.
- [ ] `test/switches_ui.la` (new, pure): `ui.switch`'s track and thumb values, on, focus, disabled.
- [ ] GTK parses the generated rules; `examples/components.la` gains a Switch section; only it built. How it looks and slides: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.35.0, CLAUDE.md; merged, tagged; pushed only on your go.
