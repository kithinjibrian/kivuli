# CONTEXT.md — kivuli

Session handoff file. Updated at the end of every session.
Read at the start of the next session alongside CLAUDE.md, MEMORY.md, and DECISIONS.md.

Every session has a name and a state: open | closed.
A session is closed only after CONTEXT.md is committed (and pushed, once a remote exists).

---

## SESSION 1 — 2026-10-17 — Context system setup — closed

Branch: main

### WHAT WAS DONE

Set up the context system from `docs/setup.md`, adapted to a Lugha library package rather than the guide's TypeScript defaults. The content comes from the kickoff brief (`../KICKOFF.md`, copied to `docs/source/stakeholder/kickoff-2026-10-16.md`) and lugha's own `CLAUDE.md`. Initialised the git repository on `main` (local only, no remote).

Found while setting up:
- `src/` is empty. `examples/hello.la` (a `lughac init --lib` scaffold) imports `kivuli.kivuli`, which doesn't exist, so `lughac check` fails with E0312. It is left in place for PRP-001 to replace.
- The brief's prerequisite milestones 91–92 (struct update, variants as functions) were taken in lugha by green threads (91, done) and actors (92, lugha session 214, open). Neither prerequisite is built. Recorded as DECISION-001.
- An empty `tests/` directory appeared during the session (not created by this session). `test/` is the planned test folder. Ask the user whether to remove `tests/`.

### FILES CREATED OR MODIFIED

```
CLAUDE.md                     — behavioral rules for a Lugha/GTK package
MEMORY.md                     — decisions 1–12 from the brief + handler-Ref trick + gtk.raw
CONTEXT.md                    — this log
DECISIONS.md                  — DECISION-001…004 open, 005…007 deferred
CHANGELOG.md                  — started
.llmignore, .claude/settings.json — protected paths
.gitignore                    — built example executables added
PRPs/TEMPLATE.md, PRPs/DISCOVERY.md — PRP template and discovery protocol
docs/CODE_STYLE.md            — /** */ doc comments, why-comments
docs/DESIGN.md                — style vocabulary → GTK CSS classes and properties
docs/setup.md                 — moved from ./setup.md
docs/source/stakeholder/kickoff-2026-10-16.md — copy of ../KICKOFF.md
docs/{specs,decisions,incidents,status}/, docs/source/*/, reports/, test/ — empty, with .gitkeep
```

### TESTS WRITTEN

- None (no code).

### DECISIONS MADE

- Tests live in `test/` (from the brief's layout), mirroring `src/`, headless only.
- File size limit 500 lines (brief §8).
- Sessions close on commit until a remote exists, because pushing is outward-facing and needs the user's go.

### PENDING DECISIONS OPENED

- DECISION-001: wait for the Lugha prerequisites or start kivuli now.
- DECISION-002: shape of `lazy` (R1). DECISION-003: `ui.testing` module path. DECISION-004: where the extra externs live.

### STILL OPEN AT CLOSE

- `examples/hello.la` fails to check (E0312).
- No LICENSE, README, or remote yet.
- Stray empty `tests/` directory.

---

## SESSION 2 — 2026-10-17 — kivuli v1 steps 0–4, lughac and lugha-gtk fixes — closed

Branch: main

### WHAT WAS DONE

The user chose to be hands-on: small steps, code shown and walked through. Steps 0–4 of the kickoff brief's plan were built that way, without PRP-001 (the user's choice; see STILL OPEN).

- Step 0: `examples/raw.la`, a counter written straight on GTK.
- Steps 1–2: `View<Msg>` as plain data (`src/views.la`), `sandbox` (`src/lib.la`), and a live layer that first rebuilt the window per message.
- Step 3: `src/diff.la` (pure, position-based diff into patches) and patching in `src/live.la`: a `Live` tree mirroring the widgets, signals connected once per widget with handlers reading their message or function from a `Ref`, a `patching` flag dropping echoes of kivuli's own patches.
- Step 4: `row`, `input` (keeps the cursor: no rewrite of text the entry already shows), `checkbox`, `spacer`, `separator`, `scroll`, `keyed_column` (rows matched by key: Remove, then Insert/Move, each kept row diffed in place).
- Public API moved to the package entry `src/lib.la` (`import kivuli { … }`) when lughac milestone 99 added it; GTK is imported as `import gtk { GtkWidget, gtk_box_new, … }` since lugha-gtk 0.1.8 globs `gtk.raw` (lughac milestone 100).
- lugha-gtk (the user's go each time; pushed and tagged): 0.1.1 box remove/insert, 0.1.2 `emit_widget` (C wrapper), 0.1.3 entry `lib.la`, 0.1.4 entry re-exports raw, 0.1.5 check buttons, 0.1.6 expand/separator/scrolled window, 0.1.7 reorder and child walking, 0.1.8 `public import raw { * }`.
- lughac quirks found while building were reported and fixed in lugha (m93–m100, PRs #32–#45): see the notes below. The user now installs lughac; Claude never does.

### FILES CREATED OR MODIFIED

```
src/lib.la          — the entry: re-exports views, adds sandbox
src/views.la        — View<Msg> and its builders (text, button, column, row, input, checkbox, spacer, separator, scroll, keyed_column)
src/diff.la         — Patch<Msg>, diff (positional and keyed), list helpers
src/live.la         — Live tree, run/dispatch, create_in (parent axis), apply, update_at
test/lib.la, test/diff.la — pure tests, compared by printed form (assert_same)
test/live.la, test/live_forms.la — GTK tests on hidden windows, skip("no display") without one
examples/counter.la — counter with a name field, a row, a checkbox, a separator
examples/raw.la     — the same counter on raw GTK, for comparison
lugha.toml, lugha.lock — gtk = lugha-gtk 0.1.8 (one lock at the root)
CLAUDE.md, MEMORY.md, DECISIONS.md, CHANGELOG.md, PRPs/TEMPLATE.md — kept in step with the above
```

### TESTS WRITTEN

- test/lib.la — every builder makes the node it says; input/checkbox handlers called directly.
- test/diff.la — each patch kind; rows and keyed rows (delete, insert, rotate, reverse, combined); scroll paths; kind changes give Replace.
- test/live.la — a run of messages patches the same widgets; separators/spacers follow their parent's axis; scroll Append/Replace; keyed rows reordered on screen (read back from GTK) and kept as the same widgets.
- test/live_forms.la — typing and ticking reach the model; kivuli's own patches send no echo (each checked to fail with the echo guard removed).

41 tests: 36 pure, 5 GTK (skipped without a display).

### DECISIONS MADE

- DECISION-001 resolved (prerequisites landed as lugha m93/m94); DECISION-004 resolved (missing GTK bindings go into lugha-gtk); DECISION-008 resolved (tests compare views by printed form: functions make View non-Eq).
- Public API is the package entry `src/lib.la`; private modules import `views`. GTK through `import gtk { … }` by name.
- All tests in `test/`; GTK tests skip without a display; test/live.la split into live.la + live_forms.la when it passed 500 lines (the user's choice), each repeating two small helpers. That rested on a wrong claim of mine: test files *can* import each other, by bare name (`import support;`); I had only tried `test.support` and `kivuli.test.support` (corrected in session 3).
- Never build or reinstall lughac: ask the user (saved in Claude's memory and CLAUDE.md).

### PENDING DECISIONS OPENED

- DECISION-008 (opened and resolved this session). DECISION-002 (`lazy`), DECISION-003 (testing helpers re-exported from the entry?) remain open.

### STILL OPEN AT CLOSE

- No PRP-001: steps 0–4 were built hands-on at the user's request. The user never answered whether to write PRP-001 or relax the PRP rule for this kind of work. Ask before step 5.
- `examples/hello.la` is empty (no `main`); a stray empty `tests/` folder exists. Both the user's to decide (delete?).
- No README, no LICENSE (the user adds LICENSE), no remote.
- lughac quirks still to report: `lughac test`'s summary leaves out files that failed to compile (exit code is right); `lughac fmt` spreads a short `if a { 1 } else { 0 }` over five lines; inference doesn't reach a call inside a tuple literal (fixed in session 3, PR #46). Not a quirk: test files importing each other works by bare name; my report was wrong (session 3).
- Not covered by a test: input's same-text check (needs `gtk_editable_get_position` in lugha-gtk); duplicate keys in keyed_column panic, but lughac test can't expect a panic.

### NOTES KEPT DURING THE SESSION

- lughac bugs to report in the lugha repo (not fixed here, per SCOPE RULE):
  1. `lughac fmt` exits 2 ("failed verification") on a `match` where a `{ … }` arm is followed by an arm starting with `(`: it drops the comma after the block, and `} (a, _) =>` then parses as a call. Repro: `fun f(x: i64, y: i64): i64 = match (x, y) { (0, _) => { 1 }, (a, _) => a, };` — runs fine, fmt crashes. Avoided in src/diff.la by moving the block into `diff_button`.
  2. `lughac fmt` puts a lone trailing array argument on one line and splits the signature (`fun view(\n    m: Model,\n): View<Msg> = column(\n    [a, b, c],\n);`) instead of hugging `column([` … `])`. Every kivuli `view` looks like this.
- lughac reinstalled from lugha main (e7abc33, m94) for struct update and variants as functions; the fmt crash (1) is still present there.
- lughac bug (3): a package whose `[link]` table has `pkg-config = ["gtk4"]` can't also list `c` files: lughac runs `pkg-config --cflags gtk4`, which prints `-mfpmath=sse -msse -msse2`, and E0329 rejects any cflag but -I/-D/-pthread instead of dropping it. Blocks lugha-gtk shipping a C wrapper for variadic `g_signal_emit_by_name` (Lugha can't declare variadics), so gtk.raw has no way to emit a signal. Fix is in lugha `src/link/` and needs the user's security review there.
- lugha-gtk: examples/test formatted (964324d, committed, not pushed); the user had already pushed src/ formatting (41ff852).
- lughac quirks 1–9 re-checked on lugha main 4c5a1c9 (m95–m97, PRP-129), lughac reinstalled: 1, 2, 4, 5, 6, 8 fixed; 3 partly (inference doesn't reach calls nested in an array argument: `assert_eq(column([text("a")]), expected)` is E0412); 7 partly (`lughac check src/live.la` from the package root can't find `gtk.raw`, fine from inside src/ and through ui.la); 9 fixed but a package's own `[link]` C files are linked twice when building inside that package (duplicate symbol, misleading "missing library" hint).
- Tests moved: diff's tests to test/diff.la; GTK smoke test restored as test/live.la with skip("no display"); stale per-folder lugha.lock files removed (one root lock now).
- lughac quirks 3, 7, 9 re-checked on lugha main 5a64da8 (PR #40): all fixed; all nine quirks closed.
- lugha-gtk 0.1.2 (tag v0.1.2, 2882c72, pushed): `raw.emit_widget` through c/signals.c, tested in its test/emit.la; README row fixed after (64c1984). kivuli requires 0.1.2; test/live.la presses the real Add button; test/diff.la no longer needs the t()/col() helpers.
- lughac reinstalled at lugha ed6c9cb (m99, package entry lib.la). kivuli: ui.la -> lib.la (`import kivuli { … }`). lugha-gtk: src/lib.la entry (0.1.3, v0.1.3) re-exporting raw too (0.1.4, v0.1.4), both pushed; kivuli requires 0.1.4 and imports `gtk;` then `gtk.raw.…`.
- lughac quirk (new): `lughac test` prints a non-compiling file's errors but its summary still counts only the files that built ("2 tests: 2 passed, 0 failed"); exit code is 1, correctly. Also: `lughac fmt` spreads a short `if a { 1 } else { 0 }` over five lines.
- lugha-gtk 0.1.5 (check buttons) and 0.1.6 (expand, separator, scrolled window) pushed and tagged; kivuli requires 0.1.6. GTK wraps a non-scrollable scrolled-window child in a GtkViewport, so kivuli keeps its own Live child.
- lughac quirks (new): sibling inference doesn't reach a call inside a tuple literal (`keyed_column([("a", text("a"))])` against an annotated value is E0412; tests annotate a `let`); test files can't import one another (`import test.support;` and `import kivuli.test.support;` are E0312), so shared GTK test helpers are repeated per file.
- lugha-gtk 0.1.7 pushed and tagged (reorder_child_after, get_first_child/get_next_sibling); kivuli requires it. keyed_column done; test/live.la split (527 lines) into live.la + live_forms.la, the user's choice.
- lughac m100 (glob re-exports, PR #45) verified on the user's install. lugha-gtk 0.1.8 (v0.1.8, 9af15ad, pushed): `public import raw { * };` in lib.la. kivuli requires 0.1.8 and imports `gtk { GtkWidget, gtk_box_new, … }` by name, no `gtk.raw.` prefix.

## SESSION 3 — 2026-10-17 — v1 steps 5–8, PRPs 001–005, kivuli 0.1.0 published — closed

Branch: main (each PRP on its own branch, merged with --no-ff)

### WHAT WAS DONE
- v1 steps 5–8 hands-on: modifiers and styles, `application` with Cmd/Sub, `lazy`, `kivuli.testing`, examples (todo, stopwatch, counters), README; `map`; cleanups and health checks.
- From then on every feature through a PRP: 001 text area, 002 long text and tooltips, 003 shortcuts and focus, 004 drop-down and slider, 005 icons and images. All merged into main.
- lugha-gtk 0.1.9 … 0.1.22 released (standing go).
- kivuli 0.1.0 published on the user's go: github.com/kithinjibrian/kivuli (public), `main` and tag `v0.1.0` at 8fe0362; LICENSE (GPL-3.0-only) added by the user. A fresh package built the README counter from GitHub.

### STILL OPEN AT CLOSE
- `test/live_forms.la` is 562 lines (over 500): proposed split, its focus and shortcut tests into `test/live_keys.la`, awaiting the user.
- DECISION-011 (lughac: private imports reachable through a module) deferred, reported for the lugha session.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.
- Reversing 1,000 keyed rows ~1 s (kivuli's keyed diff).
- This close is committed but not pushed: kivuli pushes need the user's go.

### NOTES KEPT DURING THE SESSION
- lughac PR #46 (lugha 9af1d4b, user-installed): inference reaches calls inside tuple literals; test/diff.la and test/lib.la compare keyed columns directly again.
- Correction: "test files can't import each other (E0312)" was my mistake, not a lughac bug. `import support;` (bare name) works from test/; `import test.support;` and `import kivuli.test.support;` are simply wrong forms. Reported back to the lugha session.
- Step 5 planning: the user rejected the PRP question and said to continue hands-on. DECISION-009: modifiers live in one `Styled(View, Style)` wrapper (the user's choice).
- Part 1 (layout modifiers) written: views (Align, Style, plain, Styled, padding/spacing/expand/align/width/height), diff (split style from inside, SetStyle), build.la (apply_style). lugha-gtk 0.1.9 (spacing, halign/valign, size request) committed locally, not pushed.
- Blocked: DECISION-010, lughac can't infer `Msg` for `text("x").padding(8)` (expected type doesn't reach a method's receiver). Prompt given to the user for the lugha session. Nothing of step 5 committed in kivuli yet.
- src/live.la passed 500 lines (518): split into live.la (Live, run/dispatch, apply, paths) and build.la (create, create_in, create_box, apply_style, gtk_align, flag), the user's go. GTK tests import `create` from kivuli.build.
- lughac PR #48 (4e0b028, user-installed): receiver inference fixed; DECISION-010 resolved. Its formatter now keeps a short `if … { 1 } else { 0 }` on one line (my logged quirk): src/ and examples reformatted, whitespace only.
- test/live_style.la: styles applied, changed and undone on real widgets; a padded spacer still grows (fails if spacers lose their growth).
- lugha-gtk 0.1.9 pushed and tagged (v0.1.9 at 4fc4369; a formatting commit 938305a follows on main). Standing go from the user: always push lugha-gtk (memory + CLAUDE.md). kivuli requires 0.1.9; step 5 part 1 committed.
- Step 5 part 2: size/bold as Pango attributes, dim as the `dim-label` class (the user agreed); lugha-gtk 0.1.10 pushed and tagged (Pango, label attributes, CSS classes). docs/DESIGN.md's open choice settled.
- build.la and live.la imported each other only because the Live type sat in live.la: Live and widget() moved to src/live_tree.la (not `tree.la`: `tree` is a common local name, E0311), so imports run live_tree ← build ← live (the user's go). lughac's test summary now counts files that didn't compile (my logged quirk, fixed).
- Step 5 part 3 done in one go at the user's request: ButtonStyle via theme CSS classes; disabled = Button holds Option<Msg> (decision 6), SetMsg carries Option, insensitive widget + handler sends nothing. lugha-gtk 0.1.11 pushed (set/get_sensitive).
- Step 5 part 4 and step 6 done in one go (the user's request): `.window(title, w, h)` (a root style field; the runtime sets title and default size when it changes); `application(App {…})` with Cmd/Sub in src/effects.la (pure) run by live.la (`mount`, `perform`, `resubscribe`); `sandbox` = `application` via `sandbox_app`. GTK tests now build apps through `mount(…, sandbox_app(…))`. lugha-gtk 0.1.12 pushed (GLib timers, clocks, hold/release, window title). examples/stopwatch.la.
- lughac gap, fixed in PR #53 (5bf8838, user-installed): `App { init: (0, Cmd.none()), update: step }` was E0412. I reported it as field order; the real cause was that an associated function like `Cmd.none()` (type from its result only) wasn't recognised by sibling inference. `init` is back first in the stopwatch and lib.la's doc example, workaround comments removed.
- src/live.la is 450 lines: close to the limit.
- DECISION-002 resolved (the user's choice): `lazy(key, build)`. diff.resolve builds or reuses lazies by key before each diff (cache of used keys in Running.built); equal keys give no patches; modifiers on a lazy node are ignored. src/diff.la is 445 lines and src/live.la 457: the next additions to either need a split.
- Step 8: DECISION-003 resolved (the user's choice): src/testing.la re-exported whole by lib.la (`kivuli.testing.click`). Helpers return Result<_, Lookup> (NotFound, Ambiguous(n), Silent). `.placeholder(hint)` (a Style field) and `.on_submit(msg)` (in the Input node, SetOnSubmit patch, `activate` handler) built first, as the user chose. lugha-gtk 0.1.13 pushed (entry placeholders; cleared, GTK reads back "").
- examples/todo.la (the user asked for a good-looking to-do app): theme styles only (primary Add, danger Clear done, subtle tabs and ✕), keyed lazy rows in a scroll, empty states, live window title; 6 tests of its own via kivuli.testing (`lughac test examples/todo.la`); its real view also run through mount on real widgets in scratch (12 messages, no failure). The look itself is for the user to judge.
- README.md written: intro, a counter example (checked: it compiles and builds as written), requirements, using it as a package (not published yet), the API in tables, testing with kivuli.testing, the test suite, licence.
- lughac milestone 101 (maps and sets, PR #56; the user's install): kivuli switched (the user's go). Keyed diff: Set for wanted/present keys and duplicates, a moved row searched only from its position; lazy cache: Map<string, View<Msg>> (resolve's signature takes and returns a Map); resubscribe: Set of wanted keys, Map of kept timers. index_of, keys_of, lookup, index_of_key gone; diff.la 438 lines, live.la 453. Mutation checks rerun: still caught.
- map (decision 8): View.map/Cmd.map/Sub.map; examples/counters.la. Found while planning it: lazy's global key cache would let two mapped copies of a component share a subtree (wrong handlers, silently). The user chose per-place reuse (Elm's rule): diff.resolve(new, old) walks beside the last view, matching places as the diff does; the cache is gone. DECISION-002 amended. test/live_map.la shows the right copy rebuilt alone with equal keys on both sides.
- Cleanups and health checks (the user's request): examples/hello.la removed; patch application split from live.la into src/patch.la (live 239, patch 225). lugha-gtk 0.1.14 pushed (editable cursor position, on_finalized via g_object_weak_ref). test/live_health.la: SetValue keeps the cursor (without the same-text guard GTK puts it at 0 — checked); 101 removed or replaced widgets all finalized.
- Benchmark (examples/bench.la): the diff was O(n²) from `out = joined(out, …)` copying per row — fixed (push in place). Still slow at 1,000 rows: lughac's Map/Set inserts are quadratic (1000: 31 ms, 2000: 198, 4000: 441, 8000: 2491; arrays 0 ms), which dominates resolve (~150 ms) and the keyed diff's sets (~40 ms). Prompt for the lugha session given to the user. 100 rows: 3–4 ms per update.
- From here on, every feature goes through a PRP (the user's decision): discovery one question at a time with a recommended answer, PRPs/prp-NNN-*.md, approval, then code. Steps 0–8 and the work since were built hands-on without one.
- PRP-001 (text area) approved by the user as written; building on branch prp-001-text_area. Showcase: examples/notes.la.
- lughac PR #58 (Map/Set inserts fixed; user-installed): 8,000 inserts 1–3 ms; 1,000-row keyed updates now ~2–25 ms. Reversing 1,000 rows still ~1 s: kivuli's keyed diff rebuilds its working arrays per move (a later PRP if apps need it).
- PRP-001 built: text_area + .name; lugha-gtk 0.1.15 pushed (text views, src/c/text.c). Tests written first (pure, then GTK incl. two mutation checks); examples/notes.la with 3 tests. Merged into main.
- PRP-002 (long text and tooltips) approved and built: builders split into src/widgets.la (impl blocks must stay with their type); lugha-gtk 0.1.16 pushed; check boxes get kivuli's own label child; the to-do app ellipsizes titles and its ✕ has tooltips. Tests run narrowly while building (the user's rule), the full suite once before the merge.
- PRP-003 (shortcuts and focus) approved and built; lugha-gtk 0.1.17 and 0.1.18 pushed. Shortcuts use a callback action (GTK's activate action does nothing off screen). New src/keys.la, src/walk.la.
- PRP-004 (dropdown and slider) approved and built. Styling split out of build.la into src/styling.la first. lugha-gtk 0.1.19 (drop-downs, scales), 0.1.20 (reading a list back) and 0.1.21 (gtk_widget_ref/unref) pushed. v0.1.19 was tagged with a failing test: `lughac test … | tail` in an `&&` chain hid it; fixed on main, and pushes are now gated on the real exit status (CLAUDE.md, memory). The user chose a blank first item for an unmatched `selected`. Replacing a drop-down's list inside its own notify::selected crashed GTK: synced on the next turn instead (PRP-004 notes). New test/live_choices.la; examples/settings.la.
- PRP-005 (icons and images): talked through in chat first; the user chose theme icons only (apps bring other sets themselves), image(path) with .fit, .size(px) on icons, placeholders for missing things, no new test helpers, a gallery example. Style types split into src/looks.la first. lugha-gtk 0.1.22 pushed. GTK's theme has no image-text-button class (text-button + image-button instead). Found DECISION-011 (lughac: private imports reachable through a module), deferred and reported.

---

## SESSION 4 — 2026-10-17 — PRP-006 drafted: disabled text fields — closed

Branch: main

### WHAT WAS DONE
- Closed session 3 (it had been left open) and opened this one.
- Wrote PRPs/prp-006-disabled_text_fields.md from the user's brief: `Input(string, Option<(fun(string): Msg, Option<Msg>)>)`, `TextArea(string, Option<fun(string): Msg>)`, `SetOnChange` carrying the option and the sensitivity, `kivuli.testing.is_field_disabled`, release 0.1.1. No code.

### STILL OPEN AT CLOSE
- PRP-006 awaits the user's approval.
- Sessions 3 and 4 closes are committed, not pushed (kivuli pushes need the user's go).
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 5 — 2026-10-17 — PRP-006: disabled text fields, kivuli 0.1.1 — closed

Branch: main (PRP-006 on prp-006-disabled_text_fields, merged)

### WHAT WAS DONE
- PRP-006 approved and built on prp-006-disabled_text_fields: optional field messages (None = disabled), SetOnChange sets sensitivity, is_field_disabled; kivuli 0.1.1.
- "App look" discussed before its PRP. The user chose typed styles (StyleX-like: an app enum of looks, each a typed Surface, applied with `.style([...])`, later ones win, kivuli generates the CSS) over plain CSS or per-widget colours; and `.style` for it, the button variant renamed `.button_style(ButtonStyle.…)` (breaking: 0.2.0). To record in DECISIONS.md and MEMORY.md with that PRP.

### STILL OPEN AT CLOSE
- v0.1.1 tagged locally; main (6 commits) and the tag not pushed: waiting for the user's go.
- `test/live_forms.la` split (562 lines) awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 6 — 2026-10-17 — PRP-007: app look, kivuli 0.2.0 — closed

Branch: main (PRP-007 on prp-007-app_look, merged)

### WHAT WAS DONE
- PRP-007 written after a talk-through (typed looks via `.style`, `.button_style` rename, window settings on the root) and approved; built on prp-007-app_look. lugha-gtk 0.1.23 pushed (CSS providers, maximize, header bars, C helpers in src/c/look.c). `.dark()` deferred: neither GTK setting switched the theme in tests (DECISION-017, the user's choice). DECISIONS 012 (resolved) and 013–016 (queued features) recorded. kivuli 0.2.0.

---

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: `main` and tags `v0.1.1`, `v0.2.0`.
- For the user to check by hand: `lughac run examples/frame.la` opens maximized, the address field fills the header bar, the rail and selected tab show their colours.
- DECISION-017 (forcing dark) deferred; DECISIONS 013–016 queued (file dialog, overlay, context menus, wrapping grid).
- `views.la` 487, `diff.la` 477, `test/diff.la` 498 lines: split before adding to them. `test/live_forms.la` (562) split still awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 7 — 2026-10-17 — Drop the empty-list workaround (lughac PR #68) — closed

Branch: main

### WHAT WAS DONE
- lughac PR #68 (user-installed): an empty `[]` takes its type from a sibling branch. test/live_look.la's typed `only_if` helper removed; the conditional looks are inline again; its tests pass.
- Checked the other annotated lets (test/map.la, examples/gallery.la, examples/settings.la): each is a `let` whose type shows only in later uses, which still needs its annotation by design. Left as they are.

### STILL OPEN AT CLOSE
- This session's two commits are not pushed: waiting for the user's go.

---

## SESSION 8 — 2026-10-17 — PRP-008: header look, kivuli 0.3.0 — closed

Branch: main (PRP-008 on prp-008-header_look, merged)

### WHAT WAS DONE
- Read the themes' CSS: GTK Default-dark and Yaru-dark (its gtk.gresource) draw the line under a header bar as a 1 px bottom border, not a shadow; the screenshot's 2 px is that pixel on a scaled display.
- PRP-008 approved and built: `.header_style([…])` on the root, `Surface.flat`; examples/frame.la's header and rail share one flat Frame look. No lugha-gtk change. kivuli 0.3.0, tagged locally.
- DECISION-018 opened (views.la at 499 lines: how the next modifier fits); DECISION-017 got a lead (GTK's theme picks dark by a CSS media query on its provider).

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main (session 7's and this session's commits) and tag v0.3.0.
- For the user to check by hand: `lughac run examples/frame.la`: the header bar and rail are one #242424 surface with no line between them, focused or not.
- DECISION-018 blocks the next new modifier.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 9 — 2026-10-17 — PRP-009: corner radius, kivuli 0.4.0 — closed

Branch: main (PRP-009 on prp-009-corner_radius, merged)

### WHAT WAS DONE
- Probed GTK 4.22 (C, on a private gtk4-broadwayd display): a box's radius rounds its own background but doesn't clip its children; `gtk_widget_set_overflow(HIDDEN)` clips them to the same per-corner curve.
- PRP-009 approved and built: per-corner radius fields in `Surface`, merged per corner; every corner rule for a used radius generated; rounded nodes clip their content (overflow hidden, undone only where kivuli set it). lugha-gtk 0.1.24 pushed (set/get_overflow). examples/frame.la: rail on the right, page curved only top right. kivuli 0.4.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.4.0.
- For the user to check by hand: `lughac run examples/frame.la`: the page's top-right corner curves into the frame; the other three are square.
- DECISION-018 (views.la at 499 lines) still blocks the next modifier; test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 10 — 2026-10-17 — Fix: center a button's content, kivuli 0.4.1 — closed

Branch: main (the fix on fix-button-content-center, merged)

### WHAT WAS DONE
- Bug (rapidkoi DECISION-020 item 0): kivuli's button content box kept GTK's fill alignment, so wide or tall buttons showed their content at the start. A failing GTK test first (a `.width(200)` icon button, a stretched one, an icon+label one: content box halign/valign), then the fix in src/build.la: the box is centered both ways (`GTK_ALIGN_CENTER` = 3, gtkenums.h). Mutation-checked. No new binding (lugha-gtk had set/get halign/valign and gtk_button_get_child). kivuli 0.4.1, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.4.1.
- For the user to check by hand: natural-size buttons look as before; wide ones (Rapidkoi's tabs and tiles) center their content.
- DECISION-018 still blocks the next modifier; test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 11 — 2026-10-18 — PRP-010: images from any source, kivuli 0.5.0 — closed

Branch: main (PRP-010 on prp-010-images, merged)

### WHAT WAS DONE
- Talked through the brief ("images from bytes") first. The user wants one image model that can grow to web images. Chosen: model (B): images show bytes they already have; the app fetches over the network with a non-blocking command (a later PRP, DECISION-007), never kivuli behind a view's back.
- PRP-010 written after an interview (DECISION-018 → map's body in mapping.la; `image(Image.…)` one widget, breaking `image(path)`; PNG+JPEG header-checked for every source; `.size(n)` on buttons) and approved. Built: src/pictures.la (pure checks), src/textures.la (GTK), View.Image(Image), images on the Button node, SetImage/SetButtonImage. lugha-gtk 0.1.25 pushed (GBytes, GdkTexture, paintables). test/live_pictures.la with a genuine 4097-px PNG fixture; mutation-checked. Gallery shows bytes. kivuli 0.5.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.5.0.
- Next by plan: non-blocking commands (Cmd.fetch, file read/write; DECISION-007), so apps fetch web images (model B).
- DECISION-020 (lughac: std.fs file size) reported for the lugha session. test/live_forms.la split still awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 12 — 2026-10-18 — PRP-011: background tasks, kivuli 0.6.0 — closed

Branch: main (PRP-011 on prp-011-background_tasks, merged)

### WHAT WAS DONE
- Read Lugha's spec and runtime (read only): one OS thread for all green threads; GTK's main loop is one extern call, so spawned threads never run in a kivuli app today; `task.yield_now()` is non-blocking (wakes sleepers, polls sockets with epoll_wait 0, runs ready threads one round). So a GLib source driving yield_now; OS threads impossible. File calls, DNS, std.crypto, std.process still block (one extern call each).
- Talked through with the user: build now (option 1), Cmd.task (spawn + channel), and Sub.receive(chan, f) in this PRP so apps use Lugha's spawn, channels and actors. PRP-011 drafted.
- PRP-011 approved and built: Cmd.task, Sub.receive, a turn source in live.la (4 ms tick, yield while work is found, up to 2 ms). test/live_tasks.la (sleep answered promptly, HTTP download + server, CPU sharing, receive in order and only while subscribed; mutation-checked). examples/download.la (its own slow localhost server, spinner, prime counter). kivuli 0.6.0, tagged locally. Resolves Rapidkoi's DECISION-021 (option A) in that repo.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.6.0.
- For the lugha session: DECISION-020 (std.fs file size), DECISION-021 (runtime hooks), DECISION-022 (blocking calls off the one thread).
- For the user to check by hand: `lughac run examples/download.la`.
- test/live_forms.la split still awaiting the user. `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 13 — 2026-10-18 — PRP-012: wrapping grid, kivuli 0.7.0 — closed

Branch: main (PRP-012 on prp-012-wrapping_grid, merged)

### WHAT WAS DONE
- Interview (keys: both; widths: natural with `.equal()`; `.max_per_line` only; no selection, focus inside), PRP-012 approved and built: two file splits first (lazies.la, controls.la); `wrap`, `keyed_wrap` on a GtkFlowBox; lugha-gtk 0.1.26 pushed (flow boxes, focusable, parent). test/diff_wrap.la, test/live_wrap.la (GTK order, settings, keyed moves keep widgets; mutation-checked). frame.la home tiles. DECISION-016 resolved. kivuli 0.7.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.7.0.
- For the user to check by hand: `lughac run examples/frame.la`: resize and the tiles re-flow.
- Queued: file dialog (013), overlay (014), context menus (015). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 14 — 2026-10-18 — PRP-013: overlay, kivuli 0.8.0 — closed

Branch: main (PRP-013 on prp-013-overlay, merged)

### WHAT WAS DONE
- Interview (`overlay(base, layers)`; `.place(across, down)` with Align; no built-in click-outside), PRP-013 approved and built. lugha-gtk 0.1.27 pushed (overlays). patch.la passed 500 while building: the user chose src/paths.la (committed first). test/diff_overlay.la, test/live_overlay.la (mutation-checked). Found and fixed a PRP-007 bug: replacing a header view freed a text view after it. frame.la: an Info panel over the page, a scrim layer closes it. DECISION-014 resolved. kivuli 0.8.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.8.0.
- For the user to check by hand: `lughac run examples/frame.la`: Info opens a panel over the page; clicking the dimmed page closes it.
- Queued: file dialog (013), context menus (015), Rapidkoi's clickable rows (its DECISION-020 item 0). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 15 — 2026-10-18 — PRP-014: context menus, kivuli 0.9.0 — closed

Branch: main (PRP-014 on prp-014-context_menus, merged)

### WHAT WAS DONE
- Interview (`.menu([item(…), menu_separator(), …])`, `.disabled(true)` on items; right-click, long press, Menu key / Shift+F10; `pick(view, owner, item)`), PRP-014 approved and built. Room made first: the node-rewriting modifiers' bodies moved to mapping.la (own commit). lugha-gtk 0.1.28 (menus, actions, popovers, gestures) and 0.1.29 (`GdkRectangle`'s fields public) pushed. New src/menus.la (pure), src/popovers.la (GTK); `View.Menu`, `Live.Menu` (looked through by paths), `SetMenu`. Tests: test/menus.la, test/diff_menu.la, test/lib.la, test/testing.la, test/map.la, test/live_menu.la. frame.la: tab menus (Duplicate, Close, Close other tabs). DECISION-015 resolved. kivuli 0.9.0, tagged locally.
- Found while testing: a hidden window keeps its focus widget alive after removal (checked in C, GTK 4.22; not a kivuli leak); the live test moves focus first.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.9.0.
- For the user to check by hand: `lughac run examples/frame.la`: right-click a tab (and Shift+F10 on a focused tab): Duplicate, Close, Close others (greyed out with one tab); the menu opens at the pointer.
- Queued: file dialog (013), Rapidkoi's clickable rows (its DECISION-020 item 0). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 16 — 2026-10-18 — PRP-015: clickable rows, kivuli 0.10.0 — closed

Branch: main (PRP-015 on prp-015-clickable_rows, merged)

### WHAT WAS DONE
- Interview (`button_with(content, msg)`, a real GtkButton; tests find it by tooltip, `.name` or text inside; interactive content stops the app), PRP-015 approved and built. Split first: src/buttons.la (button making, from build.la). `View.ButtonWith`, `Live.ButtonWith` (content is child 0), `walk.check_content` (checked by the paint pass and every walk). Tests: test/lib.la, test/diff_button_with.la, test/testing.la, test/map.la, test/keys.la, test/theming.la, test/live_button_with.la. frame.la: two Info panel rows. kivuli 0.10.0, tagged locally. No lugha-gtk change.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.10.0.
- For the user to check by hand: `lughac run examples/frame.la`, Info: each row highlights as a whole, clicking anywhere on it works, Tab focuses it and Enter presses it; Go back is greyed out on a tab's first page.
- Rapidkoi can now replace its arrow-only rows (its DECISION-020 item 0; not touched from here).
- Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 17 — 2026-10-19 — PRP-016: styles as values (blocked on lughac) — closed

Branch: prp-016-style_values (not merged)

### WHAT WAS DONE
- The user chose one door for styling (DECISION-023): `.style([...])` of typed values; named looks, `.bold`, `.dim`, text `.size`, `.button_style` deleted (breaking, only what was discussed). Interview in conversation: `Font` and `Text` groups (kept apart), `Theme` variants meaning "not set", the last setter wins, no registry (values on the node, reuse through functions). Utilities deferred: their names clash with the fields (E0310).
- PRP-016 approved and narrowed (bedrock only; old examples to be deleted, new ones written). Built so far: src/typography.la (Styling, Sheet, Font, Text, enums), src/sheets.la (checks, atomic CSS, font names guarded against CSS injection), theming.la cut to Color/Paint, Style holds a Sheet, paint and styling rewritten, live keeps one growing stylesheet. test/sheets.la passes (7).
- Blocked: lughac doesn't convert a struct update literal (`S { f: e, ..base }`) to an interface value where one is expected (DECISION-024). Every `.style([Font { …, ..Font.theme() }])` hits it.

### STILL OPEN AT CLOSE
- DECISION-024: fix in ../lugha (not from a kivuli session), then the user reinstalls lughac.
- Then on the branch: test/typography.la and test/theming.la should compile; migrate the other tests (lib, diff, lazy, live_style, live_look); write test/live_type.la; delete the old examples, write the new ones; docs; 0.11.0.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 18 — 2026-10-19 — PRP-016: styles as values, kivuli 0.11.0 — closed

Branch: main (PRP-016 on prp-016-style_values, merged)

### WHAT WAS DONE
- DECISION-024 fixed in lughac by the user; the blocked tests compiled and passed. Tests migrated to `.style` (looks tests deleted; header-bar tests moved to test/live_header.la); new test/live_type.la (classes, a growing stylesheet, colours read back from GTK; mutation-checked). A style's classes also go on kivuli's own label in a button or check box (CSS doesn't inherit `text-decoration`).
- Old examples deleted; new: hello, type, tasks, browser, controls (each of the last three with tests), split, bench.
- Clean-up: eleven example binaries had been committed in PRP-014 (99cd972, pushed with v0.9.0) by `git add -A`; removed from the tree, history kept (the user's call). The user deleted the binaries on disk. Examples now build with `-o examples/<name>`; .gitignore lists the new ones.
- Docs: README (example, styles section), DESIGN (the class table), CHANGELOG 0.11.0 under Breaking (with Rapidkoi's migration), CLAUDE.md, MEMORY, DECISIONS (023 resolved for fonts and text, 024 resolved); the tutorial artifact republished. kivuli 0.11.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.11.0.
- For the user to check by hand: `lughac run examples/type.la` (each value as its line names it), `examples/browser.la`, `examples/hello.la`.
- Next: style utilities (names free of the fields, E0310), then backgrounds, borders, corners and button looks through `.style`. Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 19 — 2026-10-19 — PRP-017: backgrounds, kivuli 0.12.0 — closed

Branch: main (PRP-017 on prp-017-backgrounds, merged)

### WHAT WAS DONE
- Interview (colour, gradients and images; `Linear`/`Radial` with `Stop`s; `Image` sources reused, `Fill` Cover/Contain/Tile/Stretch, centred; pictures checked and handed to GTK as bytes), PRP-017 approved and built. Rename first: src/typography.la → src/styles.la. `Background`, `Gradient`, `Stop`, `Fill`; `Sheet.background`; base64 and `data:` URLs in pictures.la (RFC 4648 vectors); background rules, picture rules (built by live from checked bytes), checks (stops, opacity, icon-name injection). Every generated rule kind parsed by GTK 4.22 with no errors (C probe). Tests: styles, sheets, pictures, theming, live_type. examples/backgrounds.la. Docs and the tutorial page updated. kivuli 0.12.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0, v0.12.0.
- For the user to check by hand: `lughac run examples/backgrounds.la` in light and dark; also examples/type.la, browser.la, hello.la from 0.11.0.
- Next: style utilities (names free of the fields, E0310); borders, corners and button looks through `.style`. Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 20 — 2026-10-19 — PRP-018: borders and outlines, kivuli 0.13.0 — closed

Branch: main (PRP-018 on prp-018-borders_and_outlines, merged)

### WHAT WAS DONE
- Interview (each side on its own, plus `all`; a side's own field beats `all` within a value, a later value wins across; outlines hide while the node has keyboard focus), PRP-018 approved and built. `Border`, `Side`, `Outline`, `Width`, `Line`, `Radius`; `Sheet.border`, `Sheet.outline`; per-side atomic CSS (a width without a line drawn solid), per-corner radius, outline rules under `:not(:focus-visible)` (GTK parses them, C probe); rounded nodes clip their content again (overflow hidden, `kv-clip`). Tests: styles, sheets_border (new: sheets.la was at 401), theming, live_type. examples/borders.la. Docs and the tutorial page updated. kivuli 0.13.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0, v0.12.0, v0.13.0.
- For the user to check by hand: `lughac run examples/borders.la` in light and dark; Tab to the outlined buttons: the focus ring shows instead of the outline.
- Next: button looks through `.style`; style utilities (names free of the fields, E0310). Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 21 — 2026-10-19 — Spacing (padding, margin): discovery — open

Branch: main

---

## NEXT SESSION START POINT

1. Open a new session in this file (step 1, before anything else) and commit it.
2. Read CLAUDE.md, MEMORY.md, DECISIONS.md, then this file's last session; go by `git log` for what's merged.
3. kivuli 0.1.0 is published (github.com/kithinjibrian/kivuli); PRPs 001–005 are merged. Pushing kivuli needs the user's go each time; lugha-gtk has a standing go.
4. Gotchas: a named import binds the module name too (E0311); `lughac fmt` reflows text scripted edits expect: re-read first; files may not exceed 500 lines; gate any push on the test run's real exit status.
