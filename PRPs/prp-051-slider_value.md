# PRP-051 — Hiding a slider's number

Status: draft (session 107)
Discovery: session 107, 2026-10-20 (the reference's slider shows no number). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `.show_value(on)` on any slider; `ui.slider` hides its number by default.

## OBJECTIVE

```
slider(0, 100, m.volume, Msg.Volume).show_value(false)    // native: shown unless turned off
ui.slider(0, 100, m.volume, Msg.Volume)                     // hidden
ui.slider(0, 100, m.volume, Msg.Volume).show_value(true)   // shown again
```

*(you: A)*

## REQUIREMENTS

- **`.show_value(on)`**: a Style field `show_value: bool` (default `true`); on a slider `gtk_scale_set_draw_value(scale, on)`, set every restyle on the same widget (build's fixed `1` goes); on other nodes it does nothing, like `.step` *(proposed)*.
- **`ui.slider`** calls `.show_value(false)` on its native slider; an app's later `.show_value(true)` wins (the last restyle does).
- **File size**: `src/view/views.la` is at 491 lines; the modifier and its doc comment bring it to about 500, the limit. Every modifier must stay in views.la (DECISION-018), so the next modifier after this needs a split, which I'll propose then *(proposed: a heads-up, not a change)*.

### Must NOT Do
- No value shown elsewhere (in a tooltip, at the ends), no formatting of the number.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/sliders_ui.la`: `ui.slider` hides the number; `.show_value(true)` after it shows it.
- [ ] `test/live_choices.la`: a slider draws its number by default; `.show_value(false)` hides it; toggling it on update keeps the widget.
- [ ] `examples/components.la`: its Slider section loses the number; one slider shows it; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.46.0; merged, tagged; pushed only on your go.
