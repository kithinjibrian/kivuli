# PRP-007 — App look: typed styles, dark, maximized, header bar

Status: approved by the user (session 6); building on branch prp-007-app_look
Discovery: sessions 5–6, 2026-10-17, from the user's brief and a talk-through of styling options. *(you)*: chosen by you; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: An app gives its own frame colours as typed styles (`.style([…])`, a `Theme`), opens maximized, and puts its own widgets in the title bar (`.header(…)`). (`.dark()` was deferred during the build: DECISION-017.)

## OBJECTIVE

Rapidkoi (a browser-like app on kivuli, rapidkoi DECISION-020 items 1–3) can paint its bar, rail and home in its own colours, look dark, open maximized, and have its address box in the title bar, with every value checked and nothing reaching into GTK's or kivuli's widget internals. kivuli itself still ships no colours: they are the app's, in one theme. *(you)*

```
enum Look { Bar, Rail, Home, Selected }

fun surface(l: Look): Surface = match l {
    Bar => Surface { background: Some(hex("#242424")), ..Surface.theme() },
    Rail => Surface { background: Some(hex("#242424")), ..Surface.theme() },
    Home => Surface { background: Some(hex("#161616")), ..Surface.theme() },
    Selected => Surface { background: Some(hex("#3a3a3a")), radius: Some(6), ..Surface.theme() },
};

fun view(m: Model): View<Msg> = row([
    rail(m).style([Look.Rail]),
    home(m).style([Look.Home]).expand(),
])
    .window("Rapidkoi", 1200, 800)
    .maximized()
    .theme(theme([Look.Bar, Look.Rail, Look.Home, Look.Selected], surface).dark())
    .header([button("", Msg.Back).icon("go-previous-symbolic").tooltip("Back")], address(m), [menu(m)]);

// a tab: the rail's look, the selected one's on top (later looks win)
tab.style([Look.Rail, Look.Selected])
```

## DECISIONS ALREADY MADE (sessions 5–6)

- **Typed styles, StyleX-like, not plain CSS and not per-widget colours.** An app names its looks in its own enum and gives each a typed `Surface`; nodes take `.style([…])`; kivuli merges the list (later wins, field by field) and generates the CSS. Typos are compile errors, colours are checked, apps can't write selectors into kivuli's or GTK's internals. Replaces the brief's stylesheet string and `.class(name)`. *(you)*
- **`.style` for these; the button variant becomes `.button_style(ButtonStyle.…)`**: a breaking rename, so this is **0.2.0**. *(you)*
- **Window-wide settings on the root view**, next to `.window(…)`: `.theme(…)`, `.maximized()`, `.header(…)`. Breaking `App` was allowed, but the header has to be in the view anyway (it holds live widgets), window settings stay in one place, and a theme in the view makes runtime switching a small later step. *(you)*

## CONTEXT

- Starting state: `src/views.la` (403 lines: `.style(ButtonStyle)`, `.window`), `src/looks.la` (Style), `src/live.la` (mount, window title/size), `src/styling.la`, `src/build.la`, `src/patch.la`, `src/diff.la` (448), `src/walk.la`; kivuli 0.1.1 (local tag); lugha-gtk 0.1.22.
- Ending state: those changed; a new pure `src/theming.la` (Color, Surface, Border, Theme, `hex`, `rgb`, `theme`, the CSS generator, the classes for a merged surface); tests in `test/theming.la` (new), `test/lib.la`, `test/diff.la`, `test/testing.la`, `test/map.la`, `test/live_look.la` (new); every example using `.style(ButtonStyle…)` updated; lugha-gtk 0.1.23; kivuli 0.2.0.
- Related existing code: `.window` (a root-only style field the runtime reads), `.shortcut` (bad input panics with a message, checked in pure code), PRP-005's button content and theme classes, `Live.Scroll` (a node whose child is set into a container, for the header's slots).
- GTK functions needed, checked against the 4.22 headers (and the GIR for deprecations), new in lugha-gtk 0.1.23:
  - styles: `GtkCssProvider`, `GdkDisplay`; `gtk_css_provider_new`, `gtk_css_provider_load_from_string` (4.12; `load_from_data` is deprecated), `gtk_css_provider_to_string` (tests; allocated, freed), `gtk_style_context_add_provider_for_display` (in `gtkstyleprovider.h`, not deprecated; priority `GTK_STYLE_PROVIDER_PRIORITY_APPLICATION` = 600), `gtk_widget_get_display`;
  - dark: the `GtkSettings` property `gtk-interface-color-scheme` = `GTK_INTERFACE_COLOR_SCHEME_DARK` (2), new in 4.20. `gtk-application-prefer-dark-theme` (the brief's name) is **deprecated since 4.20** (GIR: "Use GtkCssProvider properties instead"). Setting a property is `g_object_set`, variadic: a C helper `lgtk_prefer_dark(i32)` in lugha-gtk `src/c/`;
  - classes: a C helper `lgtk_remove_css_classes_with_prefix(widget, prefix)` over `gtk_widget_get_css_classes` (a `char **` to free with `g_strfreev`), so kivuli removes only its own classes;
  - tests: a C helper reading `gtk_widget_get_color` (a `GdkRGBA` out-parameter) as one lightness number;
  - window: `gtk_window_maximize`, `gtk_window_is_maximized` (tests), `gtk_window_set_titlebar` / `get_titlebar`;
  - header: `gtk_header_bar_new`, `set_title_widget` / `get_title_widget`, `pack_start`, `pack_end`, `remove`, `set_show_title_buttons` / `get_show_title_buttons`.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Typed styles
- **`Color`**: `hex("#rrggbb")` or `hex("#rrggbbaa")` (also `#rgb`), and `rgb(r, g, b)` (0–255). A string `hex` can't read panics, naming it, when the theme is built: the app stops at startup, as bad shortcuts do. *(proposed)*
- **`Surface`**, what one look sets; every field optional, `None` leaves the theme's: `background: Option<Color>`, `text: Option<Color>` (text and icon colour), `border: Option<Border>` (`Border { width: i64, color: Color }`, solid), `radius: Option<i64>` (corner radius in pixels). `Surface.theme()` is all `None`. Nothing else (fonts stay the theme's; room and size stay in `.padding`, `.width` …). *(proposed: the field list, enough for Rapidkoi's frame; more fields are later PRPs)*
- **`theme(looks: L[], surface: fun(L): Surface): Theme`**, generic over the app's look enum; `.dark()` on a `Theme` asks for GTK's dark variant. A look's name is its variant's printed name (`Bar`); a look with a payload, or one listed twice, panics when the theme is built (class names are then always letters, digits and `_`, the brief's name check). *(proposed)*
- **`.style(looks: L[])`** on any node: the looks merge left to right, a later look's field winning; `[]` is no look. A style field (`SetStyle` when it changes, as for every modifier). A look used in a view but missing from the theme, or a view with `.style` and no theme, panics naming the look, when the view is shown. *(proposed)*
- **The CSS**: kivuli generates atomic rules, one class per property and value, from every value in the theme (`.kv-bg-242424ff { background-color: …; background-image: none; }`, `.kv-fg-…`, `.kv-border-…`, `.kv-radius-…`), and gives a node the classes of its merged surface. Merging happens in kivuli's code, so the result never depends on GTK's cascade order, and every class a view can need exists from the start. `background` also clears the theme's `background-image`, so it shows on themed widgets (buttons). *(proposed)*
- Loaded once, when the window opens: one `GtkCssProvider` for the window's display at priority 600. A different `.theme(…)` in a later view is ignored (documented); runtime switching is a later PRP. *(you)*
- On a widget, kivuli removes only the classes it added (prefix `kv-`); the theme's and its own button classes stay. *(proposed)*

### Dark, maximized
- **Deferred during the build (DECISION-017, the user's choice):** neither GTK setting switched GTK's own theme to dark in tests here (see the notes). The text below is kept for the record.
- ~~**`.dark()`** on the theme:~~ GTK's dark variant for the whole app, set when the window opens, through `gtk-interface-color-scheme` (above). To confirm on real widgets that it switches GTK's own theme (a label's text turns light); if it doesn't, the build falls back to the deprecated `gtk-application-prefer-dark-theme`, and the notes say why. *(proposed)*
- **`.maximized()`** on the root: the window opens maximized (`gtk_window_maximize` before it's shown); `.window`'s width and height stay its size when unmaximized. Read when the window opens. *(you)*

### Header bar
- **`.header(start: View<Msg>[], title: View<Msg>, end: View<Msg>[])`** on the root of `view`: a `GtkHeaderBar` set as the window's titlebar, with GTK's window controls (`show_title_buttons`, drawn by GTK in the system's style). Each start and end view is packed into the bar itself, so the theme's header spacing applies; the title view is its title widget. *(you; proposed: the signature)*
- The title view may fill the row: `.expand()` on it lets it take the bar's spare width (for an address box). *(you; how GTK's header bar shares its width with an expanding title widget is to be confirmed on real widgets)*
- A new node, `View.Framed(body, start, title, end)`, built by `.header`; diffed and patched like the rest (its parts are its children: the body, the title, then the start and end lists by position); messages from header widgets reach `update` as usual; `kivuli.testing` finds header buttons and fields like any others; `View.map` maps them. A `.header` anywhere but the root (under `Styled` wrappers) panics with a message. *(you; proposed: the node and paths)*
- Without `.header`, windows look exactly as today. *(you)*

### Renames and docs
- `.style(ButtonStyle.X)` becomes `.button_style(ButtonStyle.X)`; every example and the README updated; the CHANGELOG says how to migrate. *(you)*
- docs/DESIGN.md: kivuli ships no colours; an app's colours live in its one theme, as typed looks; how a look becomes CSS; dark; the header. *(you)*
- README: `.style`, `.button_style`, `.theme`, `.maximized`, `.header`, `Surface`, `hex`.
- DECISIONS.md: the queued features (file dialog, overlay, context menus, wrapping grid) as deferred decisions. *(you)*

### Must NOT Do
- No per-node inline styles or colour modifiers (colours come only from the theme's looks). *(you)*
- No true full screen (F11). *(you)*
- No changing the theme, dark or the header's presence while running. *(you)*
- No custom-drawn window controls: GTK draws them. *(you)*
- No raw CSS from apps, no selectors, no states (hover, pressed) yet. *(you; states are a later PRP)*
- No change to lughac.

## ERROR HANDLING REQUIREMENTS

- Programmer errors panic with a message naming the problem, as `.shortcut` does: a colour `hex` can't read, a look with a payload or listed twice, a look missing from the theme, `.style` without a theme, `.header` below the root.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- New externs match their headers; the three C helpers (`g_object_set`, `gtk_widget_get_css_classes` + `g_strfreev`, `gtk_widget_get_color`) are in lugha-gtk `src/c/`, each freeing what it allocates.
- The CSS is generated by kivuli from checked values only (hex digits, whole numbers): no app string reaches the stylesheet unchecked, so an app can't inject rules.
- No new libraries.

## TESTS TO WRITE

Pure (`test/theming.la` new, and the existing files):
- [ ] `hex` reads `#rgb`, `#rrggbb`, `#rrggbbaa`; bad strings are errors (checked through the `Result` the panic wraps, as keys are); `rgb`.
- [ ] `theme(…)`: names; a payload look and a duplicate are errors; `.dark()`.
- [ ] the generated CSS for a theme; the classes for a merged surface; later looks win field by field; `[]` gives none.
- [ ] `.style`, `.button_style`, `.theme`, `.maximized`, `.header` land where they should; `map` keeps them and maps header messages.
- [ ] diff: adding, changing and removing `.style` is a `SetStyle`; header views diff by position (a changed title, a new end button, a removed start button); a framed body changes like any view.
- [ ] testing: `click` finds a header button; `type_into` a header field; a missing look, `.style` without a theme, `.header` below the root are errors before GTK.

Live GTK (skipped without a display), `test/live_look.la`:
- [ ] the provider is loaded for the display with the theme's CSS (read back with `to_string`); a node's classes are applied, changed and removed on update, GTK's own classes kept; a `text` colour reaches the widget (`gtk_widget_get_color`).
- [ ] `.dark()`: a label's text colour is light.
- [ ] `.maximized()`: the window is maximized once shown (or the check skips, saying why, where the window manager can't).
- [ ] the header bar is the window's titlebar, with title buttons shown; its start, title and end widgets are kivuli's; a click on a header button sends its message; changing the title view patches in place; an expanding title widget is wider than its natural size.
- [ ] lugha-gtk `test/look.la`: each new extern and C helper.

## ROLLBACK PLAN
- Branch `prp-007-app_look` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.23 is additions only.

## ACCEPTANCE CRITERIA
- [ ] An app paints parts of its frame with typed looks, asks for dark, opens maximized, and has its own title bar; without them, apps look as before.
- [ ] Every example builds after the `.button_style` rename.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines (a split proposed first if one gets there).
- [ ] README, docs/DESIGN.md, CHANGELOG (with migration), MEMORY.md, DECISIONS.md updated; 0.2.0 tagged; merged into `main`. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: a small example (`examples/frame.la`) with a dark theme, a coloured rail, a header with back/forward and an address field, opening maximized. *(proposed: the showcase)*
