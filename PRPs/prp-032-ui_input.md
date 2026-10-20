# PRP-032 — kivuli.ui: Input

Status: built (sessions 57–58); merged into main in session 59 without running its tests (the user's call)
Discovery: session 56, 2026-10-20 (the fourth component; the reference is ui.shadcn.com/docs/components/base/input, whose page shows no class string: the look is shadcn's published input source). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.input(value, on_change)`, a text field in the palette's colours, with `Input.invalid()`; and `State.FocusWithin`, a state for "focus is somewhere inside", which a GTK text field needs for its focus ring.

## OBJECTIVE

The native field with the reference's look, so everything a native `input` does still works (`.placeholder`, `.on_submit`, `.disabled`, the cursor kept while typing). Field (a label and a description around it) is the next PRP. *(you: A, A)*

```
import kivuli.ui { Input };

ui.input(m.email, Msg.Email).placeholder("m@example.com")
ui.input(m.email, Msg.Email).style([if m.email_bad { Input.invalid() } else { look([]) }])
ui.input(m.email, Msg.Email).disabled(true)
column([ui.input(m.query, Msg.Query).on_submit(Msg.Search)])
```

## REQUIREMENTS

### `State.FocusWithin` *(you: A)*
- A new `State`: CSS `:focus-within`, on while the node or anything inside it has keyboard focus, however it got there (a click or Tab). GTK text fields need it: the typing area inside the field takes focus, not the field, so `State.Focus` (`:focus-visible`) never matches a field.
- Its rank: after `Focus`, before `Press` (Hover < Focus < FocusWithin < Press < Checked < Disabled); the class repetition grows by one for the states after it. Everything a `When` allows, it allows.

### The input
- `kivuli.ui`'s `input<Msg>(value: string, on_change: fun(string): Msg): View<Msg>`: kivuli's native `input` with:
  - `Font.sm()`, `Border.rounded_md()`, a 1 px `Input` border, `Sizing.min_height(36)`, `Spacing.padding_x(12)`, `padding_y(4)`, a transparent background *(proposed: the reference tints it `Input` at 30% in dark only, and kivuli's tokens can't vary an opacity by scheme; transparent in both)*, the faint shadow buttons have, `Text.color(Token.Foreground)`;
  - **focus** (`When(State.FocusWithin, …)`): a `Ring` border, a 3 px halo of `Ring` at 50%, GTK's focus outline off;
  - **disabled**: the same look at half opacity, restated inside `When(State.Disabled, …)` to outrank GTK's own `entry:disabled`;
  - **motion**: `Motion.colors(150)`, `Motion.shadow(150)`.
- The placeholder and selected text keep the GTK theme's colours *(proposed: they're parts inside the field, which kivuli's class rules don't reach; the theme draws a placeholder as the field's text colour, dimmed)*.
- **`Input.invalid()`**: a `Destructive` border; its focus halo `Destructive` at 20%.

### Must NOT Do
- No Field, no input groups (prefixes inside the border), no file input, no text area restyle (later, with Field).

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] `test/states.la`: `State.FocusWithin`'s rule (`:focus-within`, its rank between Focus and Press), and the ranks after it.
- [x] `test/inputs_ui.la` (new, pure): the input's values, its focus-within and disabled states, `Input.invalid()`, an app's values after it winning, still a native input (`.placeholder`, `.on_submit`, `.disabled`).
- [x] GTK parses the generated rules (the C probe); `examples/components.la` gains an Input section (plain, placeholder, invalid, disabled) and its login card uses `ui.input`; only it built. How focus looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.27.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **Disabled restates the background and text too**, not only the border: GTK's `entry:disabled` changes all three.
- **GTK parsed the input's CSS** (71 lines, 19 of them `:focus-within`; 0 errors). Tests type-checked only; only examples/components.la built (an Input section; the login card's fields are `ui.input`, its model now holds them).
- **Found by hand (session 58)**: (1) a text with no style was never restyled, so it kept GTK's centring (PRP-030's default reached only styled text); `build` now places a new label at the start. `test/live_text_align.la` already checks this case: it would have caught it if it had run. (2) The focus ring was a heavy dark band: the palettes' `Ring` came from the older bases (the text colour); the current ones use a mid grey, the other scheme's muted text (zinc: `#a1a1aa` light, `#71717a` dark), now `base`'s rule for all five.
