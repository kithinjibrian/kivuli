# CHANGELOG — kivuli

Follows [Keep a Changelog](https://keepachangelog.com) format.
Updated at the end of every session when something is completed and merged.
Never deleted. Older entries are never modified.

---

## [Unreleased]

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
