# PRP-044 — Tooltips in the palette's look

Status: draft, for the user's approval (session 94)
Discovery: session 94, 2026-10-20 (the reference's tooltip: `bg-foreground text-background rounded-md px-3 py-1.5 text-xs`, fading and zooming in, with an arrow). GTK puts a tooltip in its own window, hung off the surface its widget is on (a drop-down probe, session 77, showed `popover > tooltip.background`): for the main window, the tooltip window is the root view's sibling, after it. *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Part.Tooltip`, the tooltip window beside the root (`~ tooltip`), and `kivuli.ui`'s `Tooltip.look()`: every `.tooltip(…)` in the app in the palette's look, set once on the root.

## OBJECTIVE

```
import kivuli.ui { Tooltip };

column(page).palette(Palette.zinc()).style([Tooltip.look()])
button("Save", Msg.Save).tooltip("Save the file")          // unchanged: now in the look
```

*(you: A)*

## REQUIREMENTS

- **`Part.Tooltip`**: `~ tooltip`, the tooltip windows after the node among its siblings. Meant for the root view (whose siblings in the window are the title bar and the tooltip windows); on any other node it matches nothing *(proposed: not refused, as a part can't know where it is)*.
- **`Tooltip.look()`**: `Inside(Part.Tooltip, [Background.color(Token.Foreground), Text.color(Token.Background), Font.xs(), Border.rounded_md(), Spacing.padding_y(6), Spacing.padding_x(12), Border.all(0, …), Shadow.off(), Motion.fade_in(150)])`: the theme's tooltip border and shadow removed, the label inside taking the colour and size by inheritance; fading in, instant under reduced motion *(proposed: the fade without the reference's zoom: a scale would move the text)*.
- GTK still decides when a tooltip shows, where, and for how long; no arrow (GTK's tooltips have none).

### Must NOT Do
- No rich tooltips (views inside), no per-node looks (it's one look for the app), no tooltips inside popovers and menus (those hang off the popover's surface: `.tooltip` on a menu item isn't a thing yet anyway).

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [ ] `test/parts.la`: `Part.Tooltip`'s selector (`.kv-tooltip-… ~ tooltip`).
- [ ] `test/tooltips_ui.la` (new, pure): `Tooltip.look()`'s values; it goes on the root.
- [ ] GTK parses the generated rules; `examples/components.la`'s root takes `Tooltip.look()`, and buttons there gain tooltips; only it built. **That the tooltip window is the root's sibling, and how it looks: by hand** (a C probe can't show a tooltip without a real pointer).

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.39.0, CLAUDE.md; merged, tagged; pushed only on your go.
