# PRP-038 — kivuli.ui: Slider, and disabled sliders

Status: draft, for the user's approval (session 75)
Discovery: session 75, 2026-10-20 (the ninth component; the reference is shadcn's current slider). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Part.Trough`, `Part.Highlight`, `Part.Knob`; `ui.slider(min, max, value, on_change)`; and `.disabled(…)` on a slider, native or `ui.slider` (no message: rule 6, as PRP-037 did for check boxes).

## OBJECTIVE

```
import kivuli.ui;

ui.slider(0, 100, m.volume, Msg.Volume)
ui.slider(0, 100, m.volume, Msg.Volume).disabled(m.muted)
slider(0, 100, m.x, Msg.X).style([Inside(Part.Knob, [Background.color(Token.Primary)])])   // an app's own
```

*(you: B: the slider and disabled sliders in one PRP)*

## REQUIREMENTS

### Parts
- `Part.Trough` (GTK's `trough`, the track), `Part.Highlight` (`highlight`, the filled part), `Part.Knob` (`slider`, the thumb), beside `Part.Check`, with everything `Inside` already does (states on the widget, the part after).

### The slider
- `kivuli.ui`'s `slider<Msg>(min: i64, max: i64, value: i64, on_change: fun(i64): Msg): View<Msg>`: the native slider with:
  - the track: 6 px high (a minimum), fully rounded (`Radius.All(999)`), `Muted`;
  - the filled part: `Primary`, fully rounded;
  - the thumb: 16 × 16, fully rounded, a 1 px `Primary` border on `Background`, a small shadow, centred on the track (a −5 px margin all round: (16 − 6) / 2) *(proposed)*, its shadow easing 150 ms;
  - hover and keyboard focus: a 4 px halo of `Ring` at 50% on the thumb (hover is the whole slider's: a part's own hover isn't reachable through `When`) *(proposed)*;
  - disabled: the whole slider at half opacity.

### Disabled sliders *(you: B)*
- `View.Slider(i64, i64, i64, Option<fun(i64): Msg>)`, `None` when disabled; `slider(…)` gives `Some`; `.disabled(true)` gives `None`; `map` maps it.
- Built insensitive when `None`; the change handler sends only through `Some` (connected once, rule 5); the diff's slider handler patch carries the `Option` and sets the sensitivity; the slider keeps its widget.
- `kivuli.testing`'s slider helper returns `Err(Lookup.Silent("slider \"…\""))` when disabled.

### Must NOT Do
- No range (two thumbs), no vertical styling beyond what the parts give, no ticks or marks.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [ ] `test/parts.la`: the three new parts' selectors (`trough`, `highlight`, `slider`), under a state too.
- [ ] `test/sliders_ui.la` (new, pure): the slider's parts' values, its hover and focus halos, disabled at half opacity, still native.
- [ ] `test/map.la`, `test/diff_slider.la` (new) or the diff tests, `test/testing.la`, `test/live_disabled.la`: as PRP-037's, for sliders.
- [ ] GTK parses the generated rules (the C probe); `examples/components.la` gains a Slider section (plain, disabled, beside a value); only it built. How it looks and drags: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.33.0, CLAUDE.md; merged, tagged; pushed only on your go.
