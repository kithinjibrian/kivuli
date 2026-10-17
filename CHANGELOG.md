# CHANGELOG — kivuli

Follows [Keep a Changelog](https://keepachangelog.com) format.
Updated at the end of every session when something is completed and merged.
Never deleted. Older entries are never modified.

---

## [Unreleased]

### Added
- `lazy(key, build)`: a subtree built and diffed only when its key changes.
- `application(App { id, init, update, view, subscriptions })`: apps with effects. `Cmd.none()`, `Cmd.after(ms, msg)`, `Cmd.quit()`, `Cmd.batch(cmds)`; `Sub.none()`, `Sub.every(ms, f)`, `Sub.batch(subs)`. Timers start and stop to match the subscriptions after every update.
- `.window(title, width, height)` on the root of `view`: the window's title follows the model.
- `examples/stopwatch.la`.
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
