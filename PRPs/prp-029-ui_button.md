# PRP-029 — kivuli.ui: Button

Status: built (sessions 47–48); merged into main in session 49 without running its tests (the user's call)
Discovery: session 46, 2026-10-20 (the second component, on PRP-028's convention). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.button(label, msg)` and `ui.button_with(content, msg)`, a GTK button in the palette's colours, with six variants and four sizes as `Look`s; and `Token.X.alpha(x)`, a token at an opacity, for hovers and the focus halo.

## OBJECTIVE

The button apps use most, drawn from tokens, beside the native one. Same convention as Badge: variants and sizes go through `.style`, an app's own values after them win, `.disabled(…)` works as on any button. *(you: A, A, A, A)*

```
import kivuli { View, button, icon, text, row };
import kivuli.ui { Button };

ui.button("Save", Msg.Save)                                     // default: primary
ui.button("Delete", Msg.Delete).style([Button.destructive()])
ui.button("Cancel", Msg.Cancel).style([Button.outline(), Button.sm()])
ui.button("Later", Msg.Later).style([Button.secondary()])
ui.button("Edit", Msg.Edit).style([Button.ghost()])
ui.button("Learn more", Msg.Docs).style([Button.link()])
ui.button_with([icon("list-add-symbolic"), text("New")], Msg.New).style([Button.lg()])
ui.button_with([icon("open-menu-symbolic")], Msg.Menu).style([Button.icon(), Button.ghost()])
ui.button("Save", Msg.Save).disabled(m.saving)
button("Native", Msg.A)                                         // GTK's own, unchanged
```

## REQUIREMENTS

### A token at an opacity *(you: A)*
- `Token.alpha(self, x: f64): Paint`: `Paint.Alpha(Token, f64)`, CSS `color-mix(in srgb, var(--kv-primary) 90%, transparent)` (GTK 4.22 parses it: session 34's probe). Everywhere a `Paint` goes (text, backgrounds, borders, outlines, shadows), inside `When` too. Classes `kv-…t-primary-a900`. Checked: `Token.Primary.alpha(1.5): an opacity is between 0 and 1`.

### The button *(you: A: `ui.button`)*
- `kivuli.ui`'s `button<Msg>(label: string, msg: Msg): View<Msg>` and `button_with<Msg>(content: View<Msg>[], msg: Msg): View<Msg>`: kivuli's native buttons with the look below, so clicks, `.disabled`, `.shortcut`, `.tooltip` and tests (`kivuli.testing`) work as on any button. Apps write `ui.button(…)` (or import it under another name); `button_with`'s icons are 16 px unless given a size, 8 px apart.
- **The base look** (the default variant and size): `Font.sm()`, `Font.medium()`, `Border.rounded_md()`, a 1 px transparent border (every variant one size), `Sizing.min_height(36)`, `Spacing.padding_x(16)`, `padding_y(0)`, `Primary` on `PrimaryForeground`, a hairline shadow (0 1 px 2 px, black at 5%), and GTK's own button background and shadow removed.
- **Focus**: a 3 px halo of `Ring` at 50% and a `Ring` border, instead of GTK's focus outline (`When(State.Focus, …)`).
- **Disabled**: the variant's colours at half opacity (`Effects.opacity(0.5)`). **Pressed**: as hovered.
- **Motion**: `Motion.colors(150)` and `Motion.shadow(150)`; instant under reduced motion.
- **GTK's own button rules** for `:hover`, `:active`, `:disabled` and `:checked` outrank a plain class *(proposed, found in the design)*, so each variant restates its colours inside `When(Hover / Press / Disabled, …)`, whose rules outrank GTK's.

### Variants *(you: A)*
- `Button.destructive()`: `Destructive` on `DestructiveForeground`, hover `Destructive` at 90%, focus halo `Destructive` at 20%.
- `Button.outline()`: `Background` with a `Border` border, `Foreground` text; hover `Accent` on `AccentForeground`.
- `Button.secondary()`: `Secondary` on `SecondaryForeground`, hover `Secondary` at 80%.
- `Button.ghost()`: no background or shadow; hover `Accent` on `AccentForeground`.
- `Button.link()`: no background or shadow, `Primary` text; hover underlined.
- The default's hover: `Primary` at 90%.

### Sizes *(you: A)*
- `Button.sm()`: min height 32, padding 12 across, gap 6. `Button.lg()`: min height 40, padding 24 across. `Button.icon()`: a 36 × 36 square (min width and height 36, no padding across). A size changes only size, a variant only colours, so `[Button.outline(), Button.sm()]` is both, in either order.

### Must NOT Do
- No toggle or pressed-state buttons, no loading state (an app adds `icon(…).style([Motion.spin()])`), no button groups; Input is a later PRP.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] (written, type-checked) `test/buttons_ui.la` (new, pure): the base look's values; each variant's colours and its hover, press and disabled states; each size; a variant and a size in both orders; `button_with`'s icon sizes and gap; an app's values after a variant win; `.disabled` still gives no message; the OBJECTIVE reads as written.
- [x] (written, type-checked) `test/palettes.la`: `Token.alpha`'s CSS and classes, inside `When` and in a shadow; the check.
- [x] GTK parses the generated rules (101 lines, 0 errors) (the C probe on a sample); `examples/components.la` gains a Button section (every variant, every size, icons, disabled), the only example built. How it looks and feels: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.24.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **`button_with`'s gap is 8 at every size**: the gap is on the content row, which a size's `Look` (on the button) can't reach; shadcn's `sm` uses 6.
- **The ghost and link variants are `Paint.Color(transparent)`**, not "no background": GTK's own button background would show through `Theme`.
- **State rules outrank GTK's** (checked in the generated CSS: `.kv-hover-…:hover` is two classes' worth against `button:hover`'s class and element).
- **A box inside a box (found by hand, session 48)**: `styling.style_label` gave a button's (and a check box's) own label every class of the button, padding, border, background and shadow included, so styled buttons drew a second box inside and counted their padding twice. The label now takes only its text-decoration classes (`-td-`), the one text value CSS doesn't inherit. `test/live_buttons_ui.la` checks the button keeps its box classes and its label doesn't.
- **An icon alone sat at the start** (found by hand, session 48): `button_with`'s content row filled the button; it's centred now (`.align(Align.Center)`).
- **Tests not run** (the user is fixing slow test runs): buttons_ui and palettes type-check; GTK parsed the generated rules; only examples/components.la built.
