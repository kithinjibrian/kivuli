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
- Tests of the public API (`ui`, `ui.testing`) live in `test/`, mirroring `src/`: `src/ui.la` → `test/ui.la`. Files in `test/` are outside the `kivuli` package and see only `public` items.
- Tests of private modules (`diff`, `live`, `effects`) go at the bottom of the module itself, after a `// Tests.` comment, because only there can they reach package-level items. Their test message enum must not be named `Msg` (it clashes with the `Msg` type parameter, E0316).
- Run everything with `lughac test test/ src/`.
- Tests must run with **no display and no GTK window**. Test the pure parts: view construction, modifiers, `map`, diff (tree → patches), `ui.testing`, subscription bookkeeping. The live GTK layer is covered by a windowless smoke program (`test/smoke.la`, like lugha-gtk's) that builds widgets, reads them back and quits.
- Test behavior visible to callers: inputs, outputs, and every error variant. Do not test private internals or GTK's own behavior.
- Every new `public` function gets at least one happy-path test and one test per error variant it can return.
- After any non-trivial change, run the full suite and check every example before calling the task done.
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

Ask before anything that leaves this machine: creating the GitHub repository, `git push`, tagging a release, opening PRs or issues in other repositories. The plan (`gh repo create kithinjibrian/kivuli --public`, push `main`, tag `v0.1.0`) runs only on the user's go.

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
- `../lugha-gtk/`: write only to add GTK bindings kivuli needs, when the user asks (DECISION-004). Commit only your own change, never the user's uncommitted work there, and push only on the user's go

If a task seems to require touching a protected file, stop and ask. See also `.llmignore`. `.claude/settings.json` denies Claude's Read and Edit tools on the same paths. Keep all three in sync.

---

## COMMANDS

```bash
lughac check examples/counter.la      # lex, parse, type-check (one entry point at a time)
lughac build examples/counter.la      # compile and link (needs gtk4 via pkg-config)
lughac run examples/counter.la        # build and run (opens a window, so it's the user's)
lughac test test/ src/                 # every @test (public-API tests in test/, private ones in src/), no display
lughac fmt --check src examples test  # formatting; drop --check to rewrite
lughac update examples/counter.la     # re-resolve git packages, rewrite lugha.lock
lughac explain E0312                  # what an error code means
lughac spec                           # the full Lugha language reference: check it before assuming a feature exists
```

A task is done only when `fmt --check` is clean, `lughac test test/ src/` passes, and every file in `examples/` passes `lughac check` and `lughac build`.

> Needs: `lughac` (installed from `../lugha` with `cargo install --path .`), `libgtk-4-dev` (GTK 4.22), `pkgconf`, `libgc-dev`, `cc`.

---

## STACK

- **Lugha v16/v17** via `lughac` 0.1.0. v16 is frozen and complete. v17 is in progress in `../lugha` (milestone 91 green threads done, 92 actors being built). Use only what `lughac spec` documents.
- **lugha-gtk 0.1** (0.1.1 or later: `gtk_box_remove`, `gtk_box_insert_child_after`; `gtk = { git = "https://github.com/kithinjibrian/lugha-gtk", version = "0.1" }`). kivuli uses **`gtk.raw`** only, because it manages widgets itself. It does not use `gtk.widgets`.
- **GTK 4.22** + GLib, linked through `[link] pkg-config = ["gtk4"]`.
- **Boehm GC** (through lughac): closures handed to GTK are pinned with `data`/`release` callbacks.
- License: **GPL-3.0-only**, like Lugha.

---

## ARCHITECTURE RULES

1. **Apps see only `ui` and `ui.testing`.** `views` is reached through `ui`'s re-export; `live`, `diff` and `effects` are private (package-level). *Why:* the public surface is the Elm contract, and leaking widgets lets apps bypass `update`/`view`.
2. **No private module imports `ui`.** They import `views`. *Why:* `ui` imports `live`, so a private module importing `ui` makes a cycle back to itself when its own tests run it as the main file (E0319).
3. **`diff` is pure.** It takes two `View` trees and returns patches. It never calls GTK. *Why:* it must be unit-testable with no display, and it is where most bugs will live.
4. **Only `live.la` and `effects.la` call GTK/GLib.** *Why:* one place to audit every extern call and every pinned closure.
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
├── lugha.toml         — package `kivuli`, source `src`
├── src/               — ui.la (public API: re-exports views, adds sandbox), views.la (View and its builders;
│                        imports nothing, so private modules' tests have no cycle back to it),
│                        diff.la (pure tree diff → patches, tests inside), live.la (live widgets, the run loop);
│                        planned: testing.la, effects.la
├── examples/          — hello.la (stale `lughac init` scaffold, see CONTEXT.md); planned: counter, todo, stopwatch
├── test/              — @test files mirroring src/, plus smoke.la (windowless)
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

1. **Never use a Lugha feature without checking `lughac spec`.** The language is young, and inventing syntax (trailing closures, `..base` before it lands, `Msg.Named` as a value before it lands) wastes a session.
2. **Never name a function `of`.** It is a keyword.
3. **Never give a method the same name as a field (E0310).** Name fields `ptr`, methods `handle()`, as lugha-gtk does.
4. **Never name a parameter after an imported module (E0311).** No `fun f(gtk: …)` in a file that imports `gtk`.
5. **Never call an interface's default method on the concrete type.** It is callable only through an interface value or a bound.
6. **Never compare messages or views with `==`.** Functions have no `==`, so a `View` holding closures isn't `Eq`. Diff by node kind and fields, and re-point handler `Ref`s.
7. **Never reassign a captured variable in a closure (E0511).** Capture a `Ref` and `set`/`update` it.
8. **Never add framework-owned local widget state, multiple windows, or a virtualised list in v1.** They are deferred by decision, not by oversight.
9. **Never write an extern from memory.** Open the header. GTK 4 removed or renamed many GTK 3 functions.

---

## KNOWN ISSUES — DO NOT FIX

- **`update` may block** (direct `std.fs` / `std.process` calls) and freeze the window. File, process and network I/O are not commands in v1, by decision.
- **Messages with closures can't be compared**, so handlers are never reconnected. This is the design, not a gap.
- **`examples/hello.la` fails `lughac check` (E0312)**: it imports `kivuli.kivuli`, which doesn't exist. Leave it until PRP-001 replaces it.
