# PRP-033 — kivuli.ui: Field

Status: draft, for the user's approval (session 60)
Discovery: session 60, 2026-10-20 (the fifth component; the reference is ui.shadcn.com/docs/components/base/field). The input group follows (the user's order: Field, then the group; GTK's own field icons later). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `field([...])` and `field_horizontal([...])` with `field_label`, `field_description`, `field_error`, `field_group([...])`, and `Field.invalid()`.

## OBJECTIVE

A label above a control and helper text below it, as nearly every form needs; parts are plain views, as Card's are. *(you: A)*

```
import kivuli.ui { field, field_horizontal, field_label, field_description, field_error, field_group, Field, Input };

field_group([
    field([
        field_label("Username"),
        ui.input(m.user, Msg.User),
        field_description("Choose a unique username for your account."),
    ]),
    field([
        field_label("Password"),
        ui.input(m.pass, Msg.Pass).style([Input.invalid()]),
        field_error("Must be at least 8 characters."),
    ]).style([Field.invalid()]),
    field_horizontal([checkbox("Same as shipping address", m.same, Msg.Same)]),
])
```

## REQUIREMENTS

### The parts *(you: A)*
- `field<Msg>(parts: View<Msg>[]): View<Msg>`: a column, 12 px between parts.
- `field_horizontal<Msg>(parts: View<Msg>[]): View<Msg>`: the same in a row, 12 px apart, parts centred across its height *(proposed: a style can't turn a column into a row, so horizontal is its own function, not `Field.horizontal()` as first sketched)*.
- `field_label<Msg>(text: string): View<Msg>`: `Font.sm()`, `Font.medium()`, and no colour of its own, so it takes the field's (and turns red with `Field.invalid()`).
- `field_description<Msg>(text: string): View<Msg>`: `Font.sm()`, `Token.MutedForeground`, wrapping.
- `field_error<Msg>(text: string): View<Msg>`: `Font.sm()`, `Token.Destructive`, wrapping. Several errors are several `field_error`s.
- `field_group<Msg>(fields: View<Msg>[]): View<Msg>`: a column, 24 px between fields.
- **`Field.invalid()`**: `Text.color(Token.Destructive)` on the field, so its label (which has no colour of its own) turns red; the description keeps its muted colour, the control its own. The control's own invalid look is the app's (`Input.invalid()`), as the reference marks the control separately.

### The example
- `examples/components.la`: a Field section (the OBJECTIVE's group, live: the password turns invalid while shorter than 8), and the login card's labels and fields become `field`s.

### Must NOT Do
- No FieldSet, FieldLegend, FieldSeparator, FieldContent or FieldTitle (long settings forms; later), no disabled field styling (the reference doesn't define it), no responsive orientation.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [ ] `test/fields.la` (new, pure): each part's values; a field's and a group's gaps; `field_horizontal` is a row with centred parts; `Field.invalid()` colours the field, the label has no colour of its own, the description keeps muted; the OBJECTIVE reads as written.
- [ ] `examples/components.la` built, the only one. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.28.0, CLAUDE.md; merged, tagged; pushed only on your go.
