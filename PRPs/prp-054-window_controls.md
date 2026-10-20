# PRP-054 — Styling the window controls

Status: draft (session 122)
Discovery: session 122, 2026-10-21 (rapidkoi's DECISION-035: its header bar is in its frame colour with 48 px icon buttons, but GTK's − □ × keep the desktop's look, and kivuli 0.48 can't reach them; a bar's classes go on `headerbar` only, so the controls stay GTK's: DESIGN, PRP-023). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Part.WindowButton` and `Part.WindowClose`, inside a header bar's style; `kivuli.ui`'s `WindowControls.look()`.

## OBJECTIVE

```
header_bar(start, title, end).style([
    Inside(Part.WindowButton, [
        Text.color(Token.MutedForeground),
        Border.rounded_md(),
        When(State.Hover, [Background.color(Token.Accent), Text.color(Token.AccentForeground)]),
    ]),
    Inside(Part.WindowClose, [
        When(State.Hover, [Background.color(Token.Destructive), Text.color(Token.DestructiveForeground)]),
    ]),
])

header_bar(start, title, end).style([WindowControls.look()])
```

*(you: A, A)*

## REQUIREMENTS

- **GTK keeps the controls** (*you: A*): which buttons, their order and side (the desktop's `gtk-decoration-layout`), what they do, double-click to maximise, snapping. kivuli only styles them.
- **`Part.WindowButton`**: every window control (`windowcontrols > button`); **`Part.WindowClose`** the close one alone (`windowcontrols > button.close`) *(proposed: the selectors, confirmed by a probe of GTK 4.22's header bar first)*. Both meant for a bar's style; a state inside the part puts it on the button (`:hover`, `:active`), as for other parts. The icon is GTK's symbolic one, so it takes the button's text colour; `Sizing` on the part sizes the button.
- **On a bar only**: on a view node they'd match nothing, so `paint`'s check refuses them there (`Part.WindowButton belongs in a header bar's style`) *(proposed)*.
- **`WindowControls.look()`** (`kivuli.ui`, a `Look` for a bar's `.style`): the controls as ghost icon buttons in the palette: no background or border of the theme's, `MutedForeground`, `Border.rounded_md()`, 32 × 32 *(proposed)*; hover and press `Accent` on `AccentForeground` (restated for GTK's own rules, as `Button.ghost()` does); the close one `Destructive` on `DestructiveForeground` on hover; `Motion.colors(150)`.

### Must NOT Do
- No hiding the controls, no minimise or maximise commands (B: later, if wanted), no custom icons for them, nothing for windows without a `header_bar`.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] A C probe first: GTK 4.22's header bar's controls' nodes and classes (`windowcontrols.end > button.close` and the others) under the `headerbar`.
- [ ] `test/parts.la`: the two parts' selectors, a hover inside each.
- [ ] `test/window_controls_ui.la` (new, pure): `WindowControls.look()`'s values; it applies to a bar.
- [ ] `test/palettes.la` or `checks`: the parts refused on a view node, allowed on a bar.
- [ ] `test/live_header.la`: the bar's classes reach the controls' rules (GTK reads back the close button's background colour on a styled bar).
- [ ] GTK parses the generated rules; `examples/browser.la`'s header bar takes `WindowControls.look()`; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.49.0, CLAUDE.md; merged, tagged; pushed only on your go. Then rapidkoi's DECISION-035 can take it (a kivuli bump there, in its own session).
