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

## SESSION 3 — 2026-10-17 — Step 5: modifiers and styles — open

Branch: main

Notes so far (handoff to be written at close):
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

---

## NEXT SESSION START POINT

1. Open session 3 in this file (step 1, before anything else) and commit it.
2. Read CLAUDE.md, MEMORY.md, DECISIONS.md, then this file's session 2. The user works hands-on: small steps, show and explain the code, ask before large changes. Never build lughac; ask the user to.
3. First question for the user: write PRP-001 for the rest of v1 (steps 5–8), or keep building hands-on without one and relax the PRP rule? Attach a recommendation.
4. Step 5 — modifiers and styles: `.padding(n)`, `.spacing(n)`, `.expand()`, `.align(Start|Center|End)`, `.width/height(n)`, `.size(n)`, `.bold()`, `.dim()`, `.style(Normal|Primary|Danger|Subtle)`, `.disabled(bool)` (button `Option<Msg>`, MEMORY decision 6). Read docs/DESIGN.md first (style vocabulary → GTK CSS classes). Each node needs a layout record; modifiers return changed copies with struct update (`..b`). GTK calls they need (margins exist; CSS classes, `gtk_widget_set_sensitive`, halign/valign, size_request, box spacing) go into lugha-gtk first, pushed only on the user's go.
5. Then steps 6–8: `application` with Cmd/Sub, `lazy` (DECISION-002), testing helpers (DECISION-003), examples (to-do, stopwatch) and README.
6. Gotchas: a named import binds the module name too (E0311 for a local named `live`, `views`, …); `lughac fmt` reflows multi-line text other edits expect — re-read before scripted edits; files may not exceed 500 lines.
