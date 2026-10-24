# CLAUDE.md — kivuli

kivuli is an Elm-architecture UI library for Lugha, rendered with GTK 4 through lugha-gtk. Rapidkoi (`../rapidkoi`) is built on it.

Where things are recorded: **DECISIONS.md** (every decision and its why: core decisions, then open, deferred and resolved), **PRPs/** (a spec per feature), **CHANGELOG.md**, and git's log. Commit messages say what changed and why.

---

## HOW WE WORK

- **Discuss before building.** Research rather than guess, and cite what you checked.
- **A feature needs an approved PRP** in `PRPs/prp-{NNN}-{feature_name}.md` (next unused number, from `PRPs/TEMPLATE.md`). Without one: no code; run a discovery (`PRPs/DISCOVERY.md`): research everything you can, write the PRP, and ask only the questions that truly need the user, each with a recommendation; build after approval. A chore skips the PRP when the user says so.
- **One PRP at a time.** If the scope grows mid-way: stop, say what you found, and get approval before going on.
- **A branch per PRP or chore**, from `main`. Merge into `main` only when the user asks, after the full suite passes.
- **Never make an architectural choice silently.** Check DECISIONS.md first: `open` blocks the work (ask the user); `resolved` is followed, not re-litigated; a new question is added as `open`, and you stop.
- **Missing or broken lughac or lugha-gtk features are never worked around.** Record it in DECISIONS.md, report it, and stop. It's fixed in that repository under its own rules, never from here.
- **Files stay at or under 500 lines**, tests included. At the limit, propose a split and wait for approval.

## WHERE WORK COMES FROM

Rapidkoi's open requests are in `../rapidkoi/docs/specs/kivuli-requests.md`, one section each, naming its Rapidkoi DECISION number. The suggested API there is kivuli's to decide, through a PRP. When one ships, tell the user, so Rapidkoi can resolve its decision and remove its section.

## TESTING

- **Test first.** For every new function, a failing `@test` first, then the least code that passes it. Every `public` function gets a happy-path test and one per error it can return.
- Tests live in `test/`, flat (DECISION-027), named for the module (`src/core/diff.la` → `test/diff.la`). No test code in `src/`. A test file imports another by bare name (`import support;`).
- Views, `map`, the diff and subscriptions are tested with no display, through `kivuli.testing` and the pure modules.
- GTK tests (`test/live*.la`) start with `if !has_display() { skip("no display"); }`.
- **Run the headless tests yourself**: the files covering the change while working, the full suite before saying a task is done. The user checks anything visual: never claim something looks or works on screen because it compiled.
- A task is done when `lughac fmt --check src examples test` is clean, `lughac test test/` passes (skips only for "no display"), the examples' tests pass, and the examples using what changed pass `lughac check`. Every example builds before a push.

## SECURITY

Never build these without the user's explicit review: authentication, sessions, cryptography, secrets, or running processes from library code. Never hardcode credentials, even in examples; validate external input in examples. A new dependency or `[link]` library needs its reason in the PRP. **An `extern` matches its C prototype in `/usr/include/gtk-4.0` (or GLib's headers) exactly: open the header, never write one from memory.** A wrong signature is memory corruption.

## OUTWARD-FACING ACTIONS

Ask before anything that leaves this machine: creating repositories, adding remotes, pushing, tagging, opening PRs or issues. Rapidkoi depends on kivuli by git tag, so a tag is a release. Before a push or tag, check the test run's real exit status: `… | tail` hides a failure.

## PROTECTED FILES

Never read, modify or delete: `.env` and `.env.*`; `lugha.lock` (written only by `lughac build`/`update`); `LICENSE` (GPL-3.0-only, the user's); built outputs (the paths in `.gitignore`). Sibling repositories are read for reference and never written from here: `../lugha/`, `../lugha-gtk/`, `../rapidkoi/`, `../rapidkoi-rust/`. Keep `.llmignore` and `.claude/settings.json` in step with this list. Never build or install `lughac` yourself: when a newer one is needed, ask the user (then **Developer: Reload Window** in VS Code).

---

## COMMANDS

```bash
lughac check examples/hello.la            # type-check one entry point
lughac test test/                         # every @test (or name files: lughac test test/diff.la)
for f in tasks browser controls; do lughac test examples/$f.la || break; done   # the examples' tests, one at a time (DECISION-029)
lughac fmt --check src examples test      # formatting; drop --check to rewrite
lughac build examples/hello.la -o examples/hello   # build one example (needs gtk4 via pkg-config)
lughac run examples/hello.la              # build and run: opens a window, so it's the user's
lughac spec                               # the language reference: check it before using any feature
```

## DESIGN AND CODE STYLE

- **Read `docs/DESIGN.md` before writing any widget, modifier or style.** kivuli ships no colours, fonts or pixel values of its own: looks come from the GTK theme or from the app's typed style values.
- **Read `docs/CODE_STYLE.md` before writing code.** Every `public` item has a `/** … */` comment; every non-obvious decision inside a function has a `//` comment saying why.

## ARCHITECTURE

1. **Apps see only the entry (`src/lib.la`, `import kivuli;`), `kivuli.ui` and `kivuli.testing`.** Everything else is package-private.
2. **Imports run one way.** Private modules import `views`, never the entry `lib`; `build` never imports `live`. The only cycles: `views` ↔ `mapping` (DECISION-018) and `styles` ↔ `styles/` (DECISION-025). `kivuli.ui`'s components use public style values only.
3. **`diff` is pure**: two `View` trees in, patches out, no GTK.
4. **Only `src/render/` calls GTK or GLib.** `core/effects.la` is data.
5. **Connect each GTK signal once per widget, never per render.** The handler reads its message from a `Ref` kept with the live widget; a patch only `set`s it (core decision 13).
6. **Disabled means no message**: `on_click: Option<Msg>`, `None` → insensitive. No separate `disabled` flag.
7. **State lives in one `Ref` inside `Running` (`live.la`).** `dispatch(msg)` = update → view → diff → patch → store. Nothing else touches the model.
8. **Never overwrite an entry's text that already equals the model's,** and guard `changed` against echoes of programmatic `set_text`.
9. **kivuli uses `gtk.raw` only**, imported by name through the entry: `import gtk { GtkWidget, gtk_box_new };`. A missing function is added in lugha-gtk itself.
10. **A new `Live` variant** needs its arm in `test/live.la`'s `shown()`.

**Errors:** expected failures return `Result<T, E>` with the module's own error enum; programmer errors `panic` with a message naming the node. Never `Option` for an error, never a bare `string` error, never `unwrap`/`expect` on anything from app input (`expect("…")` only for true invariants, stating them). The app API (`sandbox`, `application`, widgets, modifiers) doesn't return `Result`; `kivuli.testing` does ("not found" is an expected outcome).

## ANTI-PATTERNS

1. Never use a Lugha feature without checking `lughac spec`. If documented syntax is rejected, the installed `lughac` may be stale: ask the user.
2. Never name a function `of` (a keyword), a method like a field (E0310: fields `ptr`, methods `handle()`), or a parameter or local after an imported module (E0311: `import kivuli.core.diff { diff as patches };`).
3. Never reassign a captured variable in a closure (E0511): capture a `Ref` and `set`/`update` it.
4. Never compare views, messages or patches with `==`: functions have no `==`. Tests compare printed forms (`assert_same`) and call handlers (core decision 15).
5. Every modifier returns exactly `View<Msg>`, the receiver's type, or chaining loses `Msg` in apps.
6. Never call an interface's default method on the concrete type: only through an interface value or a bound.
7. Never add framework-owned local widget state, multiple windows or a virtualised list without a decision: they are deferred on purpose.

Known gaps waiting on lughac, lugha-gtk or GTK are DECISIONS.md's deferred entries: don't fix or work around them here.
