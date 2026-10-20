# CLAUDE.md — kivuli

Behavioral instructions for AI coding assistants. Each rule exists to prevent a specific mistake.

kivuli is a declarative UI framework for Lugha (Elm architecture, rendered to GTK 4 through lugha-gtk). The design is settled in `MEMORY.md` and `docs/source/stakeholder/kickoff-2026-10-16.md`. Do not re-litigate it.

---

## SESSION HANDOFF RULE — NON-NEGOTIABLE

Every session in `CONTEXT.md` must have:
- A **name**: a short title for what the session did
- A **state**: `open` while work is in progress, `closed` once the handoff is done
- A **branch**: the git branch the session's work lives on

Format:

    ## SESSION {n} — {YYYY-MM-DD} — {Name} — {state}
    Branch: {branch-name}

Rules:
- **Step 1 of every session, no exceptions:** append a new session entry to `CONTEXT.md` with state `open` and the current branch name. Do this before reading any other file, before planning, before writing any code. Commit it before any other work begins.
- Mark it `closed` only after CONTEXT.md is updated and committed, and pushed once a remote exists. There is no remote until the user says to publish (see OUTWARD-FACING ACTIONS).
- Never leave a session `open` at the end of a turn.
- Never start a new session without closing the previous one first.
- The NEXT SESSION START POINT block is rewritten at the end of every session.
- Sessions are never deleted. The full history stays in CONTEXT.md.

---

## PRP RULE — NON-NEGOTIABLE

Never write code for a new feature without a PRP file in `/PRPs`.

If a feature request is given without a PRP:
1. Do not write any code.
2. Run a discovery interview (`PRPs/DISCOVERY.md`), one question at a time, **each question with a recommended answer attached** (the user's standing preference).
3. Write the PRP to `/PRPs/prp-{NNN}-{feature_name}.md` from `PRPs/TEMPLATE.md`.
   - `NNN` is the next unused number, zero-padded to three digits. Numbers are never reused or renumbered.
   - `feature_name` is lowercase snake_case: `prp-001-kivuli_v1.md`.
4. Present it to the user and build only after explicit approval.

When the user says "build all", build the approved PRPs in order and merge each once local checks pass, without stopping between them.

---

## SCOPE RULE — NON-NEGOTIABLE

One PRP at a time. Never implement more than one feature's scope in a single session.

If mid-implementation the scope turns out larger than the PRP described: stop, document what was discovered, update or create a PRP, and get approval before continuing. The human decides what is "small enough to add", not the model.

**A missing or broken lughac feature is never worked around silently.** Stop, record it in `DECISIONS.md`, and report it. It is fixed in the lugha repository (`../lugha`) with its own PRP or bug-fix PR, under that repo's `CLAUDE.md`. Never edit `../lugha` from a kivuli session. `../lugha-gtk` only as PROTECTED FILES allows.

---

## TESTING RULE — NON-NEGOTIABLE

Write the test before writing the implementation. No exceptions.

- For every new function, write a failing `@test` first, then the minimum code to make it pass.
- All tests live in `test/`, named after the `src/` module they cover: `src/core/diff.la` → `test/diff.la`. `test/` stays flat until lughac lets a test in a subfolder import a shared test file (DECISION-027). Files in `test/` see the package's package-level items (lughac milestone 97), so private modules are tested from there too. Keep test code out of `src/`.
- A named import binds the module's name as well (`import kivuli.render.live { … }` binds `live`), so don't name a local after the module you import from (E0311). Import a function under another name when it clashes: `import kivuli.core.diff { diff as patches };`.
- The full run is `lughac test test/` plus the examples' own tests (`lughac test examples/tasks.la examples/browser.la examples/controls.la`): before a merge, not after every step.
- Pure tests must need **no display and no GTK window**: view construction, modifiers, `map`, diff (tree → patches), `kivuli.testing`, subscription bookkeeping. Tests of the live GTK layer (`test/live.la`) drive a hidden window and read the widgets back, and must start with `if !has_display() { skip("no display"); }` so they are skipped, not failed, without one.
- Test behavior visible to callers: inputs, outputs, and every error variant. Do not test private internals or GTK's own behavior.
- Every new `public` function gets at least one happy-path test and one test per error variant it can return.
- While working, run only the test files that cover the change (`lughac test test/diff.la test/live_forms.la`), not the whole suite after every edit (the user's request, session 3). Likewise for examples: `lughac check` only the examples that use what changed, and `lughac build` only the one the user is about to run (the user's request, session 42). Run the full suite and build every example once, before a push.
- Opening real windows is the user's job. Never claim a visual behavior works because it compiled.

If the PRP does not describe what to test, add the test cases to the PRP before writing code.

---

## SECURITY RULE — NON-NEGOTIABLE

Never implement without explicit human review and approval:
- Authentication, authorization, or session management
- Cryptographic operations
- Secrets or credential handling
- Running external processes from library code (apps may call `std.process`; kivuli itself does not)

For everything else:
- Never hardcode credentials, API keys, tokens, or secrets, not even in comments or examples.
- Never trust external input (files, CLI args, env) in examples without validating it first.
- Never add a package dependency or a `[link]` library without stating why in the PRP. Every C library is attack surface.
- Every new `extern` declaration must match the C prototype in `/usr/include/gtk-4.0` (or the GLib headers) exactly. Check the header, do not guess. A wrong signature is memory corruption, not a type error.

If unsure whether something has a security implication, stop and ask.

---

## OUTWARD-FACING ACTIONS — NON-NEGOTIABLE

Ask before anything that leaves this machine: creating the GitHub repository, `git push`, tagging a release, opening PRs or issues in other repositories. Exception: lugha-gtk releases are pushed and tagged without asking (the user's standing go, session 3). Before any push or tag, check the test run's real exit status: `lughac test … | tail` hides a failure (lugha-gtk v0.1.19 shipped a failing test that way). The plan (`gh repo create kithinjibrian/kivuli --public`, push `main`, tag `v0.1.0`) runs only on the user's go.

---

## DESIGN RULE — NON-NEGOTIABLE

Read `docs/DESIGN.md` before writing any widget, modifier, or style code.

kivuli draws nothing itself. Every visual comes from the GTK theme through the CSS classes and properties mapped in `docs/DESIGN.md`. Never hardcode a colour, font, or pixel value in GTK CSS.

---

## CODE DOCUMENTATION RULE — NON-NEGOTIABLE

Read `docs/CODE_STYLE.md` before writing any function, type, or module.

Every `public` item has a `/** … */` doc comment (the language server shows them). Every non-obvious decision inside a function has a `//` comment explaining *why*, not what.

---

## PENDING DECISIONS RULE — NON-NEGOTIABLE

Before writing code that depends on an unresolved architectural question, check `DECISIONS.md`.

- `open`: stop. Do not implement. Ask the user to resolve it.
- `resolved`: follow the recorded outcome. Do not re-litigate.
- A new unresolved question mid-implementation: add it to `DECISIONS.md` as `open` and stop. Do not guess.

Never make an architectural choice silently.

---

## FILE SIZE RULE

No file exceeds 500 lines, tests included.

When a file reaches the limit: stop, propose a specific split (new file names and what moves where), wait for approval, split, then continue.

---

## PROTECTED FILES — NON-NEGOTIABLE

Never read, modify, or delete:

- `.env` and `.env.*`
- `lugha.lock`: written only by `lughac build`/`update`, never hand-edited
- `LICENSE`: GPL-3.0-only, changed only by the human
- Built executables (the paths in `.gitignore`)
- `../lugha/`: another repository with its own rules (read for reference; never write)
- `../lugha-gtk/`: write only to add GTK bindings kivuli needs, when the user asks (DECISION-004). Commit only your own change, never the user's uncommitted work there. Push it and tag the release once its checks pass, without asking: a standing go from the user (session 3)

If a task seems to require touching a protected file, stop and ask. See also `.llmignore`. `.claude/settings.json` denies Claude's Read and Edit tools on the same paths. Keep all three in sync.

---

## COMMANDS

```bash
lughac check examples/hello.la        # lex, parse, type-check (one entry point at a time)
lughac build examples/hello.la        # compile and link (needs gtk4 via pkg-config)
lughac run examples/hello.la          # build and run (opens a window, so it's the user's)
lughac test test/                     # every @test; GTK tests skip themselves without a display
lughac fmt --check src examples test  # formatting; drop --check to rewrite
lughac update examples/hello.la       # re-resolve git packages, rewrite lugha.lock
lughac explain E0312                  # what an error code means
lughac spec                           # the full Lugha language reference: check it before assuming a feature exists
```

A task is done only when `fmt --check` is clean, `lughac test test/` passes (skips allowed only for `no display`), and the examples that use what changed pass `lughac check`. Every file in `examples/` passes `lughac build` before a push (session 42: building them all after each change was too slow).

> Needs: `lughac` (the user installs it from `../lugha` with `cargo install --path . --locked`; never do it yourself), `libgtk-4-dev` (GTK 4.22), `pkgconf`, `libgc-dev`, `cc`.

---

## STACK

- **Lugha v16/v17** via `lughac` 0.1.0. v16 is frozen and complete. v17 is in progress in `../lugha` (91 green threads, 92 actors, 93 struct update `S { f: e, ..base }`, 94 variants as function values `Msg.Named`). A stale `lughac` rejects new syntax. **Never build or reinstall `lughac` yourself**: when kivuli needs a newer one, stop and ask the user to run `cd ../lugha && cargo install --path . --locked`, then **Developer: Reload Window** in VS Code (its running `lughac lsp` keeps the old binary). Use only what `lughac spec` documents.
- **lugha-gtk 0.1** (0.1.8 or later: the raw functions kivuli uses, and the entry re-exporting `gtk.raw`'s items by name; `gtk = { git = "https://github.com/kithinjibrian/lugha-gtk", version = "0.1" }`). kivuli uses **`gtk.raw`** only, because it manages widgets itself. It does not use `gtk.widgets`. Import it through the package entry by name: `import gtk { GtkWidget, gtk_box_new, … };`, then `gtk_box_new(…)` (lugha-gtk 0.1.8 re-exports `gtk.raw` with a glob, lughac milestone 100). Never `import gtk.raw;` or `gtk.raw.…` (the user's choice, session 2).
- **GTK 4.22** + GLib, linked through `[link] pkg-config = ["gtk4"]`.
- **Boehm GC** (through lughac): closures handed to GTK are pinned with `data`/`release` callbacks.
- License: **GPL-3.0-only**, like Lugha.

---

## ARCHITECTURE RULES

The package entry is `src/lib.la` (lughac milestone 99): apps write `import kivuli { View, text, sandbox };` or `import kivuli;` then `kivuli.sandbox(…)`. There is no `ui` module.

1. **Apps see only the package entry (`src/lib.la`, `import kivuli;`), `kivuli.ui` and `kivuli.testing`.** `views` is reached through the entry's re-export; `live`, `diff` and `effects` are private (package-level). *Why:* the public surface is the Elm contract, and leaking widgets lets apps bypass `update`/`view`.
2. **Private modules import `views`, not the entry `lib`; `build` never imports `live`.** Shared types go in their own module (`views` for `View`, `live_tree` for `Live`) so imports run one way. **The one exception:** `views` and `mapping` import each other, because the bodies of `View`'s node-rewriting methods (`map`, `.disabled`, `.image`, `.on_submit`, `.menu`, …) live in `mapping.la` (every modifier must stay in `views.la` with `View`; DECISION-018). **A second:** `styles` and the files in `styles/` import each other, because every style group's `apply` takes a `Sheet` and `Sheet` holds every group; groups go in `styles/` (DECISION-025). No other cycle. *Why:* the entry is the app-facing surface over `live`; private modules need only the view types, which keeps dependencies pointing one way.
3. **`diff` is pure.** It takes two `View` trees and returns patches. It never calls GTK. *Why:* it must be unit-testable with no display, and it is where most bugs will live.
4. **Only the files in `src/render/` call GTK/GLib** (`live`, `build`, `styling`, `patch`, `textures`, `controls`, `popovers`, `buttons`, `schemes`, `windows`). `build.la` makes and styles widgets (`buttons.la` makes its buttons); `popovers.la` gives a widget its context menu; `patch.la` applies patches; `textures.la` turns a checked file or bytes image into a texture; `live.la` runs the app and carries out commands and subscriptions (GLib timers). `core/effects.la` is pure data. *Why:* one folder to audit, few places to audit every extern call and every pinned closure.
5. **Connect each GTK signal once per widget, never per render.** The handler reads the message to send from a `Ref` stored with the live widget, and a patch only `set`s that `Ref`. *Why:* messages may hold closures and can't be compared, and reconnecting per render leaks pinned closures (risk R4).
6. **Disabled means no message.** A button's `on_click` is `Option<Msg>`, and `None` maps to `gtk_widget_set_sensitive(false)`. Do not add a separate `disabled` flag. *Why:* one source of truth, so the two can't disagree.
7. **The runtime holds state in one `Ref<Option<State<Model, Msg>>>` inside `Running` (`live.la`).** `dispatch(msg)` = update → view → diff → patch → store. Nothing else mutates the model. *Why:* that is the Elm guarantee the whole framework sells.
8. **Never overwrite an entry's text when it already equals the model's, and guard `changed` against echoes from programmatic `set_text`.** *Why:* otherwise the cursor jumps and typing loops (risk R5).
9. **A GTK function `gtk.raw` lacks is added to lugha-gtk**, checked against `/usr/include/gtk-4.0`. Until a tag carries it, kivuli may declare it temporarily in `src/gtk_extra.la`. *Why:* one set of bindings (DECISION-004).

---

## ERROR HANDLING — NON-NEGOTIABLE

Expected failures return `Result<T, E>` (Lugha's prelude). Programmer errors `panic`.

```
Expected failure  →  return Err(...)
Truly unexpected  →  panic("...")   (an invariant broke: a live tree that doesn't mirror the view tree)
```

```
// Good: the caller sees the failure in the type
public fun find_button(v: View<Msg>, label: string): Result<ButtonRef<Msg>, FindError> { … }

// Bad: hides "not found" and "found two"
public fun find_button(v: View<Msg>, label: string): ButtonRef<Msg> { … }
```

- Each module that can fail defines its own error enum. Errors are typed, never a bare `string`.
- **Never** use `Option` to mean an error. `None` can't say why. (`Option<Msg>` for "no message" is a value, not an error.)
- **Never** `unwrap`/`expect` on anything derived from app input. `expect("reason")` is only for true invariants, and its message states the invariant.
- The public app API (`sandbox`, `application`, widgets, modifiers) does not return `Result`. A malformed view is a programmer error and panics with a message naming the node. `ui.testing` returns `Result`, because "button not found" is an expected test outcome.
- GTK/GLib failures at startup (no display, application failed to register) end the program with GTK's exit status from `main`. They are not caught.

---

## FILE ORGANIZATION

```
.
├── CLAUDE.md          — this file
├── MEMORY.md          — resolved decisions + project state
├── CONTEXT.md         — session handoff log
├── DECISIONS.md       — open/deferred/resolved questions
├── CHANGELOG.md       — what shipped
├── .llmignore         — protected paths
├── .claude/           — settings.json: deny rules mirroring .llmignore
├── README.md          — what kivuli is, an example, requirements, the API at a glance, testing
├── lugha.toml         — package `kivuli`, source `src`
├── src/               — module paths follow folders (`src/core/diff.la` is `kivuli.core.diff`; DECISION-026)
│   ├── lib.la         — the entry, `import kivuli;`: re-exports the view, style and widget API, adds sandbox
│   ├── testing.la     — kivuli.testing: pure helpers for app tests, re-exported by lib.la
│   ├── styles.la      — styles as values: `Styling`, `Sheet`, `State` and `When`, `Look` (several values as one), the merge: pure
│   ├── ui.la, ui/     — kivuli.ui: components from public style values only (ui/badges.la: `badge`, `badge_with`, `Badge`'s variants; ui/buttons.la: `button`, `button_with`, `Button`'s variants and sizes, apps call it `ui.button` beside the native `button`; ui/cards.la: `card` and its parts; ui/inputs.la: `input`, `Input.invalid()`; ui/fields.la: `field` and its parts, `field_group`; ui/input_groups.la: `input_group` and its parts; ui/separators.la: `separator`, `separator_with`), pure; a component file imports the modules the entry re-exports, never `lib`
│   ├── view/          — what apps build trees with, all pure: views.la (View and the modifiers), mapping.la (the bodies of `View`'s node-rewriting methods: `map`, `.disabled`, `.image`/`.icon`, `.on_submit`, `.menu`; imports views, which imports it: the one allowed cycle), widgets.la (the builders), looks.la (Style and its types), menus.la (`MenuEntry`, `item`, `menu_separator`), pictures.la (`Image`, an image's identity, PNG/JPEG header checks, base64 and `data:` URLs), walk.la (every node of a view, a named node's path, the check on a `button_with`'s content), keys.la (shortcut keys → GTK's form, the duplicate check), lazies.la (lazy nodes built or reused before each diff)
│   ├── styles/        — the style groups and their CSS, all pure: fonts.la (`Font`, `Text`), boxes.la (`Background`, `Border`, `Outline`, `Spacing`, `Sizing`), drawing.la (`Shadow`, `Effects`) — each group with its utilities, importing styles back (the second allowed cycle) —; values.la (the plain values groups are made of: enums, `Stop`, `Side`, `Edges`, `Drop`, `Filter`, `Move`), theming.la (`Color`, `hex`, `rgb`, `Paint`, `Token`, `Colorish`), palettes.la (`Palette`: shadcn's bases, `.with`, `.radius`, their CSS variables and the root's palette class), motion.la (`Motion`, `Animate`, `Easing`, `Edge`: a style group, importing styles back), motion_rules.la (motion's CSS: transitions, animations and their keyframes, the reduced-motion blocks, `rule_text` writing each rule whole), sheets.la (a `Sheet`'s checks and atomic CSS rules, a background picture's rule), box_rules.la (the CSS for a node's box, shadows and effects; a clamp's limits), paint.la (fills each node's classes before every diff, collects the view's CSS rules)
│   ├── core/          — the pure runtime: diff.la (tree diff → patches), effects.la (Cmd, Sub, App), live_tree.la (the Live tree type and widget(): data only), paths.la (finding a node in the live tree by path, looking through menus)
│   └── render/        — the only GTK callers: live.la (run loop, commands, timers), build.la (creating widgets), styling.la (applying a Style to a widget), patch.la (applying one patch), buttons.la (buttons and `button_with`'s, their click handler and shortcut controller), popovers.la (context menus: popover, actions, gestures, the Menu key), controls.la (the drop-down's and slider's helpers), textures.la (a checked image → a GTK texture), schemes.la (`.scheme(…)` → GTK's colour scheme), windows.la (the window's title and size, opening maximized, clicking away to clear focus). Imports run one way: live_tree ← styling ← build ← patch ← live; effects ← live
├── examples/          — hello.la (the first app), components.la (kivuli.ui's components in light and dark), motion.la (eases, a lifting tile, toasts sliding in, a spinner, skeletons), palette.la (palettes and tokens: bases, a brand colour, light and dark), type.la (a specimen of every Font and Text value), backgrounds.la (a specimen of every Background value), borders.la (a specimen of borders, outlines, padding, margin, sizes, states, shadows and effects), tasks.la (a to-do list: keyed lazy rows, a row menu; with its own tests), browser.la (a header bar, tab menus, a wrap, button_with rows, Cmd.task loads under an overlay note; with its own tests), controls.la (drop-down, sliders, check box, text area, icons, pictures; with its own tests; assets/sample.png), split.la (components with map), bench.la (a benchmark: build with -O2 and run)
├── test/              — @test files, flat (DECISION-027), named after the module they cover: lib.la, entry.la (the API through `import kivuli;`), diff.la, effects.la, keys.la, lazy.la, map.la, testing.la, and the GTK ones (skipped without a display) live.la, live_forms.la, live_style.la, live_effects.la, live_lazy.la, live_map.la, live_health.la (cursor kept, removed widgets finalized), live_choices.la (drop-downs, sliders), live_images.la (icons, pictures, button content), live_disabled.la (disabled text fields), live_pictures.la (images from bytes, hostile bytes, textures freed), live_tasks.la (background tasks while GTK runs), live_wrap.la (wrapping grids), live_overlay.la (overlays), live_menu.la (context menus), live_button_with.la (buttons showing a view), live_header.la (the header bar), live_type.la (styles as classes, the growing stylesheet, colours read back), live_sizing.la (the clamp), live_focus.la (clicking away clears focus), live_palette.la (a root's palette class, `.scheme` and GTK's setting), live_motion.la (motion's classes and rules, a kept widget, reduced motion), live_text_align.la (labels' xalign and justify), live_buttons_ui.la (a ui button's label takes only its decoration); diff_images.la, diff_wrap.la, diff_overlay.la, diff_menu.la, diff_button_with.la and diff_sizing.la (diff.la is full), menus.la (menu entries: pure), pictures.la (header checks: pure), fixtures.la (tiny PNG/JPEG bytes for the image tests, `import fixtures;`); styles.la (the merge rule), sheets.la and sheets_border.la (CSS and checks), spacing.la (the spacing merge), sizing.la (sizes, a clamp's limits), states.la (states: merge, rules, refusals), shadows.la (shadows and effects), utilities.la (each style utility is its full value), badges.la (the badge, its variants, `Look`), text_align.la (text alignment's values, merge, state refusal), buttons_ui.la (kivuli.ui's button: variants, sizes, states, still native), cards.la (the card and its parts), inputs_ui.la (kivuli.ui's input, its states, still native), fields.la (the field's parts, groups, invalid), input_groups.la (the group's frame, add-ons, inner field, buttons), separators.la (the line, the labelled line), palettes.la (tokens, palettes, the radius tokens, root-only checks), motion.la (motion's values, merge, CSS, keyframes, reduced motion, checks) and theming.la (colours, the paint pass) are pure. A test file imports another test file by its bare name (`import support;` for `test/support.la`), not `test.support` or `kivuli.test.support` (those are E0312)
├── PRPs/              — prp-{NNN}-{feature_name}.md (+ TEMPLATE.md, DISCOVERY.md)
├── docs/
│   ├── CODE_STYLE.md  — documentation rules for .la code
│   ├── DESIGN.md      — widget styles → GTK CSS classes
│   ├── source/        — meetings/, research/, stakeholder/ (kickoff brief), constraints/: read on demand
│   ├── specs/ decisions/ incidents/ status/
│   └── setup.md       — the context-system guide this setup follows
└── reports/           — EOD reports (YYYY-MM-DD.md)
```

Update this tree when modules are added.

---

## ANTI-PATTERNS

1. **Never use a Lugha feature without checking `lughac spec`.** The language is young, and inventing syntax (trailing closures, for example) wastes a session. If documented syntax is rejected, check the installed `lughac` is current before anything else.
2. **Never name a function `of`.** It is a keyword.
3. **Never give a method the same name as a field (E0310).** Name fields `ptr`, methods `handle()`, as lugha-gtk does.
4. **Never name a parameter after an imported module (E0311).** No `fun f(gtk: …)` in a file that imports `gtk`.
5. **Never call an interface's default method on the concrete type.** It is callable only through an interface value or a bound.
6. **Never compare messages or views with `==`.** Functions have no `==`, so a `View` holding closures isn't `Eq`. Diff by node kind and fields, and re-point handler `Ref`s.
7. **Every modifier returns `View<Msg>`, exactly the receiver's type.** lughac infers `Msg` for `text("x").align(Align.Center)` only through methods returning the receiver's own type (lugha PR #48); a modifier returning anything else breaks chaining in apps.
8. **Never reassign a captured variable in a closure (E0511).** Capture a `Ref` and `set`/`update` it.
9. **Never add framework-owned local widget state, multiple windows, or a virtualised list in v1.** They are deferred by decision, not by oversight.
10. **Never write an extern from memory.** Open the header. GTK 4 removed or renamed many GTK 3 functions.

---

## KNOWN ISSUES — DO NOT FIX

- **Some calls still block the window**: reading files, DNS lookups, `std.crypto`, `std.process` hold Lugha's one OS thread until they return, inside `Cmd.task` or not. Downloads and sockets don't (PRP-011). Only a Lugha runtime change fixes it (DECISION-022).
- **Messages with closures can't be compared**, so handlers are never reconnected. This is the design, not a gap.
