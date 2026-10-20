# CHANGELOG — kivuli

Follows [Keep a Changelog](https://keepachangelog.com) format.
Updated at the end of every session when something is completed and merged.
Never deleted. Older entries are never modified.

---

## [Unreleased]

## [0.40.0] — 2026-10-20

### Added
- A native `progress(value)` (PRP-045), out of 100; `Part.Fill`; `kivuli.ui`'s `ui.progress(value)`. Needs lugha-gtk 0.1.39.

### Fixed
- Three test expectations that were wrong (found by the first full run since the folder refactor: 421 tests).

## [0.39.0] — 2026-10-20

### Added
- Tooltips in the palette's look (PRP-044): `Part.Tooltip` and `kivuli.ui`'s `Tooltip.look()`, given once on the root.

## [0.38.0] — 2026-10-20

### Added
- Check and radio menu items (PRP-043): `check_item(label, checked, on_toggle)`, `radio_group(selected, choices)`, for `.menu` and `.dropdown_menu`; `Part.MenuCheck`, `Part.MenuRadio`; their marks in `Menu.look()`. Needs lugha-gtk 0.1.38.

## [0.37.0] — 2026-10-20

### Added
- Dropdown menus (PRP-042): `.dropdown_menu(entries)` on any view; `menu_label`, `submenu`, `item(…).hint(keys)` (for `.menu` too); `Part.MenuBox`, `MenuItem`, `MenuLabel`, `MenuSeparator`, `MenuHint`; `kivuli.ui`'s `Menu.look()` and `ui.dropdown_menu(trigger, entries)`. Submenus open beside their item. `kivuli.testing.pick` finds items in submenus. Needs lugha-gtk 0.1.37.

## [0.36.0] — 2026-10-20

### Added
- `kivuli.ui`'s Tabs (PRP-041): `tab`, `tab_with`, `ui.tabs(triggers, active)`, `ui.tabs_line(triggers, active)`.

## [0.35.0] — 2026-10-20

### Added
- A native `switch(on, on_toggle)` (PRP-040), disableable; `kivuli.testing.flip(view, name)`.
- `kivuli.ui`'s switch: `ui.switch(on, on_toggle)`. Needs lugha-gtk 0.1.36.

## [0.34.0] — 2026-10-20

### Added
- `kivuli.ui`'s dropdown (PRP-039): `ui.dropdown(choices, selected, placeholder)`, `Dropdown.invalid()`; `Part.Trigger`, `Arrow`, `List`, `Item`; a `When` inside an `Inside` (the part's own state); `.placeholder(…)` on a drop-down.

### Fixed
- Three tests stale since earlier releases (missing new fields, handlers now `Option`s).

## [0.33.0] — 2026-10-20

### Added
- `kivuli.ui`'s Slider (PRP-038): `ui.slider(min, max, value, on_change)`; `Part.Trough`, `Part.Highlight`, `Part.Knob`.
- `.disabled(…)` on a slider: greyed, sends nothing; `kivuli.testing.slide` gives `Err(Lookup.Silent(…))`.

## [0.32.0] — 2026-10-20

### Added
- `.disabled(…)` on a check box (PRP-037), native or `ui.checkbox`: greyed, sends nothing; `kivuli.testing.toggle` gives `Err(Lookup.Silent(…))`.

## [0.31.0] — 2026-10-20

### Added
- Styling a widget's parts (PRP-036): `Inside(Part.Check, [...])`, inside `When` too; `Mark.icon(name, px)`.
- `kivuli.ui`'s Checkbox: `ui.checkbox(label, checked, on_toggle)`, its tick always drawn.

## [0.30.0] — 2026-10-20

### Added
- `kivuli.ui`'s Separator (PRP-035): `ui.separator()` and `separator_with(label)`.

## [0.29.0] — 2026-10-20

### Added
- `kivuli.ui`'s input group (PRP-034): `input_group(start, control, end)`, `group_input`, `group_text`, `group_button`, `group_button_with`, `InputGroup.invalid()`.

### Changed
- `examples/components.la` scrolls.

## [0.28.0] — 2026-10-20

### Added
- `kivuli.ui`'s Field (PRP-033): `field`, `field_horizontal`, `field_label`, `field_description`, `field_error`, `field_group`, `Field.invalid()`.

## [0.27.0] — 2026-10-20

### Added
- `kivuli.ui`'s Input (PRP-032): `ui.input(value, on_change)` and `Input.invalid()`; a ring while typed in.
- `State.FocusWithin` (`:focus-within`): focus anywhere inside the node, however it got there. Ranked after `Focus`, so `Press`, `Checked` and `Disabled` rules repeat their class once more.

## [0.26.0] — 2026-10-20

### Added
- `kivuli.ui`'s Card (PRP-031): `card([...])`, `card_header(title, description)`, `card_header_with(title, description, action)`, `card_content([...])`, `card_footer([...])`. `Radius.XLarge` and `Border.rounded_xl()` (the palette's radius plus 4).

## [0.25.0] — 2026-10-20

### Breaking
- Text starts at the start of its line by default (PRP-030); GTK centred it. Centred text needs `.style([Text.center()])`.

### Added
- `Text.start()`, `Text.center()`, `Text.end()` (`TextAlign`): where text sits in its label, wrapped lines included. Needs lugha-gtk 0.1.35.

## [0.24.0] — 2026-10-20

### Added
- `kivuli.ui`'s Button (PRP-029): `ui.button(label, msg)` and `ui.button_with(content, msg)`, the native button in the palette's colours; variants `Button.destructive()`, `outline()`, `secondary()`, `ghost()`, `link()`; sizes `Button.sm()`, `lg()`, `icon()`; a ring halo on focus, half opacity when disabled, colours easing 150 ms.
- `Token.X.alpha(x)` (`Paint.Alpha`): a palette colour at an opacity, anywhere a colour goes.

## [0.23.0] — 2026-10-20

### Added
- `kivuli.ui` (PRP-028), components after shadcn, starting with `badge(label)`, `badge_with(content)` (icons and text) and their variants `Badge.secondary()`, `outline()`, `destructive()`, given through `.style`. `Look` / `look([...])`: several style values as one, a component's variant or an app's own. `examples/components.la`.

## [0.22.0] — 2026-10-20

### Added
- Motion (PRP-027): a `Motion` style group: `Motion.colors(ms)`, `opacity`, `transform`, `shadow`, `all` (transitions by property group), `Motion.easing(Easing.Ease | Linear | In | Out)`, `Motion.delay(ms)`; entrances `Motion.fade_in(ms)` and `Motion.slide_in(Edge.Top | Bottom | Left | Right, px, ms)`, played when a widget is first shown; loops `Motion.spin()` and `Motion.pulse()`. When the system asks for less motion, eases and entrances are instant and loops keep going. Works inside `When`. `examples/motion.la`. Needs lugha-gtk 0.1.34.

## [0.21.0] — 2026-10-20

### Added
- Palettes and colour tokens (PRP-026): `Token` (shadcn's set: `Background`, `Foreground`, `Card`, `Popover`, `Primary`, `Secondary`, `Muted`, `Accent`, `Destructive`, each with a `…Foreground`, and `Border`, `Input`, `Ring`) as `Paint.Token`, a CSS variable; `.palette(Palette.zinc())` (also `slate`, `stone`, `gray`, `neutral`; `.with(token, light, dark)`, `.radius(px)`) on the root, light and dark by the system's scheme; `.scheme(Scheme.System | Light | Dark)` sets GTK's colour scheme too; `Radius.Small`/`Medium`/`Large` and `Border.rounded_sm()`/`md()`/`lg()` follow the palette's radius. Colour utilities (`Text.color`, `Background.color`, `Border.all`…, `Outline.ring`) take a `Color`, a `Token` or a `Paint`. Without a palette, tokens have zinc's values. `examples/palette.la`. Needs lugha-gtk 0.1.33 (its stylesheet follows GTK's colour scheme).

### Changed
- `src/` is organised in folders (`view/`, `styles/`, `core/`, `render/`; DECISION-026). Private module paths change (`kivuli.diff` is `kivuli.core.diff`); apps, which import `kivuli` and `kivuli.testing`, see no difference.

## [0.20.0] — 2026-10-20

### Added
- Style utilities (PRP-025): short static functions on each style group, each returning the full value it stands for: `Font.h1()`…`caption()`, `xs()`…`xxl()`, `size`, `scale`, the weights, `italic`, `sans`/`serif`/`mono`, `family`, `tabular`; `Text.upper`, `underline`, `tight`, `wide`, `dim`, `faded`, `color`…; `Background.color`, `tint`, `linear`, `radial`, `image`; `Border.all`, `top`…`left`, `rounded`, `corners`, `none`; `Outline.ring`, `offset`; `Spacing.padding`, `padding_x`/`_y`/`_top`…, the same for `margin`, `gap`; `Sizing.width`…`max_height`; `Shadow.sm`…`xl`; `Effects.opacity`, `blur`, `grayscale`, `rotate`, `scale`, `translate`. They mix with full values: the last value that sets a field wins.

### Changed
- The style groups live in a `src/styles/` folder (`fonts`, `boxes`, `drawing`), which `src/styles.la` imports back (DECISION-025, widened). No change for apps.

## [0.19.0] — 2026-10-20

### Changed
- Clicking away clears keyboard focus (PRP-024): a click anywhere that doesn't take focus itself (empty space, text, pictures) leaves the focused field, as on the web; GTK alone keeps it focused. Every app, nothing to write. Needs lugha-gtk 0.1.31.

## [0.18.0] — 2026-10-20

### Breaking
- The header bar is built and styled as a node (PRP-023): `.header(start, title, end)` becomes `.header(header_bar(start, title, end))`, and `header_bar(…).style([...])` styles the bar itself (its window controls stay GTK's). For Rapidkoi: wrap its `.header(…)` arguments in `header_bar(…)`.

## [0.17.0] — 2026-10-20

### Added
- Shadows and effects (PRP-022): `Shadow { boxes, text }` (lists of `Drop { x, y, blur, spread, color, inset }`; `Shadow.off()` removes the theme's) and `Effects { opacity, filters, transform }` (`Filter.Blur`, `Brightness`, `Contrast`, `Grayscale`, `HueRotate`, `Invert`, `Saturate`, `Sepia`; `Move.Translate`, `Rotate`, `Scale`, `ScaleXY`) in `.style([...])`, inside `When` too. `examples/borders.la` has a "Shadows and effects" section.

### Changed
- New style groups live in `src/style_groups.la`, which `src/styles.la` imports back: a second allowed import cycle (DECISION-025). No change for apps.

## [0.16.0] — 2026-10-20

### Added
- States (PRP-021): `When(State.Hover | Press | Focus | Disabled | Checked, [...])` in `.style([...])`: values that apply only while a node is in that state, merged over its own, through GTK's pseudo-classes. A fixed order decides between two states on at once (Hover < Focus < Press < Checked < Disabled). `examples/borders.la` has a "States" section.

## [0.15.0] — 2026-10-19

### Breaking
- Sizing is a style value (PRP-020): `Sizing { width, min_width, max_width, height, min_height, max_height }` in `.style([...])`. The `.width(n)` and `.height(n)` modifiers are gone: they were minimums, now written `min_width` / `min_height`.
- For Rapidkoi: `.width(n)` becomes `.style([Sizing { min_width: Width.Px(n), ..Sizing.theme() }])` and `.height(n)` `min_height`.

### Added
- Exact sizes and maximums, which GTK doesn't have: a node with one sits in a clamp (lugha-gtk 0.1.30's `lgtk_clamp`), never smaller than its content's minimum, so `max_width` on wrapping text is a readable column. Minimums are GTK's CSS. `examples/borders.la` shows them; resize the window.
- Needs lugha-gtk 0.1.30.

## [0.14.0] — 2026-10-19

### Breaking
- Spacing is a style value (PRP-019): `Spacing { padding, margin, gap }` in `.style([...])`. The `.padding(n)` and `.spacing(n)` modifiers are gone. `.padding(n)` was really a margin (room outside the node); `padding` is now inside the border, `margin` outside it.
- For Rapidkoi: `.padding(n)` becomes `.style([Spacing { margin: Edges { all: Width.Px(n), ..Edges.theme() }, ..Spacing.theme() }])` to keep today's room exactly (or `padding`, for room inside a border), and `.spacing(n)` becomes `Spacing { gap: Width.Px(n), ..Spacing.theme() }`.

### Added
- `Edges { all, top, right, bottom, left }` (a side's own value beats `all`, as for borders) and `Spacing`; a negative margin pulls a node toward its neighbour. `examples/borders.la` shows padding inside a border and margin outside it.

### Changed
- `src/styles.la`'s plain value types moved to `src/style_values.la`, and `src/sheets.la`'s border and outline CSS to `src/box_rules.la` (file size); no change for apps.

## [0.13.0] — 2026-10-19

### Added
- Borders and outlines (PRP-018): `Border { all, top, right, bottom, left, radius }` and `Outline { width, color, style, offset }` in `.style([...])`. Each side is a `Side { width, color, style }`; a side's own fields beat `all`, and a later value's `all` resets earlier sides. `Line` has every CSS style; a width with no line is drawn solid. `Radius.All(n)` or `Radius.Corners(…)`; a rounded node clips what's inside it again. An outline hides while its node has keyboard focus, so the theme's focus ring shows. `examples/borders.la`.

## [0.12.0] — 2026-10-19

### Added
- Backgrounds (PRP-017): `Background { color, gradient, image, fill }` in `.style([...])`, merged like every style value. A colour (`Paint`; `Faded(x)` is a tint that follows light and dark), a `Gradient.Linear(angle, stops)` or `Gradient.Radial(stops)`, and an image (`Image.File`, `Image.Bytes`, `Image.Icon`) filling the node (`Fill.Cover`, `Contain`, `Tile`, `Stretch`), stacked image over gradient over colour. Pictures are header-checked and reach GTK as bytes (`data:` URLs), never paths; bad ones show nothing. `examples/backgrounds.la`.

### Changed
- `src/typography.la` is `src/styles.la` (it holds every style value); no change for apps.

## [0.11.0] — 2026-10-19

### Breaking
- Styling has one door (PRP-016, DECISION-023): how a node looks is `.style([...])`, a list of typed values. Removed, with no replacement in this release: `.bold()`, `.dim()`, `.size(n)` on text (it stops the app with a message pointing to `Font`; `.size` stays for icons and pictures), `.button_style` and `ButtonStyle`, and named looks (`Surface`, `Border`, `Theme`, `theme(…)`, `.theme(…)`, `.header_style`). Backgrounds, borders, corners and button looks come back through `.style` in a later release.
- The old examples are gone; the new ones are hello, type, tasks, browser, controls, split and bench.
- For Rapidkoi: replace `.bold()` with `.style([Font { weight: Weight.Bold, ..Font.theme() }])`, `.dim()` with `.style([Text { color: Paint.Faded(0.55), ..Text.theme() }])`, `.size(n)` on text with `Font { size: Size.Pt(n), ..Font.theme() }`; remove `.button_style`, `.theme(…)`, `.header_style` and its look enum until backgrounds return.

### Added
- `Styling`, `Font` (`family`, `size`, `weight`, `style`, `tabular`), `Text` (`color`, `line_height`, `letter_spacing`, `decoration`, `transform`) and `Paint` (`Theme`, `Color`, `Faded`), each field's `Theme` value meaning "not set here". Bad values (a size or line height of 0, an opacity over 1, a font name that could break out of CSS) stop the app with a message.
- The app's stylesheet grows as new values appear while it runs. A style's classes also go on the label kivuli puts in a button or check box.
- `examples/type.la`: every value, each on a line naming it.

### Fixed
- Eleven example binaries committed by mistake in 0.9.0 are removed from the tree (they stay in history).

### Needs
- A lughac with the struct-update fix (DECISION-024): `S { f: e, ..base }` converts to an interface value where one is expected.

## [0.10.0] — 2026-10-18

### Added
- Clickable rows (PRP-015): `button_with(content, msg)` is a button showing any view, so a whole row (label, summary, arrow) is one click target with the theme's button looks, keyboard focus and Enter. Button modifiers work on it; content holding anything interactive stops the app with a message. Tests find it by its tooltip, `.name` or any text inside it. `examples/frame.la`'s Info panel has two rows.

### Changed
- Making buttons moved from `src/build.la` to `src/buttons.la` (file size); no change for apps.

## [0.9.0] — 2026-10-18

### Added
- Context menus (PRP-014): `.menu([item(label, msg), menu_separator(), …])` on any node, opened by a right-click, a long press or the Menu key (Shift+F10); `item(…).disabled(true)` greys an item out. `kivuli.testing.pick(view, owner, item)` finds an item's message without GTK. `examples/frame.la`'s tabs have Duplicate, Close and Close other tabs. Needs lugha-gtk 0.1.29.

### Changed
- The bodies of `.disabled`, `.image`/`.icon` and `.on_submit` moved from `src/views.la` to `src/mapping.la` (file size); no change for apps.

## [0.8.0] — 2026-10-18

### Added
- Overlay (PRP-013): `overlay(base, layers)` draws layers on top of a view; `.place(across, down)` puts a layer where it goes. Opening and closing layers never rebuilds the base. `examples/frame.la`'s Info button opens a panel over the page. Needs lugha-gtk 0.1.27.

### Fixed
- Replacing a view in a header bar's start or end list could drop a text view after it from the bar (its last reference went while it was repacked). Later views are held alive while repacked.

### Changed
- `src/patch.la`'s path helpers moved to `src/paths.la` (file size); no change for apps.

## [0.7.0] — 2026-10-18

### Added
- Wrapping grid (PRP-012): `wrap(children)` and `keyed_wrap(children)` lay children out in lines that re-flow as the window resizes; `.equal()`, `.max_per_line(n)`, `.spacing(n)`. `examples/frame.la`'s home shows app tiles in one. Needs lugha-gtk 0.1.26.

### Changed
- `src/diff.la`'s lazy resolution moved to `src/lazies.la`, and `src/build.la`'s drop-down and slider helpers to `src/controls.la` (file size); no change for apps.

## [0.6.0] — 2026-10-18

### Added
- Background work (PRP-011): `Cmd.task(work, done)` runs work on a green thread and sends its result as one message; `Sub.receive(chan, f)` turns a Lugha channel's values into messages. kivuli gives green threads turns while GTK runs, so downloads (`std.http`, `std.net`, `std.tls`) and sleeps don't hold the window, and Lugha computation shares it. File reads, DNS lookups, `std.crypto` and `std.process` still block (DECISION-022). `examples/download.la`.

### Changed
- `spawn`, channels and actors now work inside kivuli apps: before, green threads never ran while GTK's main loop did.

## [0.5.0] — 2026-10-18

### Added
- Images from any source (PRP-010): `Image.Icon(name) | Image.File(path) | Image.Bytes(key, bytes)`; `image(…)` shows any of them, `.image(…)` puts one on a button. PNG and JPEG, checked from their headers before GTK decodes (at most 4096 × 4096 px and 16 MB): hostile bytes show nothing and never crash. Bytes are known by their key. `.size(n)` sizes a button's image, and gives a picture an n × n box. `examples/gallery.la` shows a picture from bytes. Needs lugha-gtk 0.1.25.

### Changed
- **Breaking:** `image(path)` is now `image(Image.File(path))`. To migrate, wrap each path. Files are PNG or JPEG only now (an SVG file shows nothing).
- `View.Icon` and `View.Image(path)` are one `View.Image(Image)`; `View.Button` holds its image (`Style.icon` is gone); patches `SetIcon`/`SetFile` are `SetImage`, and a button's image is `SetButtonImage`. Apps using the builders and modifiers only need the `image(path)` change.
- `View.map`'s body lives in `src/mapping.la` (DECISION-018); no change for apps.

## [0.4.1] — 2026-10-17

### Fixed
- A button's content (its icon and label) is centered in the button, as GTK's own buttons do: a button wider or taller than its content (a `.width(n)`, one stretched across a column, a big tile) no longer shows it at the start. Natural-size buttons look as before.

## [0.4.0] — 2026-10-17

### Added
- Corner radius (PRP-009): `Surface.radius_top_left`, `radius_top_right`, `radius_bottom_right`, `radius_bottom_left`; `radius` still rounds all four. A rounded node's content is clipped to its curve. `examples/frame.la`: the rail on the right, the page curved only at its top right. Needs lugha-gtk 0.1.24.

### Changed
- `Surface` has four new fields: a `Surface { … }` literal listing every field adds them (with `..Surface.theme()`, nothing changes).
- A theme's stylesheet has every corner rule for each radius it uses (one `border-radius` rule and four corner rules), so any merge of looks finds its classes.

## [0.3.0] — 2026-10-17

### Added
- Header look (PRP-008): `.header_style([…])` on the root gives the header bar the app's looks; `Surface.flat` turns a node's theme edges off (shadow and border line). `examples/frame.la`'s header bar and rail share one flat look.

### Changed
- `Surface` has a new field, `flat`: a `Surface { … }` literal listing every field adds it (with `..Surface.theme()`, nothing changes).

## [0.2.0] — 2026-10-17

### Added
- App look (PRP-007): typed styles: an app's looks as an enum, each a `Surface` (`background`, `text`, `border`, `radius`), `.style([…])` on nodes (later looks win), `.theme(theme(looks, surface))` on the root; `hex`, `rgb`, `Border`, `Color`. kivuli generates and loads the CSS. `.maximized()` and `.header(start, title, end)` (a header bar as the titlebar) on the root. `examples/frame.la`. Needs lugha-gtk 0.1.23.

### Changed
- **Breaking:** `.style(ButtonStyle.X)` is now `.button_style(ButtonStyle.X)`; `.style` takes an app's looks. To migrate, rename each call.
- `src/paint.la`: a pure pass before every diff fills each node's classes from its looks.

## [0.1.1] — 2026-10-17

### Added
- Disabled text fields (PRP-006): `.disabled(true)` on `input` and `text_area` greys them out and stops their messages; the value still follows the model. `kivuli.testing.type_into` and `submit` return `Silent` for a disabled field; new `kivuli.testing.is_field_disabled(view, hint)`.

### Changed
- `View.Input` and `View.TextArea` carry their messages as an `Option` (`None`: disabled), and `Patch.SetOnChange` an `Option` of the handler. Apps using the `input` and `text_area` builders are unaffected.

## [0.1.0] — 2026-10-17

### Added
- Icons and images (PRP-005): `icon(name)` (theme icons, `.size(px)`), `.icon(name)` on buttons (with or without a label), `image(path)` with `.fit(Fit.Contain | Cover | ScaleDown | Fill)`; `examples/gallery.la`. Needs lugha-gtk 0.1.22.
- Choices (PRP-004): `dropdown(choices, selected)` and `slider(min, max, value, on_change)` with `.step(n)`; `kivuli.testing.choose` and `slide`; `examples/settings.la`. Needs lugha-gtk 0.1.21.
- Keyboard (PRP-003): `.shortcut("Ctrl+N")` on buttons, `.autofocus()`, `Cmd.focus(name)`, and `kivuli.testing.press(view, keys)`. The to-do app starts in its new-task field; Ctrl+N adds, Ctrl+Shift+Delete clears done tasks.
- Long text (PRP-002): `.wrap()`, `.wrap_at(chars)`, `.ellipsize()` on text and check box labels; `.selectable()` text; `.tooltip(hint)` on any node, which `kivuli.testing.click` also finds buttons by; `kivuli.testing.tooltip`. The to-do app cuts long titles and shows them on hover.
- `text_area(value, on_change)`: a multi-line text field that wraps, scrolls inside itself and keeps the cursor while you type (PRP-001); `examples/notes.la`.
- `.name(id)`: a name tests find a text field or text area by.
- `View.map(f)`, `Cmd.map(f)`, `Sub.map(f)`: components with their own messages inside a bigger app; `examples/counters.la`.

### Changed
- Every button shows kivuli's own content (an icon and a label) instead of GTK's label, with the same theme classes GTK would set: label-only buttons look as before.
- The style types (`Style`, `Align`, `ButtonStyle`, `Window`, `Fit`) live in `src/looks.la`; apps still get them from `import kivuli`.
- The diff collects patches in place: updates of long lists are about twice as fast (100 rows: 3–4 ms).
- `lazy` reuses a subtree only from the lazy at the same place (as in Elm), so two copies of a component with equal keys can't share one.
- `kivuli.testing`: `click`, `is_disabled`, `type_into`, `submit`, `toggle` and `texts` work out what a user's action would send, from the view alone, with no GTK.
- Text fields: `.placeholder(hint)` and `.on_submit(msg)` (Enter).
- `lazy(key, build)`: a subtree built and diffed only when its key changes.
- `application(App { id, init, update, view, subscriptions })`: apps with effects. `Cmd.none()`, `Cmd.after(ms, msg)`, `Cmd.quit()`, `Cmd.batch(cmds)`; `Sub.none()`, `Sub.every(ms, f)`, `Sub.batch(subs)`. Timers start and stop to match the subscriptions after every update.
- `.window(title, width, height)` on the root of `view`: the window's title follows the model.
- `examples/stopwatch.la`.
- `examples/todo.la`: add (Enter or button), tick, delete, filter, clear done; keyed lazy rows; its own tests with `kivuli.testing`.
- Buttons: `.style(ButtonStyle.Primary | Danger | Subtle | Normal)` and `.disabled(bool)`, which greys the button out and stops its clicks sending anything.
- Text modifiers: `.size(points)`, `.bold()`, `.dim()`.
- Layout modifiers on any node: `.padding(n)`, `.spacing(n)`, `.expand()`, `.align(Align.Start | Center | End)`, `.width(n)`, `.height(n)`; adding, changing or dropping one restyles the same widget.
- `row`, `input(value, on_change)`, `checkbox(label, checked, on_toggle)`, `spacer()`, `separator()`, `scroll(child)` and `keyed_column(children)`, whose rows are matched by key, so reordering, inserting or deleting moves, adds or removes just those widgets; spacers grow and separators cross along their column or row; typing and ticking keep the cursor and send no echo of kivuli's own changes.
- `import kivuli;` reaches the whole public API (the package entry, `src/lib.la`).
- `View` with `text`, `button` and `column`: a screen described as plain data.
- `sandbox`: runs an app from `init`, `update` and `view` in a GTK window (rebuilt on every message for now).
- Messages now patch only the widgets whose view changed, instead of rebuilding the window.
- `examples/counter.la` and `examples/raw.la` (the same counter on raw GTK, for comparison).
- Project context system: CLAUDE.md, MEMORY.md, CONTEXT.md, DECISIONS.md, PRP templates, code style and design docs.
