# PRP-034 — kivuli.ui: Input group

Status: built (session 64) on branch prp-034-ui-input-group
Discovery: session 63, 2026-10-20 (after Field, the user's order; the reference is ui.shadcn.com/docs/components/base/input-group, whose page shows no class strings). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `input_group(start, control, end)`: one frame around a field and what sits beside it, with `group_input`, `group_text`, `group_button`, `group_button_with`, and `InputGroup.invalid()`.

## OBJECTIVE

A search box with its icon, a URL field with `https://` and `.com`, a field with a button inside its border: the frame draws the border and the focus ring, the field inside has none. *(you: A)*

```
import kivuli.ui { input_group, group_input, group_text, group_button, group_button_with, InputGroup };

input_group([icon("system-search-symbolic")], group_input(m.q, Msg.Q).placeholder("Search…"), [])
input_group([group_text("https://")], group_input(m.site, Msg.Site), [group_text(".com")])
input_group([], group_input(m.q, Msg.Q), [group_button("Search", Msg.Go)])
input_group([], group_input(m.pass, Msg.Pass), [group_button_with([icon("view-reveal-symbolic")], Msg.Show)])
input_group([group_text("$")], group_input(m.price, Msg.Price), [group_text("USD")]).style([InputGroup.invalid()])
```

## REQUIREMENTS

### The frame *(you: A)*
- `input_group<Msg>(start: View<Msg>[], control: View<Msg>, end: View<Msg>[]): View<Msg>`: a row of the start add-ons, the control (taking the spare room), the end add-ons, with:
  - the field's look on the row: a 1 px `Input` border, `Border.rounded_md()`, `Sizing.min_height(36)`, a transparent background, the faint shadow; the ring while anything inside has focus (`When(State.FocusWithin, …)`: a `Ring` border, a 3 px `Ring`-at-50% halo); `Motion.colors(150)`, `Motion.shadow(150)`;
  - the add-ons in `MutedForeground`, small text, centred across the row, icons at 16 px unless sized, 8 px apart, 12 px from the frame's edge *(proposed)*;
  - the control's padding on a side with add-ons 8 instead of 12, so text sits close to its prefix *(proposed)*.
- **`InputGroup.invalid()`**: a `Destructive` border, its halo `Destructive` at 20%.

### The parts
- `group_input<Msg>(value: string, on_change: fun(string): Msg): View<Msg>`: a `ui.input` with no border, background, shadow or ring of its own (in every state), so the frame's show. `.placeholder`, `.on_submit`, `.disabled` work.
- `group_text<Msg>(text: string): View<Msg>`: text; the frame makes it muted and small.
- `group_button<Msg>(label: string, msg: Msg): View<Msg>` and `group_button_with<Msg>(content: View<Msg>[], msg: Msg): View<Msg>`: compact ghost buttons, 24 px high, 8 px across (a lone icon: a 24 px square), `Radius.Small` corners *(proposed: the reference's `xs` and `icon-xs`)*. Any `Button` variant can go over them (`.style([Button.outline()])`).

### Must NOT Do
- No header or footer rows (`block-start`, `block-end`) and no text-area group (with a later `ui.textarea`); no spinner or ⌘K parts (an app puts `icon(…).style([Motion.spin()])` or a `badge` in an add-on); no GTK entry icons (later, the user's order).

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] (written, type-checked) `test/input_groups.la` (new, pure): the frame's values and focus-within ring; add-ons muted, centred, icons 16 px; the control's padding beside add-ons (8) and without (12); `group_input` has no frame in any state; `group_text`; the group buttons' size; `InputGroup.invalid()`; the OBJECTIVE reads as written.
- [x] GTK parses the generated rules (116 lines, 0 errors) (the C probe); `examples/components.la` gains an Input group section (the OBJECTIVE's five), the only example built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.29.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **`group_input` has a 34 px minimum**: inside the frame's 1 px border, so the group stays 36 px high.
- **`group_text` carries `Font.sm()` itself**, not the add-on row: a scale is a share of the parent's, so a small row would shrink the buttons in it too.
- **Tests not run** (the user's call): input_groups type-checks; only components built.
