# PRP-035 — kivuli.ui: Separator

Status: built (session 68) on branch prp-035-ui-separator
Discovery: session 67, 2026-10-20 (the seventh component). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.separator()`, kivuli's native separator as a 1 px line in `Border`, and `separator_with(label)`, a line with text in its middle.

## OBJECTIVE

Sections split by a quiet line in the palette's colour, and "Or continue with" between a form and its other options. *(you: A)*

```
import kivuli.ui { separator_with };

column([
    login_fields,
    separator_with("Or continue with"),
    ui.button_with([icon("…"), text("Google")], Msg.Google).style([Button.outline()]),
])
row([text("Blog"), ui.separator(), text("Docs"), ui.separator(), text("Source")])   // vertical lines in a row
```

## REQUIREMENTS

- **`kivuli.ui`'s `separator<Msg>(): View<Msg>`**: the native `separator()` (a line across a column, down a row: GTK orients it by its parent), with `Background.color(Token.Border)` and a 1 px minimum both ways, so it's one pixel thick in either direction *(proposed)*. Apps write `ui.separator()` beside the native one.
- **`separator_with<Msg>(label: string): View<Msg>`**: a row of a line, the label (`Font.sm()`, `MutedForeground`, centred across the row), and a line, 8 px apart; each line takes half the spare room. The lines are 1 px high boxes in `Border`, not GTK separators (a separator in a row runs down). For columns only: in a row it would still show, but sideways text between lines isn't what it's for (a component can't know its parent, so nothing stops it) *(proposed)*.

### Must NOT Do
- No decorative separators (dashes, gradients), no spacing of their own (the parent's gap decides).

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] (written, type-checked) `test/separators.la` (new, pure): `ui.separator()` is the native separator with the `Border` colour and 1 px minimums; `separator_with`'s row, its two growing lines (1 px, `Border`), its muted centred label; the OBJECTIVE reads as written.
- [x] `examples/components.la` gains a Separator section, and the login card a "Or continue with" separator; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.30.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **The login card keeps its footer**: its Login and Google buttons are both in the footer, so an "Or continue with" inside it would have nothing after it; the Separator section shows `separator_with` instead.
- **The test and the module were written in one step** (the test first in the file, but not seen failing first).
- **Tests not run** (the user's call): separators type-checks; only components built.
