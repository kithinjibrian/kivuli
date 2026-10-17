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

**Decision:** a button's `on_click` is `Option<Msg>`. `.disabled(true)` sets it to `None`, which greys the widget out.

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

**Decision:** `lazy(key: string, build)` skips building and diffing a subtree whose key is unchanged (DECISION-002, session 3: the app supplies a key covering every input; subtrees cached by key). `keyed_column` handles reordering. No virtualised list in v1.

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

## CURRENT PROJECT STATE

### Fully Working
- Context system (session 1).
- `sandbox(id, init, update, view)` with diff-and-patch rendering (session 2).
- Widgets: `text`, `button`, `column`, `row`, `input`, `checkbox`, `spacer`, `separator`, `scroll`, `keyed_column` (session 2).
- 41 tests (5 GTK, skipped without a display); `examples/counter.la`, `examples/raw.la`.
- Toolchain: lughac from lugha main (user-installed; m93–m100 in use), lugha-gtk 0.1.8, GTK 4.22.

### In Progress
- Nothing (session 2 closed after step 4).

### Not Started
- Step 5 modifiers and styles; step 6 `application` with Cmd/Sub; `lazy` (DECISION-002); testing helpers (DECISION-003); to-do and stopwatch examples; README; LICENSE (the user's).
- PRP-001: not written; steps 0–4 were built hands-on at the user's request. Whether to write it is the first question of session 3.

---

## NEXT SESSION START POINT

See the NEXT SESSION START POINT block in `CONTEXT.md`. It is rewritten every session.
