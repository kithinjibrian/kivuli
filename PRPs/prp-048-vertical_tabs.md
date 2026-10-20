# PRP-048 — kivuli.ui: vertical tabs

Status: draft (session 107)
Discovery: session 107, 2026-10-20 (the reference's vertical tabs: the list a column, each trigger the list's full width with its text at the start; the line variant's line on the trigger's end edge). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.tabs_vertical(triggers, active)` and `ui.tabs_line_vertical(triggers, active)`, beside `tabs` and `tabs_line`.

## OBJECTIVE

```
row([
    ui.tabs_vertical([tab("Account", Msg.Tab(0)), tab("Password", Msg.Tab(1))], m.tab),
    panel(m),
]).style([Spacing.gap(16)])
ui.tabs_line_vertical([tab("Overview", Msg.Tab(0)), tab("Reports", Msg.Tab(1))], m.tab)
```

*(you: A)*

## REQUIREMENTS

- **`tabs_vertical<Msg>(triggers: View<Msg>[], active: i64): View<Msg>`**: `tabs`' look (the muted pill, the active trigger raised, the same trigger look) on a column; each trigger fills the column's width (`Align.Fill`), its content at the start (`Text.start()` on a `tab`; a `tab_with`'s row aligned `Start`) *(proposed)*; the column itself `Align.Start` in its parent, so it's as wide as its widest trigger.
- **`tabs_line_vertical<Msg>(triggers: View<Msg>[], active: i64): View<Msg>`**: `tabs_line`'s look on a column, the 2 px line on each trigger's end edge (`Border.right`) instead of its bottom; every trigger has it, the active one's in `Foreground`, so they're one width; 4 px apart *(proposed)*.
- The horizontal `tabs` and `tabs_line` don't change.

### Must NOT Do
- No orientation argument on the existing functions (B), no keyboard arrow navigation between triggers (GTK's Tab order is it, as for `tabs`).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/tabs_ui.la`: both are columns; the active trigger's look and the others'; the end-edge line (`tabs_line_vertical`), the bottom line absent; triggers fill and start their content; past the end none is active.
- [ ] GTK parses the generated rules; `examples/components.la`'s Tabs section gains both vertical kinds; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.43.0; merged, tagged; pushed only on your go.
