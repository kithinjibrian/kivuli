# PRP-021 — States: styles on hover, press, focus, disabled, checked

Status: built (session 23) on branch prp-021-states; merged into main
Discovery: session 23, 2026-10-20. *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `When(state, [...])` in `.style([...])`: style values that apply only while a node is hovered, pressed, focused, disabled or checked.

## OBJECTIVE

Nothing kivuli styles can react to the pointer or keyboard today. `When` adds that through the one door, using GTK's own state pseudo-classes. *(you)*

The target (a test, see TESTS):

```
import kivuli { View, When, State, Background, Outline, Text, Width, Line, Paint, button, column, hex };

fun view(m: Model): View<Msg> = column([
    button("Save", Msg.Save).style([
        Background { color: Paint.Color(hex("#2a5298")), ..Background.theme() },
        Text { color: Paint.Color(hex("#ffffff")), ..Text.theme() },
        When(State.Hover, [Background { color: Paint.Color(hex("#3a62a8")), ..Background.theme() }]),
        When(State.Press, [Background { color: Paint.Color(hex("#1e3c72")), ..Background.theme() }]),
        When(State.Focus, [Outline { width: Width.Px(2), color: Paint.Color(hex("#ffffff")), style: Line.Solid, offset: -4 }]),
        When(State.Disabled, [Text { color: Paint.Faded(0.5), ..Text.theme() }]),
    ]),
])
```

## REQUIREMENTS

### The value *(you: one `When`)*
- `public enum State { Hover, Press, Focus, Disabled, Checked }` → GTK's `:hover`, `:active`, `:focus-visible`, `:disabled`, `:checked`. *(proposed: no `Selected`; nothing in kivuli is selectable)*
- `When(state, items: Styling[]): StateStyle`, a `Styling`; `Sheet` gains `states`: each state's own sheet, merged over the node's (a hovered button keeps its border and changes what its hover sets). A second `When` of the same state merges into the first.
- **Which state wins when two are on at once** *(proposed, replacing "the later wins": CSS can't order rules per node)*: a fixed order, like CSS's own: `Hover` < `Focus` < `Press` < `Checked` < `Disabled`. kivuli writes each state's rules with a higher specificity than the one before (`.c:hover`, `.c.c:focus-visible`, …), so a pressed, hovered button shows its press look and a disabled one its disabled look.
- **Allowed in a state**: fonts, text, background colour and gradient, borders, outlines, padding, margin, minimum sizes. **Refused with a message** *(proposed)*: a background image, an exact or maximum size, a gap (none of them is a plain CSS rule) and a `When` inside a `When`: `When(State.Hover, …): a state can't change a background image, an exact or maximum size, or the gap`, `When(State.Hover, …): states can't nest`.

### CSS *(proposed)*
- Each state rule is its base rule with the state's class prefix and pseudo-class: `kv-hover-bg-3a62a8ff` → `.kv-hover-bg-3a62a8ff:hover { background-color: … }`; `kv-press-…` → `.kv-press-….kv-press-…:active`; and so on. Each node gets its states' classes beside its own.

### Must NOT Do
- No transitions (later), no combined states (`Hover` and `Checked` at once as one rule), no `:selected`, no states on the header bar.

## TESTS (narrow, per the user: only what this changes)
- [x] `test/states.la` (new, pure): `When` merges over the node's values; a second `When` of a state merges into the first; the CSS for each state (prefix, pseudo-class, specificity order); paint gives the classes; each refusal.
- [x] `test/live_type.la`: a state's classes land on the widget and its rules in the stylesheet; GTK parses each selector (C probe).
- [x] Every example builds (`-o examples/<name>`). The full suite is skipped this time (the user's call, session 23).
- [ ] How states look: by hand in `examples/borders.la` (a new "States" section).

## ACCEPTANCE
- [x] The OBJECTIVE compiles and its test passes; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.16.0; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **Narrow runs only**, as the user asked: test/states.la (6), live_type (4), and the files covering `Sheet`, rules and paint (styles, sheets, sheets_border, theming, spacing, sizing); every example built. The full suite wasn't run.
- **A state's key repeats its class** (`.c.c:active`), so `paint` takes the class before the first `.` as well as the first `:`.
- **`When` is a function** returning a `StateStyle` (Lugha allows the capital), so it reads as the interview chose.
