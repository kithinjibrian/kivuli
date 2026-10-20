# PRP-036 — kivuli.ui: Checkbox, and styling a widget's parts

Status: built (session 71) on branch prp-036-ui-checkbox
Discovery: session 70, 2026-10-20 (the eighth component, and the first whose inside GTK draws). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Inside(part, [...])` and `Part.Check`, style values for a part inside a widget; `Mark.icon(name, px)`, the icon GTK draws in a part; and `ui.checkbox(label, checked, on_toggle)`.

## OBJECTIVE

A check box in the palette's colours whose tick is always drawn (today a broken theme draws none), still the native check box. The part styling is public, so Slider, Select and an input's placeholder can follow the same way. *(you: A, yes)*

```
import kivuli.ui { Checkbox };

ui.checkbox("Accept terms and conditions", m.ok, Msg.Ok)
ui.checkbox("Notifications", m.on, Msg.On).disabled(true)
field_horizontal([ui.checkbox("Same as shipping address", m.same, Msg.Same)])

// an app's own, the same way:
checkbox("Mine", m.x, Msg.X).style([
    Inside(Part.Check, [Border.all(2, Token.Primary)]),
    When(State.Checked, [Inside(Part.Check, [Background.color(Token.Primary), Mark.icon("object-select-symbolic", 14)])]),
])
```

## REQUIREMENTS

### Parts *(you: A)*
- `public enum Part { Check }` (more as components need them: `Trough`, `Highlight`, `Knob`, `Placeholder`, `Selection`). `Check` is GTK's `check` node, the box of a check box.
- `Inside(part: Part, items: Styling[]): PartStyle`: values applied to that part, merged like a node's (a second `Inside` of a part merges into the first), written as a descendant rule (`.kv-check-bg-… check { … }`; GTK 4.22 parses it: session 70's probe). Its classes go on the widget.
- Inside `When(state, …)`: the state's pseudo-class sits on the widget, the part after it (`.kv-checked-…:checked check`), so a checked box restyles. A part can't hold a `When` or another `Inside` (stops the app with a message, as states can't nest).
- What a part can't take, as a state can't: a background image, an exact or maximum size, a gap.

### The mark *(you: yes)*
- `Mark.icon(name: string, px: i64)`: the icon GTK draws in a part (`-gtk-icon-source: -gtk-icontheme("name"); -gtk-icon-size: Npx`), tinted by the part's text colour for symbolic icons. Checked like `Image.Icon`'s name (no quotes, backslashes, `;`, braces or line breaks); a size below 1 stops the app. Meant for parts; on a whole node it does what GTK does with it there.

### The checkbox *(you: yes)*
- `kivuli.ui`'s `checkbox<Msg>(label: string, checked: bool, on_toggle: fun(bool): Msg): View<Msg>`: the native check box with:
  - its label `Font.sm()`, `Font.medium()`;
  - the box (`Part.Check`): 16 × 16 (minimums), `Radius.All(4)`, a 1 px `Input` border, the faint shadow, 8 px from the label, its colours and shadow easing 150 ms;
  - checked: `Primary` fill and border, a 14 px `object-select-symbolic` in `PrimaryForeground`;
  - keyboard focus: a `Ring` border and the 3 px `Ring`-at-50% halo on the box;
  - disabled: the whole check box at half opacity.
- `.disabled`, the label and `kivuli.testing` work as on the native one.

### Must NOT Do
- No indeterminate state, no radio group, no switch (later components); no other parts yet.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] `test/parts.la` (new, pure): `Inside`'s merge; its rules' selectors (alone, under each state); its classes on the node; `Mark.icon`'s CSS and its checks; a part holding a state or a part refused; the refused values.
- [x] `test/checkboxes_ui.la` (new, pure): the checkbox's values, its checked, focus and disabled looks; still a native check box.
- [x] GTK parses the generated rules (the C probe); `examples/components.la` gains a Checkbox section and the Field section's check box becomes `ui.checkbox`; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.31.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **A native check box can't be disabled** (found in the build): `.disabled` handles buttons, fields and text areas, and `View.Checkbox` holds its toggle function, not an `Option`. Out of this PRP's scope (it changes the view, the diff and the patch): raised with the user. The checkbox's disabled look is in place for when it can be.
- **The focus state kept dropping a rule's whole tail** (for an outline's "step aside"); a part's node would have gone with it. Now only `:not(:focus-visible)` goes (a test covers it).
- **GTK parsed the checkbox's CSS** (56 lines, 0 errors). Tests type-checked only; only components built.
