# Design System — kivuli

kivuli draws nothing. Every visual comes from the user's GTK theme (Adwaita by default). This file maps kivuli's style vocabulary onto GTK, and it is the only place those mappings are decided.

---

## Tokens: kivuli vocabulary → GTK

### Styles (`.style([...])`; PRPs 016–022, DECISION-023)

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
| `Shadow.boxes` / `.text` | `kv-bsh-{css in hex}` / `kv-tsh-…` | `box-shadow: x y blur spread colour[ inset], …` / `text-shadow: x y blur colour, …`, one rule per list; `Shadow.off()` is one transparent drop |
| `Effects` | `kv-op-{x·1000}`, `kv-flt-…`, `kv-tf-…` | `opacity`, `filter: …` and `transform: …`, each list one rule; a transform is drawing only (no layout change) |
| `When(state, …)` | each of its rules' classes with `kv-{hover\|focus\|press\|checked\|disabled}-` | the rule under `:hover`, `:focus-visible`, `:active`, `:checked`, `:disabled`, its class repeated to the state's rank (1 to 5) so a later state in that order wins whatever the stylesheet's order; an outline in `Focus` drops `:not(:focus-visible)` (it's the app's focus ring there). Parsed by GTK 4.22 (C probe) |
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
| `image(Image.File(path))`, `image(Image.Bytes(key, bytes))` | a `GtkPicture` showing a `GdkTexture` decoded from the bytes, or GTK 4.22's own `GtkSvg` for an SVG (a file is read first); `.fit` → `content-fit`; `.size(n)` an n × n size request |
| `.image(…)` / `.icon(name)` on a button | kivuli's own content: an image before a label, each hidden when empty; a texture or theme icon in its `GtkImage`, `.size(n)` its pixel size; classes `text-button` (label), `image-button` (image), both for both, as GTK sets them itself (its theme has no `image-text-button`) |

**Where images come from (DECISION-019's model B, session 11).** An image shows bytes it already has: a theme icon, a local file, or bytes in memory with a key. kivuli never fetches from the network; an app downloads with a non-blocking command (a later PRP) and shows the bytes. A view can't make kivuli open a connection.

**Hostile bytes.** Every file or bytes image is checked from its header in pure Lugha (`src/view/pictures.la`) before GTK decodes anything: PNG (signature, IHDR) or JPEG (SOI, the first start-of-frame), at most 4096 × 4096 px and 16 MB. A small file claiming a huge image never reaches the decoder. A decode error is freed and shows nothing; nothing panics. **SVG** (PRP-053): told apart by its start (`<svg`, or `<?xml` then `<svg` in 1 KB, after a byte-order mark and white space); drawn by `GtkSvg` with only system resources (fonts) on: no other files or URLs, no animations, no GTK extensions, set before the bytes are read (lugha-gtk's `lgtk_svg_load`); refused before GTK sees it when it holds an `<image` element, prefixed or not, comments too (GTK 4.22.4 segfaults on one with outside files off: DECISION-030); shows nothing on a syntax error, a size of none or over 4096 px. Never a background: GTK's CSS can't turn outside files off, so SVG bytes there fail the check and an SVG file shows nothing. The diff compares an image's kind and name, path or key, never its bytes.

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

### Header bar (`.header(header_bar(…))`; PRP-023)

| kivuli | GTK |
| --- | --- |
| `.header(header_bar(start, title, end))` on the root | a `GtkHeaderBar` set as the window's titlebar, `show-title-buttons` on: GTK draws the window's controls |
| each `start` / `end` view | packed into the bar itself (`pack_start` / `pack_end`), so the theme's spacing between them applies |
| `title` | the bar's title widget; `.expand()` lets it take the bar's spare width |

**Text alignment** (PRP-030): GTK's CSS has no `text-align`, so `Text.align` adds no class: `styling` sets the label's `xalign` (0, 0.5, 1) and `justify` (left, center, right) on every restyle of a text node. kivuli's default is the start (GTK's is the middle). Labels kivuli makes inside native buttons and check boxes keep GTK's placement.

**Components** (`kivuli.ui`, PRP-028): built from style values only, so they add no CSS of their own: their classes are the values' (`kv-bgt-primary`, `kv-br-…-sm`). A variant is a `Look` whose items apply in order, so it merges field by field like the values it holds. Badge: padding 2 × 8, `Radius.Small`, a 1 px border (transparent unless outlined, so every variant is one size), `Font.xs()`, medium, `Primary` on `PrimaryForeground`; `.align(Align.Start)`, so it's as wide as its text in a column.

**Parts** (PRP-036): `Inside(part, …)` merges into `Sheet.parts`; each rule becomes a descendant rule, `.kv-{node}-{value} {node}` (`Part.Check` is GTK's `check` node). Under a state the pseudo-class sits on the widget, before the node (`.kv-checked-check-…:checked check`); in the focus state only an outline's `:not(:focus-visible)` goes, the node stays. Paint takes a rule's class before a space too. A part can't hold a `When` or an `Inside`, a background image, an exact or maximum size, or a gap. `Mark.icon(name, px)` is `-gtk-icon-source: -gtk-icontheme("name"); -gtk-icon-size: Npx`. **Checkbox**: the box 16 × 16, `Radius.All(4)`, an `Input` border, 8 px from the label; checked: `Primary` fill and border, `object-select-symbolic` at 14 px in `PrimaryForeground`; the focus ring on the box.

**Menus** (PRP-042): popover menus are made nested (`gtk_popover_menu_new_from_model_full`, `NESTED`), so a submenu opens beside its item. A `menu_label` starts a titled section; an empty untitled section is never added (it would draw a stray line). A hinted item is a `GMenuItem` with an `accel` string (kivuli's key form, through `keys.parse`). Items are numbered depth first, submenus' included. `.dropdown_menu` is `.menu` plus `Style.menu_on_click`, which styling hands to the popup: a primary-click gesture in the capture phase opens the menu below the node and claims the sequence, so a button doesn't get the click. `Menu.look()`'s parts: `popover.menu > contents`, `modelbutton`, `label.title`, `separator`, `accelerator`; a GMenu item can't carry a class, so there's no destructive item.

**Dialog** (PRP-047): one window (anti-pattern 9), so a dialog is drawn in it: `.dialog(d)` makes the root an overlay over the page whether `d` is `Some` or `None` (its style marks it `dialog_root`; paint refuses it below the root), so opening adds layers and the page keeps its widgets. While open the page is `.inert` (`gtk_widget_set_can_target` and `can_focus` off: GTK applies them to descendants). The layers: a backdrop (a `button_with` with no look, black at 50%, `.focusable(false)`, a click sends the close message) and the card, centred. `ui.dialog`'s card: `Background`, a `Border` border, `Radius.Large`, padding 24, `Shadow.lg()`, 480 to 512 px (a clamp), fading in; its ✕ a ghost icon button placed at the top end with `.shortcut("Escape")`. `.focusable` is set on buttons only (others aren't focusable by default). **Sheet** (PRP-052): the card places itself (`ui.dialog`'s `.place(Center, Center)`; `.dialog`'s layer takes it as placed). `ui.sheet`'s card is placed against its edge (`Start`/`End` across and `Fill` down on a side, 384 px both ways of its width, a clamp; `Fill` across on the top or bottom); its panel `Background`, a 1 px `Border` border on the edge facing the page only, no corners, padding 24, `Shadow.lg()`, sliding in over 300 ms from 384 px (a side) or 64 px (top, bottom: its height isn't known); on a side, the parts before the last are a column that expands, so a footer sits at the bottom.

**Avatar** (`kivuli.ui`, PRP-046): an overlay whose base is the initials (`Muted`, centred) and whose layer is the picture (`Fit.Cover`); the overlay is an exact 32 × 32 (the clamp) and `Radius.All(999)`, which clips the picture round. A picture that can't be shown shows nothing, so the fallback needs no logic. `avatar_group` wraps each avatar in a plain column for its −8 px overlap (a negative margin inside the clamp would stretch the picture) and rings each with a 2 px `Background` shadow.

**Progress** (PRP-045): `progress(value)` is a `GtkProgressBar` (`View.Progress`, `Live.Progress`), its fraction `value / 100` clamped to 0..1, patched with `SetProgress`; it sends nothing. GTK sizes the fill itself, so no CSS transition can ease it. `ui.progress`: `Part.Trough` (the bar's `trough`) 8 px, round, `Primary` at 20%; `Part.Fill` (`progress`) round `Primary`.

**Tooltips** (PRP-044): GTK shows a tooltip in its own window hung off the widget's surface: for the main window, a sibling after the root view (and the title bar). `Part.Tooltip` is `~ tooltip`, so it works from the root's style only. `Tooltip.look()`: `Foreground` behind `Background` text, `Font.xs()`, `Radius.Medium`, 6 × 12 padding, the theme's border and shadow off, a 150 ms fade (no zoom); GTK still times and places it.

**Check and radio menu items** (PRP-043): a check item names a stateful action `kv.check{n}` (a boolean), a radio group one `kv.radio{n}` (a string) that its items name with their label as the target; made once by their place and kept, their states set from the view on every refresh, so what's ticked follows the model. An `activate` handler is connected, so GTK doesn't flip a state itself. Plain items stay `kv.item{n}`, numbered among plain items only. `Menu.look()` draws the marks (the theme may draw none): `popover.menu check:checked` a 16 px `object-select-symbolic`, `radio:checked` an 8 px `media-record-symbolic`.

**Tabs** (`kivuli.ui`, PRP-041): no GTK notebook or stack: which tab is open is the model's (rule 7). `ui.tabs` styles each trigger (a native button) itself, as a row's style can't reach its buttons (DECISION-028): triggers `MutedForeground`, no background (restated in hover and press), the active one `Background` on `Foreground` with a small shadow; the list a `Muted` pill, `Radius.Large`, 3 px padding. `ui.tabs_line`: every trigger a 2 px bottom border, transparent but the active one's `Foreground`, so all are one height. `ui.tabs_vertical` and `ui.tabs_line_vertical` (PRP-048) stack the triggers in a column, each filling it with `Text.start()` (on a native button, the halign of the box holding its image and label: GTK's centre unless start or end), a `tab_with`'s row `Align.Start`; the line variant's line is a 2 px right border.

**Switch** (PRP-040): `switch(on, on_toggle)` is a `GtkSwitch` (`View.Switch`, `Live.Switch`), set before its `notify::active` handler is connected, the handler reading a `Ref<Option<…>>` (disabled: insensitive, nothing sent); patched with `SetChecked` and `SetOnToggle` like a check box. `ui.switch`: the switch node is the track (32 × 18, round, `Input`, `Primary` while `:checked`), `Part.Knob` (GTK's `slider` inside it) the thumb (16 px, round, `Background`, `PrimaryForeground` while on); GTK slides the thumb.

**Dropdown** (`kivuli.ui`, PRP-039): a drop-down's parts are paths, as bare nodes match too much (the trigger holds a `row`, the list's scrollbars have `trough`/`slider`): `Part.Trigger` is `> button`, `Arrow` `arrow`, `List` `popover > contents`, `Item` `popover row` (the pop-up is a CSS descendant of the drop-down). A part's class carries its code (`kv-item-…`). A `When` inside an `Inside` puts the state on the part (`popover row:hover`), one class enough there. The trigger: the input's look, the ring on keyboard focus, muted text while the placeholder (the blank item, labelled by `.placeholder`) shows; the list: `Popover`, a `Border` border, medium corners and shadow, 4 px padding; items 6 × 8 (32 at the end for GTK's tick), `Radius.Small`, `Accent` on hover. **Groups** (PRP-049): the model is a `GtkFlattenListModel` over one `GtkStringList` per group (lugha-gtk's `GtkGroups`, `src/c/groups.c`), so positions count through every group and GTK gives each group a section; a header factory shows a `header` before each section: a `GtkSeparator` (hidden in the first) over the heading's label (hidden when `""`). `Part.GroupHeading` is `popover listview > header`, `Part.GroupLine` `… > header separator`; `ui.dropdown_grouped`'s headings `Font.xs()`, `MutedForeground`, 6 × 8, the line `Border`. The blank item, when it shows, is a group of its own with no heading. A plain drop-down has no header factory. **Disabled** (PRP-050): a choice's message is an `Option`; `.disabled(true)` makes every one `None` (rule 6), and the drop-down is insensitive while no choice sends (set on every sync, on the same widget); `ui.dropdown` fades to 50% under `:disabled`.

**Slider** (`kivuli.ui`, PRP-038; its number PRP-051: `.show_value(on)` is `gtk_scale_set_draw_value` on every restyle, a slider without a style drawing it as built; `ui.slider` hides it): `Part.Trough`, `Highlight` and `Knob` are GTK's `trough`, `highlight` and `slider` nodes. The track: 6 px, `Radius.All(999)`, `Muted`; the fill `Primary`; the thumb 16 px round, a `Primary` border on `Background`, a 1 px shadow, a −5 px margin so it's centred on the track; a 4 px `Ring`-at-50% halo on the thumb on hover (the whole slider's) and focus. Disabled sliders take their handler away (`Option`), as check boxes do.

**Separator** (`kivuli.ui`, PRP-035): the native `GtkSeparator` (oriented by its parent) with a `Border` background and 1 px minimums both ways. `separator_with`'s lines are 1 px-high boxes, not separators: a separator inside its row would run down.

**Input group** (`kivuli.ui`, PRP-034): a row carrying the field's look (the `Input` border, the faint shadow, the `FocusWithin` ring), so the ring shows while the field or a button inside has focus. `group_input` is a `ui.input` with `Border.none()` and `Shadow.off()` in every state and a 34 px minimum, inside the 1 px border. Add-ons: a row per side, `MutedForeground`, 12 px from the edge, icons 16 px; the control pads 8 beside add-ons. Group buttons: ghost, 24 px high, 8 across, `Radius.Small`; a lone icon is a 24 px square.

**Field** (`kivuli.ui`, PRP-033): a column (or `field_horizontal`'s row, its parts centred) 12 px apart; groups 24 px apart. The label sets no colour, so `Field.invalid()` (`Destructive` text on the field) reaches it by inheritance, while the description keeps its own `MutedForeground` and the control its own colours.

**Input** (`kivuli.ui`, PRP-032): the native entry with style values. Its focus look uses `State.FocusWithin` (`:focus-within`, ranked after `Focus`): a GTK entry's typing area (`text`) takes the focus, so `:focus-visible` never matches the entry. The ring: a `Ring` border, a 3 px `Ring`-at-50% halo, GTK's outline off. GTK's `entry:disabled` outranks a plain class, so the disabled border, background and text are restated in `When(Disabled)`. The placeholder and selection are the theme's (sub-nodes kivuli's class rules don't reach).

**Card** (`kivuli.ui`, PRP-031): a column in `Card` on `CardForeground`, a 1 px border of `Foreground` at 10%, `Radius.XLarge` (`calc(var(--kv-radius) + 4px)`), padding 16 top and bottom only, 16 between parts. The parts pad themselves 16 across; the footer's −16 bottom margin cancels the card's bottom padding so its muted band (`Muted` at 50%, a `Border` top) meets the edge, and the card's rounded corners clip it. The card sets no font size: a `Size.Scale` is a percentage of the parent's, so it would compound inside. Title `Font.lg()` medium; description `Font.sm()` in `MutedForeground`.

**Button** (`kivuli.ui`, PRP-029): kivuli's native button with style values on it. GTK's own `button:hover`, `:active`, `:disabled` rules (an element and a pseudo-class) outrank a plain class, so each variant restates its colours in `When(Hover)`, `When(Press)` and `When(Disabled)` (a class and a pseudo-class, repeated by rank), which outrank GTK's. Focus: a `Ring` border and a 3 px `Ring`-at-50% halo (a spread box shadow), with GTK's focus outline set to none in the focus state. Disabled: the resting colours at opacity 0.5. Colours and shadows ease 150 ms. `Token.X.alpha(x)` is `color-mix(in srgb, var(--kv-x) N%, transparent)`, classes `kv-…t-x-a{permille}`.

**Motion** (PRP-027): a node's transitions are one class, `kv-tr-{hex}` → `transition: prop ms curve delay, …` (each group's properties: colours are `color`, `background-color`, the four `border-*-color`s and `outline-color`); its animations one class, `kv-an-{hex}` → `animation: …` (entrances `kv-in-fade` and `kv-in-{edge}-{px}`, `both` fill, eased `Out` by default; loops `kv-spin` 1 s linear and `kv-pulse` 2 s, infinite). `rule_text` writes each motion rule whole: an animation's `@keyframes` before it, and a `@media (prefers-reduced-motion: reduce)` block after (`transition: none`; the animation list with only its loops, or `none`). The app's stylesheet follows `gtk-interface-reduced-motion` (lugha-gtk 0.1.34), as it follows the colour scheme. An entrance plays when GTK first styles a widget; a restyle keeps a running animation, so a kept widget doesn't replay.

**Palettes and tokens** (PRP-026): a `Token` is a `Paint` written `var(--kv-{name})`; its classes are `kv-fgt-…`, `kv-bgt-…`, `kv-{b|o}ct…-…`. Every window carries zinc's variables (`window { --kv-…; }`, the dark ones under `@media (prefers-color-scheme: dark)`), the first rule of the app's stylesheet. A root with `.palette(p)` carries `kv-pal-{hash}`: `p`'s variables, `--kv-radius`, and the page's `background-color: var(--kv-background)` and `color: var(--kv-foreground)`, its dark values in a `prefers-color-scheme` block; its rule comes before every other, so the root's own `.style` wins. A header bar carries `kv-pal-{hash}-bar`: the variables only, so the bar keeps GTK's colours. `Radius.Small`/`Medium`/`Large` are `calc(var(--kv-radius) - 4px)`, `- 2px` and `var(--kv-radius)`. `.scheme(…)` sets GTK's `gtk-interface-color-scheme` (2 dark, 3 light), which GTK's widgets and `prefers-color-scheme` follow; `System` resets it to the platform's own (writing "default" would hide the system's choice). Set only when the root's scheme changes. The app's stylesheet is made with `lgtk_css_provider_new_for_scheme`: an app's own provider otherwise keeps `prefers-color-scheme` at "default", and its dark rules never apply.

**Style utilities** (PRP-025): `Font.bold()`, `Spacing.padding(16)`, `Shadow.md()` and the rest are static functions on each group that return an ordinary full value; they add no classes or rules of their own. `Shadow.sm()`…`xl()` are Tailwind's layered black shadows (5% and 10% black, negative spreads).

**The bar's style** (`header_bar(…).style([...])`, PRP-023): a `Bar` is not a view node; its style rides in the root's `Style` (`bar`, apart from the page's `sheet`), and `paint` gives it its own classes (`bar_classes`), which go on the `headerbar` node only. Its window controls are GTK's and stay so (which show, their order and side from the desktop's layout, what they do); a bar's style reaches them through four parts (PRP-054): `Part.WindowButton` (`windowcontrols > button`), `Part.WindowClose` (`… > button.close`), `Part.WindowIcon` (`… > button > image`) and `Part.WindowCloseIcon` (`… > button.close > image`: the close one's circle alone, session 126), refused on a view node. The themes (Default, Yaru) draw the hover circle on the icon (`button:hover > image`) and clear the button's background on hover, so a look sets the button and clears the icon (kivuli's provider outranks the theme's in every state). `WindowControls.look()`: 32 × 32, `MutedForeground`, round (`Border.rounded(99)`, the user's change), `Accent` on hover and press (restated), the close `Destructive` on hover, the icon's circle cleared and its box 24 × 24 (the themes' is 30), `Motion.colors(150)`. GTK draws the bar's own background, bottom line and shadow: an app replacing them sets those three (`Background`, `Border { bottom: … }`, `Shadow.off()`). An exact or maximum size, a gap and a background image are refused on a bar.

Dark mode: apps follow the desktop's preference. Forcing dark from an app is deferred (DECISION-017).

---

## Keyboard focus: clicking away (PRP-024)

GTK keeps a window's focused widget whatever is clicked beside it; kivuli clears it, as a web page blurs a field. When the window opens, kivuli adds a primary-button `GtkGestureClick` to the window in the bubble phase (GTK's default), so it sees only clicks no widget under the pointer took (buttons, fields, text areas, check boxes, drop-downs, sliders and selectable text take their own). On a press it calls `gtk_window_set_focus(window, NULL)` (lugha-gtk 0.1.31). Tab, `Cmd.focus` and `.autofocus` are untouched; context menus' right-click gestures are on their own nodes.

## Background work (`Cmd.task`, `Sub.receive`; PRP-011)

Lugha runs every green thread on the program's one OS thread, and GTK's main loop holds that thread inside one `extern` call (`g_application_run`). So kivuli hands Lugha's scheduler turns: while any task or `Sub.receive` is active, a GLib timeout (every 4 ms) calls `task.yield_now()`, which never blocks (it wakes due sleepers, polls sockets with `epoll_wait(…, 0)`, and runs ready threads one round, Lugha's preemption cutting a busy one off every 4,096 steps). It keeps yielding while yields find work, up to 2 ms of the 4 ms, then dispatches finished tasks' messages and received values on GTK's thread, one `update` at a time. With nothing in flight the source removes itself. An OS thread per task is impossible in Lugha (one OS thread; callbacks only on the thread that registered them).

Requests for the lugha session (not worked around): runtime hooks so GLib waits exactly instead of a timer (DECISION-021), and blocking calls off the one thread (DECISION-022).

---

## What Is Forbidden

- No hardcoded colours, fonts, or pixel values inside kivuli. Numbers reach GTK only from app-supplied modifier arguments and style values.
- No CSS stylesheet shipped by kivuli: the only one is generated from the app's own style values.
- No style applied any way but `.style([...])`: one door, so two ways can never disagree.
