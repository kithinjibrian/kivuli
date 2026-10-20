# PRP-041 — kivuli.ui: Tabs

Status: approved (session 85)
Discovery: session 84, 2026-10-20 (the reference is ui.shadcn.com/docs/components/base/tabs: a list of triggers, a panel per trigger, a "line" variant, vertical, a disabled trigger, icons). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.tabs(triggers, active)` and `ui.tabs_line(triggers, active)`, with `tab(label, msg)` and `tab_with(content, msg)`: tabs from the model, built of buttons.

## OBJECTIVE

Which tab is open is part of the model, as everything is (rule 7): a trigger sends its message, the app keeps the tab and shows its panel. *(you: A)*

```
import kivuli.ui { tab, tab_with };

column([
    ui.tabs([tab("Account", Msg.Tab(0)), tab("Password", Msg.Tab(1))], m.tab),
    if m.tab == 0 { account(m) } else { password(m) },
])
ui.tabs_line([tab("Overview", Msg.Tab(0)), tab("Reports", Msg.Tab(1)).disabled(true)], m.tab)
ui.tabs([tab_with([icon("…"), text("Preview")], Msg.Tab(0)), tab_with([icon("…"), text("Code")], Msg.Tab(1))], m.tab)
```

## REQUIREMENTS *(proposed: the triggers as views and the active one by position, not label pairs as first sketched, so `.disabled` and icons work the same way)*

- `tab<Msg>(label: string, msg: Msg): View<Msg>` and `tab_with<Msg>(content: View<Msg>[], msg: Msg): View<Msg>`: a trigger (a native button; `tab_with`'s icons at 16 px, 6 px apart). `.disabled(true)` makes it unclickable (no message, rule 6).
- `ui.tabs<Msg>(triggers: View<Msg>[], active: i64): View<Msg>`: a row of the triggers, the one at `active` (from 0) shown active; an index past the end shows none active.
- **The default look**: the list a `Muted` pill (`Border.rounded_lg()`, 3 px padding, min height 36, as wide as its triggers); a trigger: `Font.sm()`, `Font.medium()`, `MutedForeground`, padding 4 down and 8 across, `Border.rounded_md()`, no background or shadow (GTK's removed), hover `Foreground`; the active trigger `Background` on `Foreground` with a small shadow; keyboard focus the ring halo; disabled at half opacity; colours easing 150 ms.
- **`ui.tabs_line(triggers, active)`**: the line variant: the list transparent with no padding; triggers with no background; the active one a 2 px `Foreground` bottom border instead of the raised pill. A function, not a `Tabs.line()` style value *(proposed)*: a `Look` on the row can't restyle the buttons inside it (DECISION-028, still open: a container's style reaching its parts), and the active trigger's look depends on the variant.

### Must NOT Do
- No vertical tabs (later), no GTK notebook or stack, no panel node (the panel is the app's view).

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [ ] `test/tabs_ui.la` (new, pure): the list's look; a trigger's; the active one's (default and line); `active` past the end; `tab_with`'s icons; a disabled trigger sends nothing and keeps its place; the OBJECTIVE reads as written.
- [ ] GTK parses the generated rules; `examples/components.la` gains a Tabs section (default with panels, line, icons, a disabled trigger); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.36.0, CLAUDE.md; merged, tagged; pushed only on your go.
