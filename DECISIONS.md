# DECISIONS.md — kivuli

Tracks architectural and design questions that are open, deferred, or resolved.

Rules:
- Every open decision blocks implementation of the code it affects.
- The AI must not implement anything that depends on an open decision.
- When a decision is resolved, move it to the RESOLVED section and record the outcome.
- Once resolved, copy the outcome to MEMORY.md as an architectural decision.

---

## OPEN — Requires human input before implementation

### DECISION-029 — lughac: the examples' tests can't run together

**Status:** open (a lughac bug to report; run one at a time meanwhile)
**Raised:** 2026-10-20 — Session 114

**Question:** `lughac test examples/tasks.la examples/browser.la examples/controls.la` stops with E0304 (`main` belongs in the main file) for browser.la and controls.la, then "error: these test files check alone but not together; please report it". Each file alone passes (8, 5, 3 tests). lughac asks for it to be reported, in the lugha repository under its own rules.

**Outcome so far:** the examples' tests run one file at a time (`for f in tasks browser controls; do lughac test examples/$f.la; done`), which runs the same tests. Revisit once lughac runs several main files' tests in one call.

### DECISION-028 — A container's spacing reaching its parts

**Status:** open (not blocking: deferred by the user, session 53)
**Raised:** 2026-10-20 — Session 53 (PRP-031, Card)

**Question:** the reference card's `size="sm"` shrinks its spacing from 16 to 12 px, and its header, content and footer follow because they read one CSS variable from the card. kivuli's style values have no variables, and a part is built before it's put in its card, so a card's size can't reach its parts. Options: a style value that sets a named spacing for everything inside (a CSS variable under the hood); the card rewriting its parts' spacing when they're kivuli.ui parts; sizes given to each part.

**Outcome so far:** the card has one size (16 px). Decide when the small card, or another component needing it, is next.

### DECISION-027 — lughac: a test in a subfolder can't import a shared test file

**Status:** open (needs a lughac change; not worked around)
**Raised:** 2026-10-20 — Session 32 (src/ in folders)

**Question:** the user asked for `test/` in folders mirroring `src/`. A test file imports another by bare name, resolved from its own folder only: `test/render/live_images.la` with `import fixtures;` looks for `test/render/fixtures.la`, and `test.fixtures`, `kivuli.test.fixtures` and `super.fixtures` are all E0312. Five tests in three folders share `test/fixtures.la` (images as bytes).

**Outcome so far:** `test/` stays flat; `src/` is in folders (DECISION-026). Once lughac resolves a test program's imports from the package's test root too (or gives a way to name it), `test/` moves into `view/`, `styles/`, `core/` and `render/`.

### DECISION-026 — src/ in folders

**Status:** resolved
**Raised:** 2026-10-20 — Session 32
**Resolved:** 2026-10-20 — Session 32 (the user approved the layout)

**Outcome:** `src/` is grouped by folder, and a module's path follows its folder (`kivuli.core.diff`): `lib.la`, `testing.la` and `styles.la` at the root; `view/` (what apps build trees with), `styles/` (style groups and their CSS), `core/` (the pure runtime), `render/` (the only GTK callers: architecture rule 4 is "only `render/`"). No folder is named `gtk` (it would load from lugha-gtk), and no two modules share a last name (an import binds it). Apps see no difference.

### DECISION-024 — lughac: a struct update literal isn't converted to an interface value

**Status:** resolved — fixed in lughac (the user, between sessions 17 and 18; the reproduction prints `4 1 9 16`)
**Raised:** 2026-10-19 — Session 17

**Question:** Where an interface type is expected, lughac 0.1.0 converts a plain struct literal and a call's result into an interface value, but not a struct literal with `..base` (E0403, "expected Shape, found Sq"). PRP-016's whole style API is that shape: `.style([Font { weight: Weight.Bold, ..Font.theme() }])`.

```
interface Shape { fun area(self): i64; }
struct Sq { side: i64, tag: i64 }
impl Sq {
    fun base(): Sq = Sq { side: 1, tag: 0 };
    fun area(self): i64 = self.side * self.side;
}
fun main() {
    let plain: Shape = Sq { side: 2, tag: 0 };          // ok
    let call: Shape = Sq.base();                        // ok
    let updated: Shape = Sq { side: 3, ..Sq.base() };   // E0403: expected Shape, found Sq
    let listed: Shape[] = [Sq { side: 4, ..Sq.base() }];   // E0403
}
```

**Needed:** the same conversion for `S { f: e, ..base }` as for `S { … }` (milestone 93's struct update, with v5's interface values). Fixed in `../lugha` with its own PR; kivuli waits (no workaround: apps would have to bind every value to a typed `let` first).

### DECISION-023 — Styling has one door

**Status:** resolved for fonts and text (PRP-016, session 18) backgrounds (PRP-017, session 19) borders and outlines (PRP-018, session 20) spacing (PRP-019, session 21: `.padding` and `.spacing` moved into `Spacing`) and sizing (PRP-020, session 22: `.width` and `.height` moved into `Sizing`, exact and maximum sizes through a clamp in lugha-gtk) states (PRP-021, session 23: `When`) and shadows and effects (PRP-022, session 25); button looks and theme colours in `Paint` were considered and skipped (session 24, the user's call: apps build their looks from their own values)
**Raised:** 2026-10-19 — Session 17 (the user)

**Question:** Appearance comes through three overlapping ways: modifiers (`.bold`, `.dim`, `.size` on text, `.button_style`), `ButtonStyle`, and `Surface` fields (`text`, `flat`) that compete with them. Which one wins depends on the order kivuli applies them.

**Direction (the user's choice, breaking changes accepted):** modifiers keep layout, behaviour and text flow only; all appearance goes through `.style(looks)`. kivuli ships a `Role` enum of looks the GTK theme draws (its own classes: theme-following, no `.theme` needed); app looks are `Surface`s that can build on roles (`roles: […]`) and use theme colours. `.bold`, `.dim`, `.size` on text, `.button_style`, `ButtonStyle`, `Surface.flat` and `.header_style` go. A role on a node it doesn't fit stops the app with a message.

**Open details, for the PRP's interview:** the exact `Role` list (checked against GTK 4.22's default theme); theme and adaptive colours (DECISION-017); how `.style` mixes roles and app looks; the migration of examples, README and Rapidkoi.

**Outcome:** `.style([...])` of typed values implementing `Styling` (`Font`, `Text`, with `Paint`), merged field by field (the last value that sets a field wins; `Theme` sets nothing), with no registry and no utilities yet (their names would clash with the fields, E0310). `.bold`, `.dim`, text `.size`, `.button_style`/`ButtonStyle` and named looks (`Surface`, `Theme`, `.theme`, `.header_style`) are deleted. Breaking, in 0.11.0.

---

## DEFERRED — Acknowledged, not yet needed

### DECISION-021 — lughac: runtime hooks for a GUI's main loop

**Status:** deferred (a lughac request, reported; kivuli polls meanwhile)
**Raised:** 2026-10-18 — Session 12 (PRP-011)
**Revisit when:** the lugha runtime exposes them; then kivuli waits exactly instead of a 4 ms timer.

**Question:** kivuli drives Lugha's scheduler from a GLib timeout calling `task.yield_now()` every 4 ms while work is in flight, and guesses from how long a yield took whether more is ready. Hooks would make it exact: the runtime's `epoll` descriptor (GLib watches it), the next sleeper's deadline (GLib's timeout), and whether any green thread is ready.

**Notes:** Costs today: a result can arrive up to a tick late, and the 2 ms budget per tick is a heuristic (`NOTHING_RAN_US`). Read for reference in `../lugha/runtime/spawn.c`, `tasks.c`, `net.c`.

---

### DECISION-022 — lughac: blocking calls off the one OS thread

**Status:** deferred (a lughac request, reported; documented as a limit)
**Raised:** 2026-10-18 — Session 12 (PRP-011)
**Revisit when:** the lugha runtime can run a blocking call without holding every green thread (e.g. a worker pool that parks only the calling green thread).

**Question:** Reading files (`read_bytes` and the like), DNS lookups (at a connection's start), `std.crypto` (OpenSSL) and `std.process` each hold Lugha's one OS thread until they return, so inside a kivuli app they pause the window even in a `Cmd.task`. Rapidkoi's brief named reading files and hashing as background work.

---

### DECISION-020 — lughac: std.fs can't tell a file's size

**Status:** deferred (a lughac wish, reported; not worked around)
**Raised:** 2026-10-18 — Session 11 (PRP-010)
**Revisit when:** lughac's `std.fs` gains a size (`stat`); then `textures.la` checks a file's size before reading it.

**Question:** `Image.File` reads the whole file with `read_bytes` before kivuli can refuse one over 16 MB, so a large local file is read into memory first. A `std.fs.size(path)` (or `stat`) would let kivuli refuse it without reading it.

**Notes:** It's the app's own local file, and the bytes are still refused before GTK decodes them, so it's a memory spike, not a hole.

---

### DECISION-017 — How an app asks for GTK's dark variant (GTK 4.22)

**Status:** deferred (the user's choice, session 6: ship PRP-007 without `.dark()`)
**Raised:** 2026-10-17 — Session 6 (PRP-007 build)
**Revisit when:** a small PRP of its own, after reading how GTK 4.22 picks its theme's variant (its source), or testing on a desktop that prefers light.

**Question:** `gtk-application-prefer-dark-theme` is deprecated since GTK 4.20 ("use GtkCssProvider properties instead"); its replacement `gtk-interface-color-scheme` is documented as the system's preference. In tests here (desktop `prefer-dark`, theme `Yaru-dark`), neither setting, nor reloading the theme after it, nor waiting main-loop turns, changed the colours of GTK's own theme for new widgets; setting `gtk-theme-name` did take effect at once. How does an app force dark in GTK 4.22?

**Lead (session 8):** GTK's Default theme chooses its dark stylesheet with `@media (prefers-color-scheme: dark)` inside `gtk.css`, so the variant is decided by the colour-scheme preference of the provider that loads the theme (`GtkCssProvider:prefers-color-scheme`, GTK 4.20), not by a `GtkSettings` property an app sets.

**Notes:** kivuli apps already follow the desktop's dark preference (Rapidkoi is dark on the user's machine with no code). Rejected for now: loading GTK's built-in `Default-dark.css` over the theme, which is always dark but replaces the desktop's look (Yaru) with GTK's Default.

---

### DECISION-013 — A file dialog

**Status:** deferred
**Raised:** 2026-10-17 — Session 6 (rapidkoi DECISION-020, item 4)
**Revisit when:** after PRP-007; Rapidkoi's "Open a file…" uses a path field meanwhile.

**Question:** How does an app ask the user for a file? Likely a command (`GtkFileDialog` is asynchronous), which overlaps DECISION-007 (non-blocking I/O).

---

### DECISION-015 — Context menus on right-click

**Status:** resolved
**Raised:** 2026-10-17 — Session 6 (rapidkoi DECISION-020, item 6)
**Resolved:** 2026-10-18 — Session 15 (PRP-014, the user's choices)

**Question:** How does a node get a right-click menu (a popover), and how are its items' messages sent?

**Outcome:** `.menu([item(label, msg), menu_separator(), …])` on any node, items disabled with `.disabled(true)`; opened by right-click, long press and the Menu key / Shift+F10; a `GtkPopoverMenu` over a `GMenu` whose items are actions reading their messages from a `Ref`; tests use `kivuli.testing.pick(view, owner, item)`.

---

### DECISION-011 — lughac: a module's private imports can be imported through it

**Status:** deferred (a lughac bug, for the lugha repository; kivuli doesn't rely on it)
**Raised:** 2026-10-17 — Session 3 (PRP-005)
**Revisit when:** lughac fixes it; then nothing in kivuli should change.

**Question:** After `Style`, `plain` and `Fit` moved from `views.la` to `looks.la` (which `views.la` imports without `public`), `import kivuli.views { Style, plain };` and `import kivuli.views { Fit };` still compile in `test/`. A plain `import` should not re-export: only `public import` should.

**Notes:** Reproduced with a one-line test importing `Fit` through `kivuli.views` (passed). kivuli's tests now import from `kivuli.looks`, where the items live, so a fix breaks nothing here. Not worked around: reported to the user for the lugha session.

---

### DECISION-005 — Multiple windows and dialogs

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 2
**Revisit when:** an example or a user app needs a second window or a dialog.

**Question:** How does `view` describe more than one window?

**Notes:** v1 is one window by decision.

---

### DECISION-006 — Framework-owned local widget state

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 7
**Revisit when:** a real widget can't be built without it (a v2 dropdown, for example).

**Question:** Should some widgets keep state outside the app's `Model`?

**Notes:** Elm manages without it. Adding it weakens "the view is a shadow of the model".

---

### DECISION-007 — Non-blocking I/O commands

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 6
**Revisit when:** a blocking `update` is a real problem in an app, or lugha's green threads/actors (v17) are stable enough to drive from the GLib loop.

**Question:** Should file/process/network I/O become `Cmd`s?

**Notes:** v1 lets `update` block.

---

## RESOLVED

### DECISION-025 — A second allowed import cycle: styles ↔ styles/ (was style_groups)

**Status:** resolved
**Raised:** 2026-10-20 — Session 25 (PRP-022)
**Resolved:** 2026-10-20 — Session 25 (the user approved PRP-022)

**Question:** `src/styles.la` is near 500 lines, and every style group must sit with `Sheet`: `Sheet` holds each group, and each group's `apply` takes a `Sheet`. Moving groups out makes the two modules import each other, which architecture rule 2 forbids (one exception: `views` ↔ `mapping`).

**Outcome:** a second exception, `styles` ↔ `style_groups`: new style groups (from `Shadow` and `Effects` on) live in `src/style_groups.la`, which imports `styles` for `Sheet` while `styles` imports it for `Sheet`'s fields. No other cycle.

**Widened (session 31, PRP-025, the user's choice of folders):** every style group now lives in the `src/styles/` folder (`styles.fonts`: `Font`, `Text`; `styles.boxes`: `Background`, `Border`, `Outline`, `Spacing`, `Sizing`; `styles.drawing`: `Shadow`, `Effects`; `style_groups.la` is gone), so the cycle is `styles` ↔ the files in `styles/`. `src/styles.la` keeps `Styling`, `Sheet`, `applied`, `State`, `When`.

### DECISION-014 — An overlay

**Status:** resolved
**Raised:** 2026-10-17 — Session 6 (rapidkoi DECISION-020)
**Resolved:** 2026-10-18 — Session 14 (PRP-013, the user's choices)

**Question:** How does a view put one subtree over another (`GtkOverlay`), and how are the layers diffed?

**Outcome:** `overlay(base, layers)`, layers by position (later on top), placed with `.place(across, down)` using `Align`; no built-in click-outside (the app adds a full-size layer).

---

### DECISION-016 — A wrapping grid

**Status:** resolved
**Raised:** 2026-10-17 — Session 6 (rapidkoi DECISION-020)
**Resolved:** 2026-10-18 — Session 13 (PRP-012, the user's choices)

**Question:** A container whose children wrap onto more lines as the window narrows (`GtkFlowBox`), keyed like `keyed_column`?

**Outcome:** `wrap` (positional) and `keyed_wrap` (keyed), natural widths with `.equal()`, `.max_per_line(n)` only, no selection (focus goes straight to what's inside).

---

### DECISION-019 — Where web images come from

**Status:** resolved
**Raised:** 2026-10-18 — Session 11 (the "images from bytes" brief)
**Resolved:** 2026-10-18 — Session 11 (the user: model B)

**Question:** Rapidkoi will need images from the web as well as bundle bytes and files. Does kivuli fetch (`Image.Url`, an HTTP stack inside kivuli), or does the app?

**Outcome:** the app fetches, with a non-blocking command (a later PRP, with DECISION-007), and shows the bytes: `image(Image.Bytes(url, bytes))`. kivuli's images only ever show bytes they already have.

**Rationale:** the view stays pure and testable; loading and failure are explicit in the model; no HTTP stack in a UI library; for Rapidkoi, a guest app can't make the shell open a connection by putting a URL in its view: the shell decides.

**Copied to MEMORY.md:** yes (decision 17).

---

### DECISION-018 — Where View's modifiers go once views.la is full

**Status:** resolved
**Resolved:** 2026-10-18 — Session 11 (PRP-010)
**Raised:** 2026-10-17 — Session 8 (PRP-008)

**Question:** `src/views.la` reaches about 499 lines with PRP-008. Lugha attaches an `impl` only to a type in the same file, so the modifiers can't move out. Options: (A) move `View.map`'s body (about 60 lines) to a new module whose free function the method calls: `views` and that module would import each other (Lugha allows cycles; CLAUDE.md rule 2 says imports run one way), so it needs an explicit exception; (B) move the `View` enum and its `impl` into their own file and keep the rest of `views.la`; little else is left in it, so this barely helps; (C) raise the file size rule for `views.la` alone.

**Outcome:** (A), the user's choice: `View.map`'s body moves to `src/mapping.la`; `views` and `mapping` import each other, the one written exception to CLAUDE.md's architecture rule 2.

**Copied to MEMORY.md:** with PRP-010's merge.

---

### DECISION-012 — How apps style their frame

**Status:** resolved
**Raised:** 2026-10-17 — Session 5 (the "app look" brief)
**Resolved:** 2026-10-17 — Sessions 5–6

**Question:** The brief asked for an app stylesheet (a CSS string) and `.class(name)`. Plain CSS, per-widget colour modifiers (Flutter-like), or typed styles (StyleX-like)? And where do window-wide settings go?

**Outcome:** typed styles: an app names its looks in its own enum, each a typed `Surface`; nodes take `.style([…])`, merged left to right (later wins) in kivuli's code; kivuli generates atomic CSS. The button variant is renamed `.button_style(ButtonStyle.…)` (breaking, 0.2.0). Theme, dark, maximized and the header go on the root view next to `.window(…)`.

**Rationale:** checked by the compiler and testable without GTK; apps can't write selectors into kivuli's or GTK's internals (PRP-005 changed button internals); colours in one place, as Rapidkoi's `themes.la` already is; the header must be in the view anyway, and a theme in the view makes runtime switching a small later step.

**Copied to MEMORY.md:** with PRP-007's merge.

---

### DECISION-003 — Module path of the testing helpers

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §7
**Resolved:** 2026-10-17 — Session 3

**Question:** How do app tests reach the testing helpers?

**Outcome:** `src/testing.la`, re-exported whole from the entry (`public import testing;` in `lib.la`): `import kivuli;` gives `kivuli.testing.click(…)`, and `import kivuli.testing { click };` works too. Text fields are found by placeholder, so `.placeholder(s)` and `.on_submit(msg)` (decision 3) are built first.

**Rationale:** matches decision 10's `ui.testing`, renamed with the entry. The helpers are pure, and lughac builds only what's used.

**Copied to MEMORY.md:** yes

### DECISION-002 — The shape of `lazy` (risk R1)

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §6
**Resolved:** 2026-10-17 — Session 3

**Question:** `lazy` must recognise an unchanged input next render, but `View` is generic only over `Msg`, so it can't hold the input. How is the input compared?

**Outcome:** a string key supplied by the app: `lazy(key: string, build: fun(): View<Msg>)`. Before each diff, kivuli resolves the new view against the last one: a lazy reuses what the lazy **at the same place** built last update, if its key is the same, without calling `build` (Elm's rule). Places are matched as the diff matches them: by position, and by row key in a `keyed_column`. The diff skips two lazies with equal keys entirely. Modifiers go inside `build`; modifiers on `lazy(…)` itself are ignored.

**Amended (session 3, the user's choice):** at first, built subtrees were cached globally by key. With `View.map`, two copies of one component could share a key and reuse each other's subtree, handlers and all (a silent wrong message). Reuse is now per place, so keys only need to differ from the last update at the same spot.

**Rationale:** works with no language support, and it's the brief's preferred "simplest that works". The cost: a key that leaves out something the subtree depends on shows a stale subtree. Rejected: comparing the input with `==` (needs a type-erased comparable value Lugha may not have), and leaving `lazy` out of v1.

**Copied to MEMORY.md:** yes

### DECISION-010 — Modifiers wait for lughac: the expected type doesn't reach a method's receiver

**Status:** resolved
**Raised:** 2026-10-17 — Session 3 (step 5, part 1)
**Resolved:** 2026-10-17 — Session 3

**Question:** `fun v(): View<Msg> = text("x").padding(8);` was E0412: a generic call whose parameter appears only in its result couldn't be a method's receiver.

**Outcome:** fixed in lughac (lugha PR #48, 4e0b028): a method returning its receiver's own type passes the expected type down to the receiver, also inside lists, tuples, generic calls and `==`. Every kivuli modifier returns `View<Msg>`, so chains like `text("Total").padding(8).align(Align.Center)` check inside `column([...])`.

**Rationale:** the API stays as designed (decision 5). Limit to remember: only methods returning exactly the receiver's type (same parameters, same order) pass it down. A modifier returning anything else (say `Option<View<Msg>>`) wouldn't.

**Copied to MEMORY.md:** no (a lughac fix, not a kivuli decision). The limit is noted in CLAUDE.md.

### DECISION-009 — How a node carries its modifiers

**Status:** resolved
**Raised:** 2026-10-17 — Session 3 (step 5)
**Resolved:** 2026-10-17 — Session 3

**Question:** Does every variant get its own node struct with a layout field (the brief's sketch), or do modifiers live in one wrapper?

**Outcome:** one wrapper, `View.Styled(View<Msg>, Style)`, holding every modifier in one `Style` record. Chained modifiers merge into one `Style` (struct update from the current one, or from `plain()`), never nest. The diff splits each side into its inside and its style: same kind of inside gives `SetStyle(path, style)` when the styles differ, plus the inside's own patches, at the same path. A different kind gives `Replace` with the styled node. `live` applies the whole style on create and on `SetStyle`, using only the fields that fit the widget. `disabled` still goes through `Button`'s `Option<Msg>` (decision 6).

**Rationale:** no existing variant, diff arm or test changes, and new widgets get every modifier for free. Rejected: a record per node (type-precise, but every variant, arm and test changes, and every widget repeats the layout field).

**Copied to MEMORY.md:** yes

### DECISION-008 — How tests compare views once they hold functions

**Status:** resolved
**Raised:** 2026-10-17 — Session 2 (adding `input`)
**Resolved:** 2026-10-17 — Session 2

**Question:** `input`'s node holds `on_change: fun(string): Msg`. Functions have no `==`, so `View` (and `Patch`, which holds views) can't be compared with `assert_eq` (E0404). How do tests compare them?

**Outcome:** compare printed forms with a helper, `assert_same(actual, expected)` = `assert_eq("{actual}", "{expected}")`. Functions print as `<fun>`, so structure is checked fully. Which function a node holds is checked separately, by calling it (`f("x")` gives `Msg.Named("x")`).

**Rationale:** smallest change. The existing tests keep their shape, and the runtime stays as designed. Rejected: keeping `View` comparable through handler ids in a side table (a runtime redesign), and a hand-written `describe()` format (more code to keep in step with `View`).

**Copied to MEMORY.md:** yes

### DECISION-001 — Wait for the Lugha prerequisites, or start kivuli v1 now?

**Status:** resolved
**Raised:** 2026-10-17 — Session 1
**Resolved:** 2026-10-17 — Session 2

**Question:** Build struct update syntax and variants-as-functions in lugha first, or start kivuli without them?

**Outcome:** moot. Both landed in lugha before kivuli needed them: milestone 93 struct update (PR #32, tag m93) and milestone 94 variants as functions (PR #33, tag m94). Verified on a `lughac` reinstalled from lugha `main` at e7abc33.

**Rationale:** the brief's order (prerequisites first) held.

**Copied to MEMORY.md:** yes

### DECISION-004 — Where the extra GTK externs live long-term

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §3
**Resolved:** 2026-10-17 — Session 2

**Question:** Keep kivuli's extra externs private, or contribute them to lugha-gtk?

**Outcome:** contribute them to lugha-gtk's `gtk.raw`. The user directed it. `gtk_box_remove` and `gtk_box_insert_child_after` were pushed to lugha-gtk `main` (1d20cae). Tagged `v0.1.1` (6303b64); kivuli uses them from `gtk.raw` and `src/gtk_extra.la` is gone.

**Rationale:** one set of GTK bindings, not two.

**Copied to MEMORY.md:** yes
