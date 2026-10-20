# PRP-028 — kivuli.ui: the first component, Badge

Status: built (sessions 42, 44); merged into main in session 45 without running its tests (the user's call)
Discovery: sessions 34 and 41, 2026-10-20 (the shadcn plan: a library module, A; one component first, the easiest). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `kivuli.ui`, a module of components drawn from palette tokens, starting with `badge(label)` and its variants as style values (`Badge.secondary()`, `outline()`, `destructive()`), shadcn v4's look.

## OBJECTIVE

The first component sets the convention every later one follows: where components live, how a variant is written, and how an app adjusts one. A variant is a named bundle of style values, merged through the one `.style` door, so an app's own values after it win, field by field. *(you: A, C, A)*

```
import kivuli { View, Token, Palette, Text, Border, row, column, text };
import kivuli.ui { badge, Badge };

row([text("Inbox"), badge("12").align(Align.Center)])
badge("Beta")                                            // default: primary colours
badge("Draft").style([Badge.secondary()])
badge("Draft").style([Badge.outline()])
badge("Failed").style([Badge.destructive()])
badge("Beta").style([Badge.outline(), Text.color(Token.Primary)])   // adjusted: later wins
badge("Beta").style([Border.rounded(999)])                          // the v3 pill, if wanted
```

## REQUIREMENTS

### The module *(proposed)*
- **`kivuli.ui`** (`src/ui.la`), re-exporting each component's file in `src/ui/` (`ui/badges.la`: a file named `badge` would clash with the function `badge` on import, E0314). Apps write `import kivuli.ui { badge, Badge };`. Components use only the public style API (tokens, utilities, `Look`), as an app's own would, so any component can be copied into an app and changed.
- **Architecture rule 1** gains `kivuli.ui` beside the entry and `kivuli.testing` as what apps may import.

### `Look`: a bundle of style values *(proposed)*
- `public struct Look { items: Styling[] }`, a `Styling` that applies its items in order; `look(items: Styling[]): Look`. A variant is a `Look`, and an app's own variant is a function returning one. In `styles.la`, exported from the entry.

### Badge *(you: v4)*
- `badge<Msg>(label: string): View<Msg>`: text, shadcn v4's badge: `Spacing.padding_x(8)`, `padding_y(2)`, `Border.rounded_sm()` (the palette's radius − 4), `Border.all(1, …)` (transparent, so every variant is one size), `Font.xs()`, `Font.medium()`, no wrapping, and `.align(Align.Start)` so it's as wide as its text in a column (`w-fit`); in a row, `.align(Align.Center)` centres it across the row's height *(proposed: kivuli's `align` is one value for the parent's cross axis)*.
- Its default variant: `Background.color(Token.Primary)`, `Text.color(Token.PrimaryForeground)`.
- **`Badge.secondary()`**: `Secondary` / `SecondaryForeground`. **`Badge.outline()`**: no background (`Paint.Theme` can't unset, so `Background.color(transparent)`), `Text.color(Token.Foreground)`, `Border.all(1, Token.Border)`. **`Badge.destructive()`**: `Destructive` / `DestructiveForeground`.
- `Badge` is a namespace for its variants (an empty struct with statics).

### Badge with content *(added in session 44, the user's request: A)*
- `badge_with<Msg>(content: View<Msg>[]): View<Msg>`: the badge's look on a row of `content`, 4 px apart; icons shrink to 12 px unless given a size. Symbolic icons take the text colour.

### Must NOT Do
- No other component (Button is next, reusing this convention); no hover (shadcn's only applies to badges that are links).

## TESTS (narrow; run once the user's faster tests are in)
- [x] (written, type-checked) `test/badges.la` (new, pure): `badge_with`'s row, gap, icon sizes and variants; a badge's sheet (padding, radius `Small`, border width, font, colours `Primary`/`PrimaryForeground`, align `Start`); each variant's colours over it; an app's values after a variant win; `Look` applies its items in order and merges like its items; the OBJECTIVE reads as written.
- [ ] Every example builds; `examples/components.la` (new): every variant in light and dark, with a scheme toggle and a palette switch. How it looks next to shadcn's: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README (a `kivuli.ui` section), DESIGN, CHANGELOG 0.23.0, CLAUDE.md (module tree, rule 1); merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **A component file imports the modules the entry re-exports** (`view.views`, `styles.boxes`, …), not `kivuli`: inside the package the entry is `lib`, and rule 2 keeps package modules off it. The items are the same public ones an app uses.
- **`Look`** lives in `styles.la` beside `applied`, which it calls.
- **Tests not run** (the user is fixing slow test runs): badges type-checks; every example builds; fmt clean. How it looks next to shadcn's: by hand in `examples/components.la`.
