# PRP-026 — Palettes and colour tokens

Status: draft, for the user's approval (session 34)
Discovery: session 34, 2026-10-20 (shadcn-like components; this is the first of their PRPs). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: named colour tokens (`Token.Primary`, `Token.Muted`, …) that follow light and dark mode, an app-wide `.palette(…)` with shadcn's five bases, a radius token, and `.scheme(…)` to follow the system or force light or dark.

## OBJECTIVE

shadcn's look is named colours, each with a light and a dark value, and one radius. kivuli gets the same, done by GTK's own CSS (custom properties, `@media (prefers-color-scheme: dark)`, `calc()`: all parsed by GTK 4.22, session 34's probe), so dark mode needs no app code. The `ui` components (a later PRP) use only tokens, so a palette restyles them all. *(you: A, A, A, A, A)*

```
import kivuli { View, Token, Palette, Scheme, Background, Border, Text, Spacing, column, text, hex, sandbox };

fun view(m: Model): View<Msg> = column([
    column([
        text("Storage"),
        text("12 GB of 64 GB used").style([Text.color(Token.MutedForeground)]),
    ]).style([
        Background.color(Token.Card),
        Border.all(1, Token.Border),
        Border.rounded_lg(),
        Spacing.padding(24),
    ]),
])
    .palette(Palette.zinc().with(Token.Primary, hex("#2563eb"), hex("#3b82f6")).radius(10))
    .scheme(m.scheme)        // Scheme.System (the default), Light or Dark
```

## REQUIREMENTS

### Tokens *(you: shadcn's set)*
- `public enum Token { Background, Foreground, Card, CardForeground, Popover, PopoverForeground, Primary, PrimaryForeground, Secondary, SecondaryForeground, Muted, MutedForeground, Accent, AccentForeground, Destructive, DestructiveForeground, Border, Input, Ring }`.
- **A token is a `Paint`**: `Paint.Token(t)`, written `var(--kv-primary)` in CSS. Every utility and full value taking a colour takes a token where it takes a `Paint` (`Border.all(1, Token.Border)` through `Paint.Token`). *(proposed: the colour-taking utilities, `Text.color`, `Background.color`, take a `Paint`-like value: `Text.color(hex(…))` and `Text.color(Token.Muted)` both work, through a small `Colorish` interface implemented by `Color` and `Token`; if lughac can't infer it, `Text.token(t)` / `Background.token(t)` instead)*
- Works everywhere a `Paint` does, inside `When` too.

### Palettes *(you: A)*
- `Palette.zinc()`, `slate()`, `stone()`, `gray()`, `neutral()`: shadcn's base colours (its v3 themes, as hex), each token with a light and a dark value.
- `p.with(token, light: Color, dark: Color)`: one token's two values; `p.radius(px)`: the radius (default 10, shadcn's `0.625rem`).
- **The radius as a value**: `Radius.Small`, `Radius.Medium`, `Radius.Large` = the radius − 4, − 2, and itself (`calc(var(--kv-radius) - 4px)`), with utilities `Border.rounded_sm()`, `rounded_md()`, `rounded_lg()`.
- **Without `.palette(…)`**, tokens still work with zinc's values, so components look right in any app; only `.palette` colours the window. *(proposed)*

### The window *(you: A)*
- With `.palette(p)`, the window's background is `Background` and its text `Foreground`. Native widgets keep GTK's own look.
- `.palette` and `.scheme` are root modifiers, like `.window`; below the root they stop the app with a message (`.palette: only the root view sets the palette`).
- **CSS** *(proposed)*: one rule on the window, `window.kv-palette { --kv-…: light values; background-color: var(--kv-background); color: var(--kv-foreground); }`, and the dark values under `@media (prefers-color-scheme: dark)`. The palette has its own CSS provider (not the growing stylesheet), reloaded only when the palette changes, so a palette in the model can change at run time.

### The scheme *(you: A)*
- `public enum Scheme { System, Light, Dark }`; `.scheme(s)` sets GTK's `gtk-interface-color-scheme` (`GTK_INTERFACE_COLOR_SCHEME_DEFAULT`, `LIGHT`, `DARK`, `gtkenums.h`, since 4.20), so GTK's own widgets and the palette agree. Set when it changes, not every render.
- **lugha-gtk 0.1.32**: the property is set through `g_object_set`, which is variadic (no extern can call it), so a C helper beside the clamp: `void lgtk_set_color_scheme(int scheme)` on `gtk_settings_get_default()`, and `int lgtk_color_scheme(void)` to read it back for tests. Pushed under the standing go.

### Must NOT Do
- No `ui` components (the next PRP after motion), no motion, no coloured themes (`.with` is one line), no `Paint` arithmetic on tokens (`Faded` stays the theme's text colour).

## TESTS (narrow, per the user's preference)
- [ ] `test/palettes.la` (new, pure): a token's CSS (`var(--kv-primary)`); each base has every token, light and dark; `.with` replaces both values; `.radius` and `Radius.Small`/`Medium`/`Large`'s CSS; the window rule and its dark block for a palette; tokens in `When`; `.palette`/`.scheme` below the root refused; the OBJECTIVE reads as written.
- [ ] `test/theming.la`: the paint pass gives token values classes like any `Paint`.
- [ ] `test/live_palette.la` (new, GTK): the window carries `kv-palette` with a palette, not without; a changed palette reloads its provider (not the stylesheet); `.scheme(Scheme.Dark)` sets the setting (read back), `System` restores the default.
- [ ] lugha-gtk `test/settings.la`: set and read the colour scheme.
- [ ] Every example builds; a new `examples/palette.la` (a card, muted text, a theme toggle with `.scheme` in the model). How light and dark look: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.21.0, CLAUDE.md, DECISIONS (the palette's own provider); merged, tagged; pushed only on your go.
