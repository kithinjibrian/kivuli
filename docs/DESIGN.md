# Design System — kivuli

kivuli draws nothing. Every visual comes from the user's GTK theme (Adwaita by default). This file maps kivuli's style vocabulary onto GTK, and it is the only place those mappings are decided.

---

## Tokens: kivuli vocabulary → GTK

### Styles (`.style([...])`; PRPs 016–020, DECISION-023)

A style is a list of typed values (`Font`, `Text`, `Background`, `Border`, `Outline`, `Spacing`, `Sizing`) merged into one `Sheet` per node: for each field the last value that sets it wins, and `Theme` sets nothing. kivuli turns a `Sheet` into atomic CSS classes, one per property and value, so every node using a value shares its class and a class's rule never changes:

| kivuli | class | GTK CSS |
| --- | --- | --- |
| `Font.weight` `Thin` … `Black` | `kv-fw-100` … `kv-fw-900` | `font-weight: 100` … `900` |
| `Font.size` `Pt(n)` / `Scale(x)` | `kv-fs-{n}pt` / `kv-fs-s{x·1000}` | `font-size: {n}pt` / `{x·100}%` |
| `Font.family` `Sans` / `Serif` / `Monospace` | `kv-ff-sans` / `serif` / `mono` | `font-family: sans-serif` / `serif` / `monospace` |
| `Font.family` `Named(name, fallback)` | `kv-ff-{name in hex}-{fallback}` | `font-family: "name", fallback` |
| `Font.style` `Italic` / `Normal` | `kv-fst-italic` / `normal` | `font-style` |
| `Font.tabular` | `kv-tnum` | `font-variant-numeric: tabular-nums` |
| `Text.color` `Color(c)` | `kv-fg-{rrggbbaa}` | `color: #rrggbbaa` |
| `Text.color` `Faded(x)` | `kv-fade-{x·1000}` | `color: alpha(currentColor, x)`: the theme's text colour faded, so it follows light and dark as `dim-label` does |
| `Text.line_height` `Times(x)` | `kv-lh-{x·1000}` | `line-height: x` |
| `Text.letter_spacing` n | `kv-ls-{n}` / `kv-ls-m{-n}` | `letter-spacing: {n}px` |
| `Text.decoration` | `kv-td-underline` / `strike` / `none` | `text-decoration-line` |
| `Text.transform` | `kv-tt-upper` / `lower` / `cap` / `none` | `text-transform` |

| `Background.color` `Color(c)` / `Faded(x)` | `kv-bg-{rrggbbaa}` / `kv-bgfade-{x·1000}` | `background-color: #…` / `alpha(currentColor, x)` |
| a colour alone | `kv-bgi-none` | `background-image: none`: a themed widget's own image (a button's) would cover the colour |
| `Background.gradient` (no image) | `kv-bgi-g{css in hex}` | `background-image: linear-gradient({angle}deg, …)` / `radial-gradient(circle, …)`, stops as `colour at%` |
| `Background.image` (with the gradient and fill) | `kv-bgp-{identity, gradient, fill in hex}` | `background-image: url("data:image/png;base64,…") \| -gtk-icontheme("name"), gradient; background-size; background-position: center; background-repeat` |

| `Border` side (after `all` is folded in) | `kv-bw-{side}-{n}`, `kv-bs-{side}-{line}`, `kv-bc-{side}-{rrggbbaa}` / `kv-bcfade-{side}-{x·1000}` | `border-{side}-width`, `-style`, `-color`; a width with the line left `Theme` is `solid` |
| `Border.radius` | `kv-br-{tl\|tr\|br\|bl}-{n}` | `border-{corner}-radius: {n}px`, one class per corner; a rounded node gets `overflow: hidden` (`gtk_widget_set_overflow`, marked `kv-clip`) |
| `Spacing.padding` / `.margin` (after `all` is folded in) | `kv-p-{side}-{n}` / `kv-m-{side}-{n}`, `kv-m-{side}-m{n}` | `padding-{side}` / `margin-{side}: {n}px` (a margin may be negative); kivuli no longer sets GTK's widget margins |
| `Sizing.min_width` / `.min_height` | `kv-minw-{n}` / `kv-minh-{n}` | `min-width` / `min-height: {n}px` |
| `Sizing.width`, `.max_width`, `.height`, `.max_height` | (no class) | GTK's CSS has no `width`, `max-width` or `max-height` (session 22): the node goes in a clamp (`lgtk_clamp_*`, lugha-gtk 0.1.30, a C widget with one child) whose limits keep its size, never below the child's minimum; given more room than its maximum it gives the child the maximum, at the start. The parent holds the clamp, so `.expand` and `.align` go on it; the node fills it. Gaining or losing the clamp rebuilds the node (`Replace`); new numbers reset the limits |
| `Spacing.gap` | (no class) | not CSS: `gtk_box_set_spacing` on a column or row, `gtk_flow_box_set_row_spacing` and `set_column_spacing` on a wrap (a flow box reads no CSS gap) |
| `Outline` | `kv-ow-{n}`, `kv-os-{line}`, `kv-oc-…`, `kv-oo-{n}` / `kv-oo-m{n}` | `outline-width`, `-style`, `-color`, `-offset`, each under `:not(:focus-visible)` so the theme's focus ring (an outline) still shows |

`background-image` is one CSS property, so the gradient and the image are one class: two classes never set it differently. A picture's rule needs its bytes: `paint` (pure) names the class, and the runtime reads a file and checks the bytes (`pictures.size_of`) the first time the class is seen, then embeds them as a `data:` URL. GTK never decodes unchecked bytes or opens a path from CSS. A picture that can't be shown gives `background-image: none` (or the gradient alone). Icon names and font names go inside CSS strings and are refused when they hold `"`, `\\`, `;`, braces or line breaks. Checked against GTK 4.22's CSS parser (no parsing errors, session 19).

The rules live in one app stylesheet at application priority (600), created when the window opens; each render adds the rules of values not seen before (rules are never removed). A style's classes also go on the label kivuli puts in a button or a check box, because CSS doesn't inherit every text property (`text-decoration` isn't). A font name goes inside a CSS string, so one holding `"`, `\`, `;`, braces or a line break is refused. kivuli removes only its own `kv-` classes from a widget.

### Layout (`.expand`, `.align`; room is `Spacing` and size is `Sizing`, below)

| kivuli | GTK |
|--------|-----|
| `.expand()` | `gtk_widget_set_hexpand` / `vexpand(true)` (along the parent's axis) |
| `.align(Start\|Center\|End)` | `gtk_widget_set_halign` / `valign` (GTK_ALIGN_START/CENTER/END) |

Units are pixels. All numbers come from the app. kivuli has **no default spacing or padding of its own**: a fresh `column` looks exactly like a fresh `GtkBox`.

### Icons and pictures (`Image`, `icon`, `image`, `.image`, `.fit`; PRPs 005, 010)

| kivuli | GTK |
| --- | --- |
| `icon(name)`, `image(Image.Icon(name))` | a `GtkImage` showing the theme's icon of that name; `.size(n)` sets its pixel size, else the theme's (`-1`) |
| `image(Image.File(path))`, `image(Image.Bytes(key, bytes))` | a `GtkPicture` showing a `GdkTexture` decoded from the bytes (a file is read first); `.fit` → `content-fit`; `.size(n)` an n × n size request |
| `.image(…)` / `.icon(name)` on a button | kivuli's own content: an image before a label, each hidden when empty; a texture or theme icon in its `GtkImage`, `.size(n)` its pixel size; classes `text-button` (label), `image-button` (image), both for both, as GTK sets them itself (its theme has no `image-text-button`) |

**Where images come from (DECISION-019's model B, session 11).** An image shows bytes it already has: a theme icon, a local file, or bytes in memory with a key. kivuli never fetches from the network; an app downloads with a non-blocking command (a later PRP) and shows the bytes. A view can't make kivuli open a connection.

**Hostile bytes.** Every file or bytes image is checked from its header in pure Lugha (`src/pictures.la`) before GTK decodes anything: PNG (signature, IHDR) or JPEG (SOI, the first start-of-frame), at most 4096 × 4096 px and 16 MB. A small file claiming a huge image never reaches the decoder. A decode error is freed and shows nothing; nothing panics. The diff compares an image's kind and name, path or key, never its bytes.

Icons are the theme's only. Prefer symbolic names (`…-symbolic`): they take the theme's text colour, so they follow light, dark and high-contrast themes. kivuli never recolours an icon or a picture, and ships no icon set: an app that wants another one (Font Awesome, Lucide …) brings it itself. A missing icon or file shows GTK's placeholder.

### Disabled

Disabled means no message (MEMORY decision 6), mapped to `gtk_widget_set_sensitive(false)`; the theme greys it out.

| kivuli | GTK |
| --- | --- |
| a button with `on_click == None` | the button insensitive |
| an input with no messages (`.disabled(true)`) | the entry insensitive: it still shows its value, the user can't type, and Enter sends nothing |
| a text area with no `on_change` | its editor (the text view) insensitive, not its scrolled window, so long text still scrolls |

---

## Component Rules

- A modifier that doesn't apply to a node is ignored silently (decision 5).
- A view's root carries `.window(title, w, h)`. Window size is the default size, not a fixed size.
- Correct: a look is a typed value on the node, through the one door.

      text(m.note).style([Text { color: Paint.Faded(0.55), ..Text.theme() }])

- Wrong: a look smuggled in any other way (a widget's CSS, a Pango attribute, a second modifier for the same property).

---

### Overlay (`overlay`, `.place`; PRP-013)

| kivuli | GTK |
| --- | --- |
| `overlay(base, layers)` | a `GtkOverlay`: the base as its child, each layer added on top with `add_overlay`, in order |
| `.place(across, down)` | the layer's `halign` / `valign` (a layer is styled with its own axis, so `.align` and `.expand` don't apply) |

The overlay takes the base's size (`measure-overlay` off). A `GtkOverlay` only adds on top, so replacing a middle layer takes it and those above off (holding the ones above alive) and adds them back in order. Click-outside-to-close is a full-size layer the app adds, not a feature.

### Buttons showing a view (`button_with`; PRP-015)

| kivuli | GTK |
| --- | --- |
| `button_with(content, msg)` | a `GtkButton` whose child is the content's widget (`gtk_button_set_child`), filling it (GTK's default for a button's child) |

Everything else is a plain button's: the theme's button look, hover and pressed states and focus ring; `clicked` connected once, reading the message from a `Ref`; the same shortcut controller; `.style` as on any node. New content of another kind replaces the button's child, not the button. Interactive content is refused before anything is built (the paint pass), since a GTK button inside a GTK button fights it for clicks and focus.

### Context menus (`.menu`; PRP-014)

| kivuli | GTK |
| --- | --- |
| `.menu(entries)` | a `GtkPopoverMenu` parented to the node's widget (no arrow), showing a `GMenu` |
| `item(label, msg)` | a menu item naming an action `kv.itemN` in a `GSimpleActionGroup` inserted on the node's widget as `kv`; the action's `activate` sends the message |
| `.disabled(true)` on an item | the action disabled: the theme greys the item out |
| `menu_separator()` | a new section of the `GMenu` (the theme draws the line) |
| opening | a `GtkGestureClick` on button 3 and a `GtkGestureLongPress` (pointing at the press), and a local `GtkShortcutController` for `Menu\|<Shift>F10` (pointing at the whole node) |

Everything is made once per widget: an update re-points the actions' messages (one `Ref`), enables or disables them, and rebuilds the `GMenu` only when labels or separators change. A menu taken away (`[]`) keeps its parts and opens nothing. The node's `destroy` unparents the popover, so both are freed together. A menu takes no place in a patch path.

### Wrapping grid (`wrap`, `keyed_wrap`; PRP-012)

| kivuli | GTK |
| --- | --- |
| `wrap`, `keyed_wrap` | a `GtkFlowBox`, selection `NONE`; each child inside a `GtkFlowBoxChild` GTK makes, set not focusable so Tab goes straight to what's inside |
| `Spacing.gap` | row and column spacing |
| `.equal()` | `homogeneous` |
| `.max_per_line(n)` | `max-children-per-line`; without it, 65535 (GTK's own default caps a line at 7) |

A keyed move takes a child out and puts it back at its new place (a flow box can't reorder), holding the widget alive meanwhile. "Selected" tiles are the app's model plus a look.

### Header bar (`.header`)

| kivuli | GTK |
| --- | --- |
| `.header(start, title, end)` on the root | a `GtkHeaderBar` set as the window's titlebar, `show-title-buttons` on: GTK draws the window's controls |
| each `start` / `end` view | packed into the bar itself (`pack_start` / `pack_end`), so the theme's spacing between them applies |
| `title` | the bar's title widget; `.expand()` lets it take the bar's spare width |

Dark mode: apps follow the desktop's preference. Forcing dark from an app is deferred (DECISION-017).

---

## Background work (`Cmd.task`, `Sub.receive`; PRP-011)

Lugha runs every green thread on the program's one OS thread, and GTK's main loop holds that thread inside one `extern` call (`g_application_run`). So kivuli hands Lugha's scheduler turns: while any task or `Sub.receive` is active, a GLib timeout (every 4 ms) calls `task.yield_now()`, which never blocks (it wakes due sleepers, polls sockets with `epoll_wait(…, 0)`, and runs ready threads one round, Lugha's preemption cutting a busy one off every 4,096 steps). It keeps yielding while yields find work, up to 2 ms of the 4 ms, then dispatches finished tasks' messages and received values on GTK's thread, one `update` at a time. With nothing in flight the source removes itself. An OS thread per task is impossible in Lugha (one OS thread; callbacks only on the thread that registered them).

Requests for the lugha session (not worked around): runtime hooks so GLib waits exactly instead of a timer (DECISION-021), and blocking calls off the one thread (DECISION-022).

---

## What Is Forbidden

- No hardcoded colours, fonts, or pixel values inside kivuli. Numbers reach GTK only from app-supplied modifier arguments and style values.
- No CSS stylesheet shipped by kivuli: the only one is generated from the app's own style values.
- No style applied any way but `.style([...])`: one door, so two ways can never disagree.
