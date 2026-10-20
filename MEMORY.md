# MEMORY.md — kivuli

Records resolved architectural decisions and current project state.
Read this at the start of every session before writing any code.

Decisions 1–12 were agreed in the design discussion in the lugha repository (after milestone 90) and recorded in the kickoff brief, `docs/source/stakeholder/kickoff-2026-10-16.md`. They are final.

---

## ARCHITECTURAL DECISIONS

### 1. The Elm architecture, not React/Flutter/Solid

**Decision:** Model → `update(model, msg)` → `view(model)` → diff → patch. The app never touches a widget.

**Why:** value semantics, exhaustive `match` on `Msg`, and a pure, testable `update`/`view` fit Lugha. This was the user's stated preference.

**Rules out:** framework comparisons (the user asked for no more), component-local mutable state, reactive signals.

---

### 2. Two entry points: `sandbox` and `application`

**Decision:** `kivuli.sandbox(id, init, update, view): i32` is pure. `kivuli.application(App { id, init: (Model, Cmd<Msg>), update: fun(Model, Msg): (Model, Cmd<Msg>), view, subscriptions: fun(Model): Sub<Msg> }): i32` adds effects.

**Why:** most apps need no effects, and Elm's `Browser.sandbox` / `Browser.element` split keeps them simple.

**Rules out:** a single entry point with optional effect fields.

---

### 3. One window per app in v1, configured from `view`

**Decision:** the root modifier `.window(title, width, height)` sets the window, so the title may depend on state.

**Rules out (v1):** multiple windows and dialogs.

---

### 4. The v1 widget set

**Decision:** `text`, `button(label, msg)`, `input(value, on_change)` (+ `.placeholder`, `.on_submit`), `checkbox(label, checked, on_toggle)`, `column`, `row`, `spacer`, `scroll`, `separator`, `keyed_column(children: (string, View<Msg>)[])`.

**Rules out (v1):** text area, dropdown, slider, image, table (these are v2).

---

### 5. Modifiers are chained methods on `View`

**Decision:** a modifier that doesn't apply to a node is ignored. Layout: `padding`, `spacing`, `expand`, `align(Start|Center|End)`, `width`, `height`. Text: `size`, `bold`, `dim`. Button: `style(Normal|Primary|Danger|Subtle)`, `disabled(bool)`. Root: `window`. Modifiers return changed copies (`match self { Button(b) => …, _ => self }`).

**Rules out:** modifier-argument structs, errors for inapplicable modifiers.

**How (session 3, DECISION-009):** modifiers don't change a node's own variant. They wrap it once in `Styled(inner, Style)`, and further modifiers update that `Style`. Inapplicable fields are stored and ignored when `live` applies them.

---

### 6. Disabled = no message

**Decision:** a button's `on_click` is `Option<Msg>`. `.disabled(true)` sets it to `None`, which greys the widget out. Text fields follow the same rule (PRP-006, session 5): `Input(value, Option<(on_change, on_submit)>)` and `TextArea(value, Option<on_change>)`, `None` when disabled; an input's edit and Enter messages travel together, so Enter can't send while edits can't.

**Rules out:** a separate `disabled` flag.

---

### 7. Effects as data, minimal

**Decision:** `Cmd.none()`, `Cmd.batch`, `Cmd.after(ms, msg)`, `Cmd.quit()`. `Sub.none()`, `Sub.every(ms, fun(i64): Msg)`, and `Sub.batch(subs)` (added in session 3, so an app can listen to more than one timer). File, process and network I/O are not commands: `update` may call `std.fs`/`std.process` directly, and blocking is accepted for now.

**Rules out (v1):** HTTP/file commands, async I/O.

**How (session 3):** `src/effects.la` holds the types as data (pure); `live.la` runs them. Subscription timers are keyed by interval and their count among equal intervals (`"1000#0"`): a key asked for again keeps its GLib timer, with its function re-pointed through a `Ref`; others are removed. `sandbox` is `application` with no effects (`sandbox_app`), so there is one runtime.

---

### 8. Composition by mapping, no framework-owned local state

**Decision:** `View<A>.map(f: fun(A): B): View<B>`, and the same on `Cmd` and `Sub`. Local state is added only if a real widget needs it.

---

### 9. Performance tools: `lazy` and `keyed_column`

**Decision:** `lazy(key: string, build)` skips building and diffing a subtree whose key is unchanged at the same place (DECISION-002, session 3: the app supplies a key covering every input; reuse is per place, as in Elm, so mapped components can't mix). `keyed_column` handles reordering. No virtualised list in v1.

---

### 10. `kivuli.testing` with no GTK

**Decision:** inspect a `View` tree headlessly: `testing.find_button(v, "Reset").is_disabled()`, `testing.click(v, "Add") == Some(Msg.Add)`, `testing.type_into(v, placeholder, "text")`. Apps test with `lughac test`. Module: `kivuli.testing`, re-exported from the entry (DECISION-003, session 3); fields are found by `.placeholder`.

---

### 11. Packaging

**Decision:** its own repository (GitHub `kithinjibrian/kivuli`, created and pushed only on the user's go), package `kivuli`, entry module `src/lib.la` (`import kivuli;`; originally a `ui` module, moved to the entry when lughac milestone 99 added `lib.la`, at the user's direction in session 2), depends on `gtk` (lugha-gtk) as a git package. Rendering and diffing are private. License GPL-3.0-only.

---

### 12. Order of work

**Decision:** two Lugha compiler features come first, in the lugha repo with their own PRPs: struct update syntax (`S { f: e, ..base }`) and variants as function values (`Msg.Named` as `fun(string): Msg`). No trailing-closure syntax. Then kivuli v1 in eight steps (CONTEXT.md, NEXT SESSION START POINT), then examples: counter, to-do, stopwatch.

**Done:** both landed as lugha milestones 93 (struct update, PR #32) and 94 (variants as functions, PR #33). Use them freely: `Model { count: m.count + 1, ..m }`, `input(m.name, Msg.Named)`, `xs.map(Some)`. Needs a `lughac` installed from lugha at m94 or later.

---

### 13. The handler-`Ref` trick

**Decision:** each live widget connects its GTK signal once, at creation. The handler reads the message from a `Ref<Option<Msg>>` (or `Ref<fun(string): Msg>` for `input`/`checkbox`) kept with the live widget. A patch only `set`s the `Ref`.

**Why:** messages can't be compared (they may hold closures), and reconnecting per render would pin closures without bound.

**Rules out:** disconnect/reconnect on every render, comparing messages.

---

### 14. Use `gtk.raw`; missing externs go into lugha-gtk

**Decision:** kivuli manages widgets through `gtk.raw`. A GTK function it needs that `gtk.raw` lacks is added to lugha-gtk (`../lugha-gtk/src/raw.la`), checked against the header, and pushed when the user says so (DECISION-004). Until a lugha-gtk tag carries it, kivuli declares it temporarily in `src/gtk_extra.la`.

**Why:** the user's direction (session 2): one set of bindings.

**How it's imported:** through lugha-gtk's entry, by name: `import gtk { GtkWidget, gtk_box_new, … };` (lugha-gtk 0.1.8's `public import raw { * };`, lughac milestone 100). Before that glob existed it was `import gtk;` then `gtk.raw.…` (0.1.4). The user chose this over moving kivuli onto the high-level wrappers (`gtk { Window, Box, … }`), which would have needed a lugha-gtk wrapper for every GTK feature first.

---

### 15. Tests compare views by their printed form

**Decision:** `View` and `Patch` aren't `Eq` once a node holds a function (from `input` on). Tests use `assert_same(a, b)` (`assert_eq("{a}", "{b}")`; functions print as `<fun>`), and check handlers by calling them (DECISION-008).

**Rules out:** `assert_eq` on views or patches; handler ids or a registry just to make views comparable.

---

### 16. Styling has one door: typed values (DECISION-023, PRP-016; replaced DECISION-012's named looks)

**Decision:** how a node looks is `.style([...])`, a list of typed values (`Font`, `Text`; more groups later) that each implement `Styling`. They merge into one `Sheet` per node: for each field the last value that sets it wins, and a `Theme` variant sets nothing. No names to register: a style is written on the node and reused through ordinary functions. kivuli turns each value into an atomic CSS class and adds its rule to the app's one stylesheet the first time it's seen. Modifiers are for layout, behaviour and text flow only. Window-wide settings (`.maximized`, `.header`) go on the root view, next to `.window`.

**Rules out:** a second way to set any look (`.bold`, `.dim`, `.button_style` are gone), named-look registries, app-written CSS or selectors, a cascade beyond what GTK's CSS inherits, colours shipped by kivuli.

---

### 17. Images show bytes they already have (session 11, PRP-010)

**Decision:** one `Image` type (`Icon(name)`, `File(path)`, `Bytes(key, bytes)`) for `image(…)` and a button's `.image(…)`. kivuli never fetches from the network: an app downloads with a non-blocking command and shows the bytes (model B, DECISION-019). Every file or bytes image is header-checked in pure Lugha (PNG, JPEG; 4096 × 4096 px; 16 MB) before GTK decodes it. The diff compares an image's kind and name, path or key, never its bytes. SVG (PRP-053) is drawn by GTK's own `GtkSvg` with outside files, animations and extensions off; an SVG holding `<image>` is refused before GTK sees it (DECISION-030: GTK 4.22.4 crashes on one); no SVG backgrounds.

**Rules out:** `Image.Url`, kivuli-side HTTP, decoding unchecked bytes.

### 18. View.map's body lives in mapping.la (DECISION-018)

**Decision:** modifiers stay in `views.la` with `View` (an `impl` must sit with its type); `map`'s body is in `src/mapping.la`, and `views` ↔ `mapping` is the one allowed import cycle.

---

### 19. Background work: kivuli hands Lugha's scheduler turns (PRP-011)

**Decision:** `Cmd.task(work, done)` (spawn + a channel) and `Sub.receive(chan, f)`. While either is in flight, a GLib timeout calls `task.yield_now()` (non-blocking) up to a budget each tick; results reach `update` on GTK's thread. `update` stays pure: it describes work, kivuli runs it.

**Rules out:** OS threads (Lugha has one), kivuli-side networking (decision 17), cancellation in v1. Blocking calls (files, DNS, `std.crypto`, `std.process`) still hold the window until Lugha changes (DECISION-022).

---

## CURRENT PROJECT STATE

### Fully Working
- `sandbox` and `application` (Cmd/Sub), diff-and-patch rendering, per-place `lazy`, `map` for components, `kivuli.testing`.
- Widgets: text, button, input (placeholder, on_submit), checkbox, column, row, keyed_column, scroll, spacer, separator; modifiers for layout, text, buttons and the window.
- Examples: hello, type (a specimen of every style value), tasks, browser and controls (each with tests), split, bench; README.
- Toolchain: lughac from lugha main (user-installed), lugha-gtk 0.1.29, GTK 4.22.

### Known limits
- Speed: 100-row keyed lists update in 3–4 ms; 1,000 rows take ~200 ms because Lugha's `Map`/`Set` inserts are quadratic (milestone 101; reported to the lugha session in session 3). Reversing a list is O(n²) in kivuli's keyed diff by design.

### Not Started
- Widgets for real apps (v2): text area, dropdown, slider, image/icons, label wrapping and tooltips, keyboard shortcuts, focus, big-list virtualisation; dialogs and multiple windows (DECISION-005); non-blocking I/O commands (DECISION-007).
- Publishing (GitHub repo, v0.1.0 tag) and LICENSE: the user's go.

---

## NEXT SESSION START POINT

See the NEXT SESSION START POINT block in `CONTEXT.md`. It is rewritten every session.
