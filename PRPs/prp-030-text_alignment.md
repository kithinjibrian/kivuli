# PRP-030 — Text alignment

Status: built (session 51) on branch prp-030-text-align
Discovery: sessions 48–50, 2026-10-20 (raised from the components screenshot: headings and a card's description centred). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: where text sits inside its label: `Text.start()`, `Text.center()`, `Text.end()`, with start as the default for all text (breaking: GTK centres).

## OBJECTIVE

A label stretched across a column centres its text, GTK's default, and kivuli couldn't change it. After this, text starts at the start, as on the web, and an app moves it with a Text value. *(you: A, B)*

```
column([
    text("Storage").style([Font.h3()]),          // at the start: the new default
    text("12 GB of 64 GB used"),
    text("Centred").style([Text.center()]),
    text("At the end").style([Text.end()]),
    text(m.body).wrap().style([Text.center()]),  // wrapped lines centred too
])
```

## REQUIREMENTS

### The values *(you: A)*
- `Text` gains `align: TextAlign` (`Theme`, `Start`, `Center`, `End`), merged as its other fields; `Text.start()`, `Text.center()`, `Text.end()`.
- **A widget property, not CSS**: GTK's CSS has no `text-align`. On a text node, `gtk_label_set_xalign` (0, 0.5, 1) places the text in the label's width, and `gtk_label_set_justify` (`LEFT`, `CENTER`, `RIGHT`) lines up wrapped lines. Set on every restyle, so a changed value lands on the same widget. `Start` and `End` follow right-to-left as GTK's `xalign` does *(proposed: by hand later; kivuli has no RTL test today)*.
- **No class, no state**: `When(State.Hover, [Text.center()])` stops the app: `When(State.Hover, …): a state can't change where text sits` (it isn't CSS, as a background image isn't).

### The default *(you: B: breaking)*
- Text nobody aligns starts at the start (xalign 0, justify left). Labels kivuli makes inside native buttons and check boxes keep GTK's own placement.
- **Breaking**: centred text now needs `Text.center()`. The examples are updated where a centred look was meant; CHANGELOG lists it under Breaking.

### lugha-gtk 0.1.35
- `gtk_label_set_xalign(label: GtkWidget, xalign: f32)`, `gtk_label_get_xalign(label): f32`, `gtk_label_set_justify(label, jtype: i32)`, `gtk_label_get_justify(label): i32` (`gtklabel.h`; `GtkJustification` 0 left, 1 right, 2 center, 3 fill). Pushed under the standing go.

### Must NOT Do
- No `Fill` justification, no per-line alignment, no alignment for text inside buttons (their content is centred by the button).

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] `test/styles.la` (or a new `test/text_align.la`): the three values and their merge; the state refusal.
- [x] `test/live_text_align.la` (GTK): a plain text's label at xalign 0 and justify left; `Text.center()` 0.5 and center, `Text.end()` 1 and right; a changed value on the same widget; a native button's label unchanged.
- [x] lugha-gtk: set and read back xalign and justify.
- [x] The examples that use what changed type-check; `examples/components.la` built (its headings start now). How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.25.0 (Breaking); merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **lugha-gtk 0.1.35** (`gtk_label_set_xalign`/`get_xalign` with an `f32`, `gtk_label_set_justify`/`get_justify`): its test reads them back; pushed and tagged.
- **`Text.align`** is the field; `TextAlign` is in `styles/values.la`; the state refusal sits beside the background-image one in `sheets.check`.
- **No example needed `Text.center()`**: their headings read well at the start. If one was meant centred, it's one value.
- **Tests not run** (the user's call): text_align and live_text_align type-check; the examples with Text values type-check; only components built.
