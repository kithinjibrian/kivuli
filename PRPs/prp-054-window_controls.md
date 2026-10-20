# PRP-054 — Styling the window controls

Status: built (session 124), `Part.WindowIcon` added on the user's yes; merged into main in session 124 (its own tests run and pass)
Discovery: session 122, 2026-10-21 (rapidkoi's DECISION-035: its header bar is in its frame colour with 48 px icon buttons, but GTK's − □ × keep the desktop's look, and kivuli 0.48 can't reach them; a bar's classes go on `headerbar` only, so the controls stay GTK's: DESIGN, PRP-023). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Part.WindowButton`, `Part.WindowClose` and `Part.WindowIcon`, inside a header bar's style; `kivuli.ui`'s `WindowControls.look()`.

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
- **`Part.WindowIcon`** (session 124, the user's yes): the icon inside each control (`windowcontrols > button > image`), where the themes draw their round hover circle (GTK's Default and Yaru: `button:hover > image { background-color: … }`): clearing it there lets the button's own colours show. kivuli's provider outranks the theme's, so one clear covers every state.
- **On a bar only**: on a view node they'd match nothing, so `paint`'s check refuses them there (`Part.WindowButton belongs in a header bar's style`; the same for the other two) *(proposed)*.
- **`WindowControls.look()`** (`kivuli.ui`, a `Look` for a bar's `.style`): the controls as ghost icon buttons in the palette: no background or border of the theme's, `MutedForeground`, `Border.rounded_md()`, 32 × 32 *(proposed)*; the icon's circle cleared (`Part.WindowIcon`: no background, no corners); hover and press `Accent` on `AccentForeground` (restated for GTK's own rules, as `Button.ghost()` does); the close one `Destructive` on `DestructiveForeground` on hover; `Motion.colors(150)`.

### Must NOT Do
- No hiding the controls, no minimise or maximise commands (B: later, if wanted), no custom icons for them, nothing for windows without a `header_bar`.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] A C probe first: GTK 4.22's header bar's controls' nodes and classes (`windowcontrols.end > button.close` and the others) under the `headerbar`.
- [x] `test/parts.la`: the three parts' selectors, a hover inside each.
- [x] `test/window_controls_ui.la` (new, pure): `WindowControls.look()`'s values; it applies to a bar.
- [x] `test/palettes.la` or `checks`: the parts refused on a view node, allowed on a bar.
- [x] `test/live_header.la`: the bar's classes reach the controls' rules (GTK reads back the close button's colour and its icon's cleared background on a styled bar).
- [x] GTK parses the generated rules; `examples/components.la` gains a header bar taking `WindowControls.look()` (not browser.la: it has no palette, so the tokens would be unset); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.49.0, CLAUDE.md; merged, tagged; pushed only on your go. Then rapidkoi's DECISION-035 can take it (a kivuli bump there, in its own session).

## NOTES FROM THE BUILD

- **Probed** (session 123, `scratchpad/probe/wc.c`): `headerbar > windowhandle > box > box.end > windowcontrols.end > button.image-button.{minimize,maximize,close} > image`; a rule on `.kv-… windowcontrols > button.close` reaches the button. The proposed selectors hold.
- **Found: the themes draw the hover circle on the icon, not the button.** GTK's Default and Yaru-dark (this desktop's): `windowcontrols button > image { border-radius: 9999px; min-width/height: 30px }`, `button:hover > image { background-color: … }`, and `button:hover { background: none }`. So a background on `Part.WindowButton` sits behind the theme's round circle, which still shows. Stopped: a third part for the icon (`windowcontrols > button > image`) is beyond the PRP's two; asked the user.
- **The look goes in `examples/components.la`**, on a new header bar: `browser.la` has no palette, so `WindowControls.look()`'s tokens would be unset there.
- **The parts' refusal is in `paint`** (`window_parts`, for every view node's sheet and its states); a bar's sheet goes through `bar_classes`, which takes them.
- **Tests run and pass**: parts, window_controls_ui (19 pure); live_header (the controls' colours read back: minimise, close and the close's icon), live_type, palettes (21). Every test and example type-checks; components built; GTK parsed the rules (43 lines, 0 errors).
