# PRP-027 — Motion: transitions and named animations

Status: approved (session 39)
Discovery: session 38, 2026-10-20 (the shadcn plan, after PRP-026). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: a `Motion` style group: transitions by property group (`Motion.colors(150)`), easing and delay, entrance animations (`fade_in`, `slide_in`) and looping ones (`spin`, `pulse`), with reduced motion respected.

## OBJECTIVE

shadcn's polish is motion: colours easing on hover, a card lifting, dialogs and toasts fading in, spinners and skeletons. GTK 4.22 does all of it in CSS (transitions, `@keyframes`, `prefers-reduced-motion`: session 38's probe), so kivuli draws nothing and runs no timers. *(you: B, A, A, A)*

```
import kivuli { View, Motion, Easing, Edge, When, State, Token, Background, Effects, Shadow, button, column, icon, text };

button("Save", Msg.Save).style([
    Background.color(Token.Primary),
    When(State.Hover, [Background.color(Token.Accent)]),
    Motion.colors(150),
])
tile.style([
    When(State.Hover, [Effects.scale(1.05), Shadow.lg()]),
    Motion.transform(200),
    Motion.shadow(200),
    Motion.easing(Easing.Out),
])
toast(t).style([Motion.fade_in(200), Motion.slide_in(Edge.Bottom, 8, 200)])
icon("process-working").style([Motion.spin()])
column([]).style([Sizing.height(16), Background.color(Token.Muted), Motion.pulse()])   // a skeleton line
```

## REQUIREMENTS

### Transitions *(you: A)*
- `Motion.colors(ms)`: `color`, `background-color`, the four `border-*-color`s, `outline-color` (Tailwind's `transition-colors`); `Motion.opacity(ms)`; `Motion.transform(ms)`; `Motion.shadow(ms)`: `box-shadow`; `Motion.all(ms)`: `all`, opt-in (it animates sizes too).
- `Motion.easing(e)`: `Easing.Ease` (the default, Tailwind's `cubic-bezier(0.4, 0, 0.2, 1)`), `Linear`, `In` (`0.4, 0, 1, 1`), `Out` (`0, 0, 0.2, 1`). `Motion.delay(ms)`.
- They merge as style values do: the groups are one list, and a group set again takes its new duration (`[Motion.colors(150), Motion.transform(200)]` is both; `[Motion.colors(150), Motion.colors(300)]` is 300); easing and delay, the later wins. Inside `When(…)` they apply while the state is on (a slower ease out of hover, as CSS does).
- CSS: one class per transition list, `kv-tr-{hex}` → `transition: color 150ms cubic-bezier(…) 0ms, background-color …;`.

### Named animations *(you: B)*
- **Entrances**, played when the node's widget is first shown *(you: A)*: `Motion.fade_in(ms)` (opacity 0 → 1), `Motion.slide_in(edge, px, ms)` (from `px` beyond `edge` to its place; `Edge.Top`, `Bottom`, `Left`, `Right`). Both together are one animation (shadcn's `fade-in slide-in-from-bottom-2`). A widget the diff keeps doesn't replay; a new widget (a new node, a new key, a rebuilt lazy node, a node of another kind) plays. Easing: `Out`, unless `Motion.easing` says otherwise *(proposed)*.
- **Loops**: `Motion.spin()` (one turn a second, linear, forever), `Motion.pulse()` (opacity 1 → 0.5 → 1 over 2 s, forever: Tailwind's `animate-pulse`).
- An entrance and a loop on one node run together (one `animation` list).
- CSS: one `@keyframes` per distinct entrance (`kv-in-{fade}-{edge}-{px}`) and one each for spin and pulse, added to the stylesheet with the first class that uses them; one class per node's animation list, `kv-an-{hex}`.

### Reduced motion *(you: A)*
- When the system asks for less motion (`gtk-interface-reduced-motion`, GTK 4.22): transitions and entrances are instant; `spin` and `pulse` keep running (they mean "still loading"). One `@media (prefers-reduced-motion: reduce)` rule per transition and animation class.
- **lugha-gtk 0.1.34**: the app's stylesheet (`lgtk_css_provider_new_for_scheme`) also binds `prefers-reduced-motion` to `gtk-interface-reduced-motion` (an app's provider otherwise keeps "no preference", as it did with dark mode, session 36), and `lgtk_set_reduced_motion` / `lgtk_reset_reduced_motion` / `lgtk_css_provider_reduced_motion` for tests. Pushed under the standing go.
- GTK's own `gtk-enable-animations` (off) stops every CSS animation, loops included: GTK's call, left as it is.

### Checks (stop the app with a message) *(proposed wording)*
- `Motion.colors(-5): a duration can't be negative`; `Motion.delay(-1): a delay can't be negative`; `Motion.slide_in(Edge.Bottom, -8, 200): a slide's distance can't be negative`.

### Must NOT Do
- No exit animations, no replay, no custom keyframes, no animating layout unless asked (`all` only); no timers or frame callbacks in kivuli.

## TESTS (narrow; run once the user's faster tests are in)
- [ ] `test/motion.la` (new, pure): each utility's value; merging (groups by name, easing, delay, entrance parts, loops); each CSS: transition lists, easings, delays, keyframes for fade, slide (each edge), fade + slide, spin, pulse; an entrance and a loop together; the reduced-motion rules (transitions and entrances off, loops kept); a transition in `When`; each check's message; the OBJECTIVE reads as written.
- [ ] `test/live_motion.la` (GTK): a node's transition and animation classes land and their rules (keyframes included) join the stylesheet; a kept widget keeps its classes across an update; `lgtk_set_reduced_motion` reaches the app's stylesheet.
- [ ] lugha-gtk `test/settings.la`: reduced motion set, read back, reset; the provider follows it.
- [ ] Every example builds; `examples/motion.la` (hover colours, a lifting tile, a toast list that fades in, a spinner, skeleton lines). How motion feels: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.22.0, CLAUDE.md; merged, tagged; pushed only on your go.
