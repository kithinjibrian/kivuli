# kivuli

*kivuli* (Swahili: "shadow"): the view is a shadow of the model.

A declarative UI framework for [Lugha](https://github.com/kithinjibrian/lugha), built on the Elm architecture and rendered to real GTK 4 widgets through [lugha-gtk](https://github.com/kithinjibrian/lugha-gtk). An app is three pure functions: a model, `update` making the next model from a message, and `view` describing the screen. kivuli shows the view, turns clicks and keystrokes into messages, and on each one diffs the new view against the last and changes only the widgets that differ.

```
import kivuli {
    View,
    Width,
    Edges,
    Spacing,
    Font,
    Text,
    Weight,
    Size,
    Paint,
    button,
    column,
    input,
    row,
    sandbox,
    text,
};

/** The same room on every side, inside the node. */
fun pad(n: i64): Spacing = Spacing {
    padding: Edges { all: Width.Px(n), ..Edges.theme() },
    ..Spacing.theme(),
};

/** Room between a column's, row's or wrap's children. */
fun gap(n: i64): Spacing = Spacing { gap: Width.Px(n), ..Spacing.theme() };

struct Model {
    count: i64,
    name: string,
}

enum Msg {
    Add,
    Reset,
    Named(string),
}

fun update(m: Model, msg: Msg): Model = match msg {
    Add => Model { count: m.count + 1, ..m },
    Reset => Model { count: 0, ..m },
    Named(name) => Model { name: name, ..m },
};

fun view(m: Model): View<Msg> = column([
    input(m.name, Msg.Named).placeholder("Your name"),
    text("Hello, {m.name}").style([
        Font { size: Size.Pt(18), weight: Weight.Bold, ..Font.theme() },
    ]),
    text("Clicked {m.count} times").style([Text { color: Paint.Faded(0.55), ..Text.theme() }]),
    row([button("Add", Msg.Add), button("Reset", Msg.Reset).disabled(m.count == 0)]).style([
        gap(8),
    ]),
]).style([pad(24), gap(8)]).window("Hello", 320, 240);

fun main(): i32 = sandbox("org.kivuli.hello", Model { count: 0, name: "" }, update, view);
```

The app never touches a widget. A disabled button has no message to send; a text field keeps its cursor while you type; nothing is rebuilt that didn't change.

## Requirements

- `lughac` built from [lugha](https://github.com/kithinjibrian/lugha)'s `main`, milestone 100 or later (kivuli uses struct update, variants as functions, package entries, glob re-exports and recent inference fixes). In a lugha checkout: `cargo install --path . --locked`.
- GTK 4's development files and `pkg-config`. On Ubuntu:

```sh
sudo apt install libgtk-4-dev pkgconf
```

lugha-gtk (0.1.13 or later) is fetched by `lughac` itself, as a git package.

## Using it

List it in your program's `lugha.toml`:

```toml
[packages]
kivuli = { git = "https://github.com/kithinjibrian/kivuli", version = "0.1" }
```

and write `import kivuli { … };` or `import kivuli;` (then `kivuli.sandbox(…)`).

Inside this repository, run the examples directly:

```sh
lughac run examples/hello.la      # the app above
lughac run examples/type.la       # every Font and Text value, each on a line naming it
lughac run examples/backgrounds.la   # colours, tints, gradients, pictures in each fill, all three layers
lughac run examples/borders.la    # each line style, one side at a time, corners, outlines
lughac run examples/tasks.la      # a to-do list: keyed rows, a row menu
lughac run examples/browser.la    # a header bar, tab menus, tiles, whole-row buttons, background loads
lughac run examples/controls.la   # drop-down, sliders, check box, text area, icons, pictures
lughac run examples/split.la      # one component, two copies, through map
```

## What's there

**Running an app**

| | |
| --- | --- |
| `sandbox(id, init, update, view)` | An app without effects: `update(model, msg)` returns the next model. |
| `application(App { id, init, update, view, subscriptions, on_open })` | An app with effects: `init` and `update` also return a `Cmd`, and `subscriptions(model)` says what to keep listening to. `on_open: Some(Msg.Open)` makes it open things: the items on its command line, and a later launch's (which then exits), arrive as `Open(items)` after `init`, each a local file's absolute path or an address as GIO writes it (`rapidkoi://host` gains a trailing `/`); `None` opens nothing, and GTK sees no command line. A second launch brings the running window to the front. |

**Widgets**

| | |
| --- | --- |
| `text(s)` | A line of text. |
| `button(label, msg)` | Sends `msg` when clicked. |
| `button_with(content, msg)` | A button showing any view, such as a whole row (`row([text("Account").expand(), text(summary), icon("go-next-symbolic")])`): one click target, drawn by the theme as a button, with keyboard focus and Enter. The content fills it. `.disabled`, `.style`, `.tooltip`, `.shortcut` and `.menu` work as on `button`. The content can't hold anything interactive (buttons, fields, check boxes, drop-downs, sliders): kivuli stops with a message naming it. |
| `input(value, on_change)` | A one-line text field; each edit sends `on_change(text)`. |
| `text_area(value, on_change)` | A multi-line text field that wraps and scrolls inside itself; each edit sends `on_change(text)`, Enter starts a new line. |
| `checkbox(label, checked, on_toggle)` | Each toggle sends `on_toggle(ticked)`. |
| `dropdown(choices, selected)` | A drop-down of `(label, msg)` choices; picking one sends its message. `selected` is the label shown picked; one matching no label shows a blank item. |
| `icon(name)` | A theme icon by name (`"user-trash-symbolic"`); symbolic icons follow the light or dark theme. An unknown name shows GTK's placeholder. |
| `image(Image.File(path))`, `image(Image.Bytes(key, bytes))` | A PNG, JPEG or SVG picture, from a file or from bytes in memory (an SVG is drawn by GTK at any size, reading nothing outside itself), scaled to its room (`.fit`, a `Sizing`, or `.size(n)` for an n × n box). `image(Image.Icon(name))` is `icon(name)`. Bytes are known by their key: the same key is never decoded again, a new key replaces the picture. A relative path is relative to where the app was started. Anything that can't be shown (not PNG, JPEG or SVG, corrupt, over 4096 × 4096 px or 16 MB, a missing file, an SVG holding `<image>`) shows nothing; the app keeps running. |
| `slider(min, max, value, on_change)` | A whole-number slider showing its number; each move to a new step sends `on_change(value)`. `.step(n)` moves it in steps of `n` from `min`. |
| `column(children)`, `row(children)` | Children top to bottom, or left to right. |
| `keyed_column(children)` | Like `column`, with `(key, view)` children matched by key: reordering, inserting or deleting moves, adds or removes just those widgets. |
| `wrap(children)`, `keyed_wrap(children)` | Children side by side, moving to a new line when a line is full, so they re-flow as the window resizes (tiles, tags). `keyed_wrap` matches `(key, view)` children by key, like `keyed_column`. No selection: clicks come from the children. |
| `popover(trigger, open, content, on_close)`, `popover_at(side, …)` | `content` in a popover beside `trigger` (below by default) while `open`; a click outside or Escape sends `on_close`, so the model says whether it's open, and a button inside closes it by setting the model. The content is built while closed too, so its fields keep their text. `kivuli.ui`'s `ui.popover(…)`/`ui.popover_at(…)`: a panel in the popover colours, 288 px wide, no arrow. |
| `overlay(base, layers)` | `layers` drawn on top of `base`, in order (a later one above). `[]` shows only the base, so opening and closing a panel is just the list changing; the base is never rebuilt. The overlay takes the base's size. |
| `scroll(child)` | Scrolls its child when it's bigger than the room it's given; it takes the spare room. |
| `scroll_fit(child)` | As big as its child until its parent can't give that much, then it scrolls; it takes no spare room, so what follows it comes right after (a list then a + button). |
| `clip(child)` | Cuts its child off where it doesn't fit, and never pushes its parent (or the window) bigger: the child gets at least the room it needs, at the top start, and what sticks out at the end side and the bottom is cut off. `.on_overflow(f)` sends `f(Overflowing { across, down })`, in pixels, whenever that changes (0 when it fits). `.expand()` it to fill. Inside a `scroll` styled `Overflow.scroll_y()`, a too-wide row is cut instead of widening the list. |
| `spacer()`, `separator()` | Room that grows along its column or row; a line across it. |
| `lazy(key, build)` | Calls `build` only when `key` changes at that place; otherwise reuses the subtree and skips diffing it. The key must cover everything the subtree shows. |

**Modifiers**, chained on any view (a modifier that doesn't fit a node is ignored):

| | |
| --- | --- |
| `.expand()`, `.align(Align.Start \| Center \| End)` | Take spare room along the parent; place across it. |
| `.style([...])` | How the node looks: typed values (`Font`, `Text`), the last that sets a field winning. See **Styles** below. |
| `.size(n)` | Icons and a button's image: `n` pixels; a picture: an n × n box. Not text: a text's size is a `Font`'s, and `.size` on text stops the app with a message saying so. |
| `.wrap()`, `.wrap_at(chars)`, `.ellipsize()` | Long text on text and check box labels: onto more lines (at most `chars` wide), or cut with "…"; both: up to three lines, the last cut. |
| `.selectable()` | Text the user can select and copy. |
| `.tooltip(hint)` | Hover text on any node; tests can click a button by it. |
| `.shortcut(keys)` | Keys that press a button from anywhere in the window: `"Ctrl+N"`, `"Ctrl+Shift+Delete"`, `"Escape"`. A disabled button's shortcut does nothing; unreadable keys, or two enabled buttons with the same ones, stop the app with a message. |
| `.autofocus()` | Keyboard focus goes to the node when it's first shown. Clicking anywhere that doesn't take focus itself (empty space, text, pictures) clears the window's focus, as on the web. |
| `.disabled(bool)` | Buttons, text fields and text areas: greyed out by the theme, and they send nothing. A disabled field still shows its value and follows the model. `.disabled(false)` changes nothing. |
| `.step(n)` | Sliders: move in steps of `n`, counted from the least value. |
| `.equal()`, `.max_per_line(n)` | Wraps: every child as wide as the widest (a tidy grid); at most `n` children on a line (`n <= 0`: no limit, the default). |
| `.menu([item(label, msg), menu_separator(), …])` | Any node: a context menu, opened by a right-click (at the pointer), a long press, or the Menu key / Shift+F10 while the node or something in it has focus. Picking an item sends its message; `item(…).disabled(true)` greys it out and it sends nothing. `[]` takes the menu away. The theme draws it as a native popover menu. |
| `.place(across, down)` | An overlay's layer: where it sits on the base, each way an `Align` (`Fill` covers that whole way). Without it a layer covers the base. |
| `.image(Image.…)`, `.icon(name)` | Buttons: an image (a theme icon, a file or bytes) before the label; with the label `""`, an image-only button (give it a `.tooltip`, which tests find it by). `.icon(name)` is `.image(Image.Icon(name))`; `.icon("")` takes the image away. |
| `.fit(Fit.Contain \| Cover \| ScaleDown \| Fill)` | Pictures: the whole picture (default), filled and cropped, never enlarged, or stretched. |
| `.placeholder(hint)`, `.on_submit(msg)` | Text fields and text areas: a hint, dimmed, while empty; the message Enter sends (one-line fields). |
| `.name(id)` | A name tests find a text field, text area, drop-down or slider by (not shown). |
| `.window(title, width, height)` | On the root of `view`: the window's title (it may follow the model) and size. |
| `.maximized()` | On the root: the window opens maximized; `.window`'s size is its size when unmaximized. |
| `.header(header_bar(start, title, end))` | On the root: a header bar as the window's title bar, with `start` views, a `title` view (`.expand()` fills the bar) and `end` views; GTK draws the window's controls. `header_bar(…).style([...])` styles the bar itself, and GTK's window controls through `Inside(Part.WindowButton, …)`, `Part.WindowClose`, `Part.WindowIcon` and `Part.WindowCloseIcon` (GTK keeps which show, their order and what they do); `kivuli.ui`'s `WindowControls.look()` makes them ghost buttons in the palette's colours, the close one red on hover; the root's `.style` styles the page. An exact or maximum size, a gap and a background image are the window's, and refused on a bar. |

Everything visual comes from the GTK theme unless a node's `.style` says otherwise, and apps follow the user's light or dark theme. kivuli ships no colours.

**Styles.** How a node looks is a list of typed values, given with `.style([...])`. There are no class names to invent and nothing to register: a style is written on the node, and reused through an ordinary function.

```
text("Storage").style([Font.size(22), Font.bold()])
text(m.folder).style([Font.mono(), Font.tabular(), Text.dim()])
card.style([Spacing.padding(16), Border.rounded(12), Border.all(1, Paint.Faded(0.15)), Shadow.md()])
text("Price").style([Text { decoration: Decoration.Strikethrough, ..Text.theme() }])   // the full value
```

The short forms (utilities) are static functions on each group, each returning the full value it stands for, so the two mix freely:

| Group | Utilities |
| --- | --- |
| `Font` | `h1()` (black, 2×), `h2()` (black, 1.5×), `h3()` (bold, 1.5×), `h4()` (bold, 1.3×), `heading()` (bold, 1.1×), `body()` (normal, 1.1×), `caption()` (normal, 0.9×); `xs()` 0.75×, `sm()` 0.875×, `base()`, `lg()` 1.125×, `xl()` 1.25×, `xxl()` 1.5×; `size(pt)`, `scale(x)`; `thin()`, `light()`, `normal()`, `medium()`, `semibold()`, `bold()`, `black()`; `italic()`; `sans()`, `serif()`, `mono()`, `family(name, fallback)`; `tabular()` |
| `Text` | `upper()`, `lower()`, `capitalize()`; `underline()`, `strike()`; `tight()` (1.2), `loose()` (1.6), `line_height(x)`; `wide()` (+1 px), `narrow()` (−1 px), `letter_spacing(px)`; `dim()` (faded 0.55), `faded(x)`, `color(c)` |
| `Background` | `color(c)`, `tint(x)` (the theme's text colour at `x`), `linear(angle, stops)`, `radial(stops)`, `image(img)` (cover) |
| `Border`, `Outline` | `all(px, paint)`, `top`/`right`/`bottom`/`left(px, paint)` (solid), `rounded(px)`, `corners(tl, tr, br, bl)`, `none()`; `Outline.ring(px, paint)` (solid), `Outline.offset(px)` |
| `Spacing` | `padding(px)`, `padding_x`, `padding_y`, `padding_top`, `_right`, `_bottom`, `_left`; the same for `margin` (may be negative); `gap(px)` |
| `Sizing` | `width(px)`, `min_width`, `max_width`, `height`, `min_height`, `max_height` |
| `Overflow` | `scroll_y()` (a `scroll` or `scroll_fit` scrolls up and down only: text wraps at its width, and a row that can't shrink makes it wider rather than scroll sideways), `scroll_x()` (sideways only); without it a scroll scrolls both ways. Not under `When` or `Inside` |
| `Accessible` | `role(Role.Checkbox)` (`Button`, `Checkbox`, `Switch`, `Tab`, `TabList`, `Heading`, `List`, `ListItem`, `Group`, `Separator`: on `column`, `row`, `button_with` and `text`; changing it rebuilds the node), `checked(on)`, `selected(on)`, `label(text)` (also on `input` and `text_area`): what a screen reader hears. A disabled node is announced as such from `.disabled`. Not under `When` or `Inside` |
| `Shadow`, `Effects` | `Shadow.sm()`, `md()`, `lg()`, `xl()` (Tailwind's layered shadows), `Shadow.off()`; `Effects.opacity(x)`, `blur(px)`, `grayscale()`, `rotate(deg)`, `scale(x)`, `translate(x, y)` |

**Palettes and colour tokens.** A colour can be a name instead of a value: `Token.Primary`, `Token.MutedForeground`, `Token.Border` and the rest of shadcn's set (`Background`, `Foreground`, `Card`, `Popover`, `Primary`, `Secondary`, `Muted`, `Accent`, `Destructive`, each with a `…Foreground`, and `Border`, `Input`, `Ring`). A palette on the root gives each a light and a dark value, and GTK picks one as the system's scheme is (or as `.scheme(…)` sets it), so dark mode needs no code. The page takes the palette's `Background` and `Foreground`; GTK's own widgets keep their look.

```
text(m.note).style([Text.color(Token.MutedForeground)])
card.style([Background.color(Token.Card), Border.all(1, Token.Border), Border.rounded_lg()])

column(rows)
    .palette(Palette.zinc().with(Token.Primary, hex("#2563eb"), hex("#3b82f6")).radius(8))
    .scheme(m.scheme)      // Scheme.System (the default), Light or Dark
```

`Palette.zinc()`, `slate()`, `stone()`, `gray()` and `neutral()` are shadcn's bases; `.with(token, light, dark)` changes one colour; `.radius(px)` is the radius `Border.rounded_lg()` uses (`rounded_md()` is 2 less, `rounded_sm()` 4 less; `Radius.Large`, `Medium`, `Small`). Without a palette, tokens have zinc's values. Every colour utility takes a `Color`, a `Token` or a `Paint`. `examples/palette.la` switches bases and schemes.

**Where text sits.** Text starts at the start of its line (the left, or the right in right-to-left text), as on the web; `Text.center()` and `Text.end()` move it, wrapped lines included. It's a label's own setting, not CSS, so it can't change inside `When`.

**Components** (`kivuli.ui`). Ready-made pieces in the palette's colours, after shadcn, beside GTK's own widgets. A component's variant is a `Look` (several style values as one) given through `.style`, so an app's own values after it win; each component is built only from the public style API, so an app can copy one and change it.

```
import kivuli.ui { badge, badge_with, Badge };

row([text("Inbox"), badge("12").align(Align.Center)])
badge_with([icon("emblem-ok-symbolic"), text("Verified")])
badge("Draft").style([Badge.outline()])      // also Badge.secondary(), Badge.destructive()
badge("Beta").style([Badge.outline(), Text.color(Token.Primary)])
```

So far:
- `badge` (small, medium-weight, corners at the palette's radius − 4) and `badge_with([icon(…), text(…)])` (icons shrink to 12 px).
- `ui.button(label, msg)` and `ui.button_with(content, msg)` (icons 16 px), beside the native `button`: variants `Button.destructive()`, `outline()`, `secondary()`, `ghost()`, `link()`; sizes `Button.sm()`, `lg()`, `icon()` (a square). A variant sets colours, a size sets size, so `[Button.outline(), Button.sm()]` is both. It's still a native button: `.disabled`, `.shortcut` and `kivuli.testing` work as on any.

- `card([...])` with `card_header(title, description)`, `card_header_with(title, description, action)`, `card_content([...])` and `card_footer([...])` (a muted band to the card's edges). Each part is a plain view, so any can be left out or repeated. One size for now.

- `ui.input(value, on_change)` beside the native `input`: an `Input` border, a ring while typed in, `Input.invalid()` (red). `.placeholder`, `.on_submit`, `.disabled` work as on any field.

- `field([field_label(…), control, field_description(…)])`, `field_horizontal([...])` (a check box beside its text), `field_error(…)`, `field_group([...])` (fields 24 px apart) and `Field.invalid()` (the label turns red; mark the control with `Input.invalid()`).

- `input_group(start, control, end)`: one frame around a `group_input` and what sits beside it: icons, `group_text("https://")`, `group_button(…)` / `group_button_with([icon(…)], …)` (compact ghost buttons). `InputGroup.invalid()` turns the frame red.

- `ui.separator()` (a 1 px `Border` line: across a column, down a row) and `separator_with("Or continue with")`.

- `ui.checkbox(label, checked, on_toggle)`: a 16 px box, filled with a tick when checked; `.disabled(…)` greys it.

- `ui.slider(min, max, value, on_change)`: a round track, a primary fill and a round thumb with a halo on hover and focus. `.disabled(…)` works on any slider.

- `ui.dropdown(choices, selected, placeholder)`: a trigger like `ui.input`, a list in the popover colours, items that turn `Accent` on hover; `Dropdown.invalid()`. (`dropdown`, not `select`: `select` is a Lugha keyword.) `.placeholder(…)` on any drop-down labels its blank item. `.show_value(on)` shows or hides a slider's number (`slider` shows it, `ui.slider` hides it). `dropdown_grouped([(heading, choices), …], selected)` and `ui.dropdown_grouped(…, placeholder)` show the choices under headings, a line between groups. `.disabled(on)` greys any drop-down, which then sends nothing (`kivuli.testing.choose` gives `Err(Lookup.Silent(…))`).

- `ui.switch(on, on_toggle)`: a round track (`Input` off, `Primary` on) and a thumb GTK slides; put its label beside it in a `field_horizontal`.

- `ui.tabs([tab("Account", Msg.Tab(0)), …], m.tab)` (a muted pill, the active trigger raised) and `ui.tabs_line(…)` (the active one underlined); `tab_with([icon(…), text(…)], msg)`; `.disabled(…)` on a trigger; `ui.tabs_vertical(…)` and `ui.tabs_line_vertical(…)` stack them. Which tab is open is the model's: show its panel beside the tabs.

- `ui.dropdown_menu(trigger, entries)`: a click on `trigger` opens its menu below it, in the palette's look (`Menu.look()`, which a right-click `.menu` can take too). Entries: `item`, `.hint("Ctrl+P")` (a shortcut shown at its end), `.disabled`, `menu_label` (a group's title), `menu_separator`, `submenu(label, entries)` (opening beside), `check_item(label, checked, on_toggle)` and `radio_group(selected, choices)` (ticked from the model). Any view takes `.dropdown_menu(entries)`. GTK draws no icons beside menu items.

- `Tooltip.look()` on the root view: every `.tooltip(…)` in the app as a small inverse label, fading in (GTK puts tooltips beside the root, so the look goes there, once).

- `ui.progress(value)` (out of 100): a round track in `Primary` at 20% and a `Primary` fill; the native `progress(value)` is GTK's.

- `ui.avatar(image, initials)`: a round 32 px picture over its initials, which show when the picture can't (`None`, a missing or broken file); `Avatar.size(px)`; `avatar_group([...])` stacks them, ringed in the page colour.

- `page.dialog(if open { Some(ui.dialog(Msg.Close, [dialog_header(title, description), content, dialog_footer([buttons])])) } else { None })` on the root: a card over a dimmed page, which is inert while it's open; the ✕, Escape and a backdrop click send the close message. One window: the dialog is drawn in it. Any node takes `.inert(on)` and `.focusable(on)`. `ui.sheet(close, Edge.Right, parts)` is a `Dialog` too: a panel against an edge (384 px wide on a side, the full width on the top or bottom), sliding in.

**A widget's parts.** `Inside(Part.Check, [...])` styles the part inside a widget: a check box's box (`Part.Check`), a slider's track, fill and thumb (`Part.Trough`, `Highlight`, `Knob`), a drop-down's trigger, arrow, list and items (`Part.Trigger`, `Arrow`, `List`, `Item`); a `When` inside an `Inside` is the part's own state (an item's hover); inside `When`, the state is the widget's (`When(State.Checked, [Inside(Part.Check, [...])])`). `Mark.icon(name, px)` is the icon GTK draws in a part (the tick). More parts come with the components that need them.

A token can be used at an opacity: `Token.Primary.alpha(0.9)` (a hover), `Token.Ring.alpha(0.5)` (a focus halo). `look([...])` makes an app's own variant. `examples/components.la` shows each in light and dark.

**Motion.** What eases when a style changes, how a node comes in, and what loops: GTK's own CSS transitions and animations.

```
button("Save", Msg.Save).style([When(State.Hover, [Background.color(Token.Accent)]), Motion.colors(150)])
tile.style([When(State.Hover, [Effects.scale(1.05), Shadow.lg()]), Motion.transform(200), Motion.shadow(200), Motion.easing(Easing.Out)])
toast(t).style([Motion.fade_in(200), Motion.slide_in(Edge.Bottom, 8, 200)])
icon("process-working").style([Motion.spin()])
```

`Motion.colors(ms)` (text, background, borders, outline), `opacity`, `transform`, `shadow` and `all` (sizes too: opt-in) ease those properties; `Motion.easing(…)` (`Ease`, Tailwind's default; `Linear`, `In`, `Out`) and `Motion.delay(ms)` set the curve and the wait. `fade_in(ms)` and `slide_in(edge, px, ms)` play when a node's widget is first shown (a widget kept across updates doesn't replay); `spin()` and `pulse()` loop. When the system asks for less motion, eases and entrances are instant and `spin` and `pulse` keep going. `examples/motion.la` shows each.

The full values, field by field:

| Value | Fields |
| --- | --- |
| `Font` (the typeface) | `family` (`Sans`, `Serif`, `Monospace`, `Named("Inter", Family.Sans)`: a named font names its fallback), `size` (`Pt(14)` points, or `Scale(1.5)` times the theme's), `weight` (`Thin`, `Light`, `Normal`, `Medium`, `Semibold`, `Bold`, `Black`), `style` (`Normal`, `Italic`), `tabular` (digits all one width) |
| `Background` (behind the content) | `color` (a `Paint`: `Faded(0.08)` is a tint of the theme's text colour that follows light and dark), `gradient` (`Linear(180, stops)` in degrees, 0 up and 90 right, or `Radial(stops)`; each `Stop { color, at }`, `at` from 0.0 to 1.0), `image` (`Some(Image.File(…))`, `Image.Bytes(…)` or `Image.Icon(…)`), `fill` (`Cover`, `Contain`, `Tile`, `Stretch`; always centred). The image sits over the gradient, over the colour. |
| `Border` (around the content) | `all` (every side) and `top`, `right`, `bottom`, `left` (each a `Side { width: Width.Px(n), color, style }`; a side's own fields beat `all`), `radius` (`Radius.All(12)` or `Radius.Corners(tl, tr, br, bl)`; a rounded node clips what's inside it). `Line` is `Solid`, `Dashed`, `Dotted`, `Double`, `Groove`, `Ridge`, `Inset`, `Outset` or `None`; a width with no line is drawn solid. |
| `Spacing` (room around and between things) | `padding` (inside the border) and `margin` (outside it; may be negative), each an `Edges { all, top, right, bottom, left }` of `Width.Px(n)` (a side's own value beats `all`), and `gap` (between a column's, row's or wrap's children). |
| `Sizing` (how big) | `width`, `min_width`, `max_width`, `height`, `min_height`, `max_height`, each a `Width.Px(n)`; `icon`, the size icons are drawn at inside the node (`Sizing.icon(24)`). An exact size is kept between the minimum and the maximum (the minimum wins), and a node is never smaller than its content's own minimum, so `max_width` on wrapping text makes a readable column. |
| `Shadow` (shadows) | `boxes` (under the box: a card lifting off the page) and `text` (under the text), each a list of `Drop { x, y, blur, spread, color, inset }` drawn together (`spread` and `inset` on boxes only). `Shadow.off()` removes the theme's own box shadow. |
| `Effects` (how the whole node is drawn) | `opacity` (`Some(0.5)`: children and all), `filters` (`Filter.Blur(px)`, `Brightness`, `Contrast`, `Grayscale`, `HueRotate(deg)`, `Invert`, `Saturate`, `Sepia`, in order) and `transform` (`Move.Translate(x, y)`, `Rotate(deg)`, `Scale`, `ScaleXY`, in order; never changes the room the node takes, so a hover can grow it safely). |
| `When(state, [...])` (a look for one state) | `State.Hover`, `Press`, `Focus` (keyboard), `FocusWithin` (focus anywhere inside: a text field being typed in), `Disabled`, `Checked`: the values apply only while the node is in that state, merged over its own. When two are on at once the later in that order wins (Hover < Focus < FocusWithin < Press < Checked < Disabled). A background image, an exact or maximum size and the gap can't change per state. **Group states**: `.style([Group.mark()])` makes a node a group, and a node inside it takes `State.GroupHover` while the pointer is anywhere over the group, `State.GroupFocusWithin` while the focus is in it (a row's × shown on hover); for an overlay's layers, mark the overlay. |
| `Outline` (a line outside the border that takes no room) | `width`, `color`, `style` (a `Line`), `offset` (pixels; negative draws inside). It hides while the node has keyboard focus, so the theme's focus ring still shows. |
| `Text` (what's done with the letters) | `color` (`Paint.Color(hex("#e0e0e0"))`, or `Paint.Faded(0.55)`: the theme's text colour at that opacity, so it follows light and dark), `line_height` (`Times(1.4)`), `letter_spacing` (pixels; negative tightens), `decoration` (`Underline`, `Strikethrough`, `None`), `transform` (`Uppercase`, `Lowercase`, `Capitalize`, `None`) |

Each field's `Theme` value (and `Font.theme()`, `Text.theme()` for a whole group) means "not set here". The values apply in order, and for each field the last one that sets it wins: `[big, Font { weight: Weight.Thin, ..Font.theme() }]` keeps `big`'s size and makes it thin. `.style(a).style(b)` is `.style(a + b)`. What nothing sets stays the GTK theme's, so an app follows the user's light or dark theme with no work. A bad value (`Size.Pt(0)`, a line height of 0, an opacity over 1, a font name with quotes or braces) stops the app with a message naming it. kivuli turns each value into one CSS class (`kv-fw-700`), adds its rule to the app's stylesheet the first time it's seen, and never draws anything itself.

A background picture from a file or bytes is checked like `image(…)` (PNG or JPEG, no SVG, at most 4096 × 4096 px and 16 MB) and handed to GTK as bytes, never as a path; one that can't be shown shows no image, and the app keeps running. Button looks aren't styles yet: they come in a later release.

**Components.** A part of an app can have its own model, messages, `update` and `view`, and sit in a bigger app through `map`, which sends `f(msg)` wherever the part would send `msg`:

```
row([counter(m.left).map(Msg.Left), counter(m.right).map(Msg.Right)])
```

`Cmd` and `Sub` have `map` too, for a part's commands and subscriptions. `examples/counters.la` shows two copies of one component.

**Effects**, for `application`:

| | |
| --- | --- |
| `Cmd.none()`, `Cmd.batch(cmds)` | Nothing; several commands. |
| `Cmd.after(ms, msg)` | Send `msg` once, `ms` milliseconds from now. |
| `Cmd.quit()` | End the app. |
| `Cmd.focus(name)` | Move keyboard focus to the node named `name` (`.name(name)`). |
| `Cmd.scroll_to(name)` | Scroll the nearest `scroll`/`scroll_fit` around the node named `name` as little as it can to show it (nothing if it's in view); the focus stays where it is. |
| `Cmd.copy_text(text)` | Put `text` on the clipboard, for pasting anywhere until something else is copied. |
| `Cmd.task(work, done)` | Run `work` (a function taking nothing) in the background, on a green thread, and send `done(result)` when it returns: exactly one message. `update` stays fast and testable: it only describes the work. |
| `Sub.none()`, `Sub.batch(subs)` | Nothing; several subscriptions. |
| `Sub.every(ms, f)` | Send `f(now)` every `ms` milliseconds (`now` in milliseconds since 1970) while `subscriptions` keeps returning it. |
| `Sub.receive(chan, f)` | Send `f(value)` for every value sent on a Lugha channel, in order, while `subscriptions` keeps returning it: how a worker built with `spawn`, channels or actors reports progress. |
| `Sub.window_active(f)` | Send `f(true)` when the window becomes the active one (comes to the front) and `f(false)` when it stops being active, while `subscriptions` keeps returning it. Changes only: nothing when it starts listening, never the same value twice. |

**Background work** (`Cmd.task`, `Sub.receive`). Lugha's green threads share the program's one OS thread with GTK, and kivuli gives them turns while the window runs. What that means in practice:

| Work | |
| --- | --- |
| Downloads and sockets (`std.http`, `std.net`, `std.tls`), `task.sleep` | in the background: the window stays responsive |
| Computation written in Lugha | shares the window in slices: the window stays responsive, the work runs slower than alone |
| A single blocking call: reading a file, a DNS lookup (at a connection's start), `std.crypto`, `std.process` | still holds the window until it returns, task or not (a Lugha runtime change, DECISION-022) |

A task runs on a copy of what it captured, must not call kivuli or GTK, and can't be cancelled (ignore a result you no longer want); a panic in it ends the program. `examples/download.la` shows a download with a spinner and a counter reporting progress.

## Testing an app

`update` and `view` are pure, so app tests need no window. `kivuli.testing` works out what a user's action would send from a view alone:

```
import kivuli.testing { Lookup, click, is_disabled, is_field_disabled, type_into, submit, toggle, choose, slide, texts };

@test
fun adding_a_task() {
    assert_eq(is_disabled(view(start()), "Add"), Ok(true));
    assert_eq(type_into(view(start()), "What needs doing?", "milk"), Ok(Msg.Draft("milk")));
    assert_eq(click(view(start()), "Remove"), Err(Lookup.NotFound("button \"Remove\"")));
}
```

`click` and `is_disabled` (by a button's label or tooltip; a `button_with` by its tooltip, `.name` or any text inside it), `press` (by shortcut), `tooltip`, `type_into` and `submit` (finding a text field or text area by its placeholder or `.name`), `toggle`, `choose(view, name, label)` and `slide(view, name, value)` (a drop-down or slider by its `.name`; `slide` clamps and rounds to the step as the slider would) and `texts` each return a `Lookup` error when what's asked for isn't there (`NotFound`), isn't unique (`Ambiguous`) or sends nothing (`Silent`: a disabled button or field, or Enter without `.on_submit`). `is_field_disabled(view, hint)` tells whether a text field or text area is disabled. `pick(view, owner, item)` is the message of a context menu's item, the menu found by its node: a button's label or tooltip, or a `.name` (`Silent` for a disabled item). `examples/todo.la` ends with its own tests: `lughac test examples/todo.la`.

## Tests

```sh
lughac test test/
```

The tests of the view tree, the diff, effects, `lazy` and the testing helpers need no display. The tests of the live GTK layer (`test/live*.la`) build hidden windows and read the widgets back from GTK; without a display they report themselves skipped.

## License

GPL-3.0-only, like Lugha.
