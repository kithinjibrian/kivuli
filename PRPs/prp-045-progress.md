# PRP-045 — Progress: a native progress bar, and kivuli.ui's

Status: approved (session 99)
Discovery: session 97, 2026-10-20 (the reference's progress: an 8 px round track in `Primary` at 20%, a `Primary` fill). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: a native `progress(value)` on GtkProgressBar (the value out of 100, display only), `Part.Fill`, and `ui.progress(value)` in the palette's colours.

## OBJECTIVE

```
progress(m.uploaded)                          // GTK's look
ui.progress(m.uploaded)                       // a round track, a primary fill
column([text("Uploading…"), ui.progress(66)]).style([Spacing.gap(8)])
```

*(you: A)*

## REQUIREMENTS

### lugha-gtk 0.1.39
- `gtk_progress_bar_new`, `gtk_progress_bar_set_fraction`/`get_fraction` (`gtkprogressbar.h`). Pushed under the standing go.

### The native progress bar
- `progress<Msg>(value: i64): View<Msg>`: `View.Progress(i64)`, the value clamped to 0–100 when shown (no error: a value past the ends is a full or empty bar); `Live.Progress`, built with its fraction set (`value / 100`); diffed with `SetNumber`-like `SetProgress(path, value)` when it changes, on the same widget; mapped and walked as a leaf (it sends nothing); its arm in `test/live.la`'s `shown()`.
- **No easing** *(proposed, found in the design)*: GTK sizes the fill from the fraction itself, not through a CSS property, so a transition has nothing to animate; the fill moves to each value at once, as GTK's own does.

### The look
- **`Part.Fill`**: the progress bar's `progress` node. Its track is GTK's `trough`: the existing `Part.Trough`.
- `kivuli.ui`'s `progress<Msg>(value: i64): View<Msg>`: the track 8 px high (minimums on the track and the fill), fully rounded, `Primary` at 20%; the fill `Primary`, fully rounded; no border or shadow of the theme's.

### Must NOT Do
- No "busy" pulsing mode (an app puts `Motion.pulse()` on a muted bar), no text on the bar, no vertical bar.

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [ ] lugha-gtk: a fraction set and read back.
- [ ] `test/progress.la` (new, pure): `progress`'s node; the diff's patch on a change and none without; map and walk.
- [ ] `test/live_progress.la` (new, GTK): built at its fraction; an update moves it on the same widget; a value past 100 or below 0 shows full or empty; `shown()` covers it.
- [ ] `test/progress_ui.la` (new, pure), `test/parts.la`: the look's values; `Part.Fill`'s selector.
- [ ] GTK parses the generated rules; `examples/components.la` gains a Progress section (driven by the Volume slider); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.40.0, CLAUDE.md; merged, tagged; pushed only on your go.
