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

## SESSION 21 — 2026-10-19 — PRP-019: spacing as a style, kivuli 0.14.0 — closed

Branch: main (PRP-019 on prp-019-spacing, merged)

### WHAT WAS DONE
- Found that `.padding(n)` was a margin (GTK widget margins). Interview: padding and margin in `.style` as `Spacing`, shaped like `Border` (`Edges` with `all` and each side); the gap moves in too; `.padding` and `.spacing` removed (breaking). PRP-019 approved and built. Splits first: src/style_values.la (plain value types) and src/box_rules.la (box CSS), `permille` to theming. `Edges`, `Spacing`, `Sheet.spacing`; padding and margin as CSS (negative margins), the gap through GTK's spacing properties; kivuli no longer sets widget margins. Tests: spacing (new), sheets_border, theming, live_style, live_wrap; every `.padding`/`.spacing` use migrated. Examples use `pad(n)`/`gap(n)` helpers; borders.la shows padding and margin. Docs and the tutorial page updated. kivuli 0.14.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.14.0.
- For the user to check by hand: `lughac run examples/borders.la`'s "Padding and margin" section; any example: the room looks as before.
- Next: button looks through `.style`; style utilities (names free of the fields, E0310). Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 22 — 2026-10-19 — PRP-020: sizing, kivuli 0.15.0 — closed

Branch: main (PRP-020 on prp-020-sizing, merged)

### WHAT WAS DONE
- Found GTK 4.22's CSS has only `min-width`/`min-height` (no width, max-width, max-height), and size requests are minimums. Interview: a clamp widget in lugha-gtk for exact and maximum sizes; `.width`/`.height` (minimums) move into `Sizing` (breaking). PRP-020 approved and built.
- lugha-gtk 0.1.30 pushed: `src/c/clamp.c` (one child, measured within limits, never below its minimum; mutation-checked), `lgtk_clamp_*`, `gtk_widget_measure`.
- kivuli: `Sizing`, `Sheet.sizing`; minimums as CSS; `needs_clamp`/`clamp_limits` (CSS's combining rule); `Live.Clamp` (looked through by paths; `at_outer` for restyles); the diff rebuilds a node gaining or losing the clamp; layout props move onto the clamp. Tests: sizing, sheets_border, diff_sizing, live_sizing (new); every `.width`/`.height` migrated. Examples use `at_least(w, h)`; borders.la shows sizes. Docs and the tutorial page updated. kivuli 0.15.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.15.0.
- For the user to check by hand: `lughac run examples/borders.la`'s "Sizing" section: resize the window; the readable column stops at 560 px.
- Next: button looks through `.style`; style utilities (names free of the fields, E0310). Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 23 — 2026-10-20 — PRP-021: states, kivuli 0.16.0 — closed

Branch: main (PRP-021 on prp-021-states, merged)

### WHAT WAS DONE
- Interview (one `When(state, [...])`), PRP-021 approved and built fast, per the user: narrow test runs only, no full suite. `State` (Hover, Press, Focus, Disabled, Checked), `StateStyle`, `When`, `Sheet.states`; each state's rules under its pseudo-class with its class repeated to its rank (Hover < Focus < Press < Checked < Disabled; GTK parses them, C probe); an outline in `Focus` is the app's focus ring; image, exact/max size and gap refused in a state, as is nesting. Tests: states (new), live_type, and the files covering Sheet/rules/paint. borders.la has a "States" section. Docs and the tutorial page updated. kivuli 0.16.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.16.0.
- The full suite wasn't run this session (the user's call); run it before a push.
- For the user to check by hand: `lughac run examples/borders.la`'s "States" section: hover, press, Tab, the disabled and ticked looks.
- Next: button looks and theme colours in `Paint`; shadows, effects, motion; style utilities. Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 24 — 2026-10-20 — Theme colours and button looks: skipped — closed

Branch: main

### WHAT WAS DONE
- Discussed theme colours in `Paint` (`Paint.Accent` and GTK's other named colours) and button looks (the theme's own classes, or kivuli presets). The user's call: skip both; apps build their looks from their own values (a palette is ordinary functions). No code, no PRP.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.16.0; the full suite before any push.
- Next: shadows and effects (session 25).

---

## SESSION 25 — 2026-10-20 — PRP-022: shadows and effects, kivuli 0.17.0 — closed

Branch: main (PRP-022 on prp-022-shadows_and_effects, merged)

### WHAT WAS DONE
- Interview (one `Shadow` with box and text lists of `Drop`s; one `Effects` with opacity, filters, transforms), PRP-022 approved with a second allowed import cycle, `styles` ↔ `style_groups` (DECISION-025; CLAUDE.md rule 2). `Drop`, `Filter`, `Move` (style_values); `Shadow`, `Effects` (style_groups); `Sheet.shadow`, `Sheet.effects`; CSS one rule per list, checks, all of it in `When`; `Shadow.off()`. `paint_css` and `hexed` moved to theming. GTK parsed every generated kind (C probe). Narrow runs only: shadows (new), styles, states, sheets, sheets_border, theming, spacing, sizing, live_type; every example built. borders.la has a "Shadows and effects" section. Docs and the tutorial page updated. kivuli 0.17.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.17.0. The full suite hasn't run since 0.15.0: run it before any push.
- For the user to check by hand: `lughac run examples/borders.la`'s "Shadows and effects": the lifted card, the tile growing on hover, the greyscale and blurred pictures, the tilted text.
- Next: motion (transitions, animations); style utilities. Queued: file dialog (013). For the lugha session: DECISIONS 020-022. test/live_forms.la split awaiting the user.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 26 — 2026-10-20 — Clean-up of old code — closed

Branch: main (cleanup, merged)

### WHAT WAS DONE
- Scanned src for unused functions (none) and unused imports (13, left by the styling removals: removed). Fixed stale doc comments (`button_with`'s `.button_style` and `.dim()`; CLAUDE.md anti-pattern 7's `.padding`). Regenerated every module header's "Depends on" and "Used by" from its real imports (25 had drifted).
- The full suite, the first since 0.15.0: 275 tests passed; the examples' 16 passed; every example built; fmt clean. Merged into main. No version change (no behaviour change).

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.11.0 through v0.17.0 (the full suite is green now).
- test/live_forms.la is 562 lines (over 500): its split still waits for the user. DECISION-007 (non-blocking I/O) still says deferred though PRP-011 shipped `Cmd.task`: the user may want it closed.
- Next: motion (transitions, animations); style utilities; the file dialog (013). For the lugha session: DECISIONS 020-022.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 27 — 2026-10-20 — Style utilities: parked — closed

Branch: main

### WHAT WAS DONE
- Started the style-utilities interview. Question 1 (where they live): A, a `kivuli.tw` module of free functions with Tailwind-style names (`tw.bold()`, `tw.p(16)`, `tw.rounded(12)`; module-qualified calls checked with lughac), or B, statics on each group with names avoiding the fields (E0310). The user parked it before answering. No code, no PRP.

### STILL OPEN AT CLOSE
- Style utilities parked at question 1.
- Next, whenever the user picks: motion (transitions, animations); the file dialog (013); test/live_forms.la split (562 lines); DECISION-007 to close. For the lugha session: DECISIONS 020-022.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 28 — 2026-10-20 — PRP-023: the header bar as a node, kivuli 0.18.0 — closed

Branch: main (PRP-023 on prp-023-header_bar, merged)

### WHAT WAS DONE
- The user asked whether the header bar can be styled; chose the bar as a node over a second `.header_style` door. PRP-023 approved and built, narrow runs only (the user's call): `header_bar(start, title, end)` gives a `Bar` with `.style([...])`; `.header(bar)` (breaking); the bar's style rides in the root's `Style` (`bar`, `bar_classes`), checked by paint (size, gap and image refused), on the bar only. Tests: lib, theming, live_header (new test), plus every `.header` use migrated; browser.la styles its bar. Docs updated. kivuli 0.18.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.18.0. The full suite wasn't run (the user's call): run it before the push.
- For the user to check by hand: `lughac run examples/browser.la`: the bar's faint tint and hairline, the window controls still GTK's.
- Next: motion; style utilities (parked); the file dialog (013); test/live_forms.la split; DECISION-007 to close.
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 29 — 2026-10-20 — PRP-024: clicking away clears focus, kivuli 0.19.0 — closed

Branch: main (PRP-024 on prp-024-click_away, merged)

### WHAT WAS DONE
- The user's `examples/search_bar.la` kept its field focused when clicking away: GTK's behaviour (a window keeps a focused widget; a click moves focus only onto something that takes it), not autofocus repeating. Interview: always on. PRP-024 approved and built: lugha-gtk 0.1.31 (`gtk_window_set_focus`, pushed); a primary-button bubble-phase click gesture on the window clears its focus (`Running.blur`). Test: live_focus (new, mutation-checked); live_forms, live_menu, live still pass; every example built. Docs updated. kivuli 0.19.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tag v0.19.0. The full suite wasn't run: run it before the push.
- For the user to check by hand: `lughac run examples/search_bar.la`: click beside the field and it loses focus; buttons and other widgets still take their clicks.
- The user's examples/search_bar.la is untracked (theirs; its app id is hello's, "org.kivuli.hello"); my build loop left examples/search_bar (a binary, not in .gitignore; protected, left alone).
- `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 30 — 2026-10-20 — Style utilities, resumed: discovery and PRP-025 draft — closed

Branch: main

### WHAT WAS DONE
- lughac now allows a field and a function of one name, so the style utilities are unblocked. Interview: statics on each group (A), word names (B). PRP-025 drafted.
- The user asked why kivuli has no folders: only history; Lugha maps `src/styles/fonts.la` to `styles.fonts`, and a file and a folder may share a name (checked in a scratch package). The user chose folders: PRP-025's file plan is now `src/styles/{fonts,boxes,effects}.la`, with `styles.la` keeping `Styling`, `Sheet`, `applied`, `State`, `When`, and DECISION-025's cycle widened to `styles` ↔ `styles/`.

### STILL OPEN AT CLOSE
- PRP-025 awaits the user's approval of the whole PRP (the utility list and values).
- Not pushed, waiting for the user's go: main and tags v0.18.0, v0.19.0. Run the full suite before the push.
- The user's examples/search_bar.la and its binary are untracked; `reports/.gitkeep` deleted in the working tree, not by Claude; left alone.

---

## SESSION 31 — 2026-10-20 — PRP-025: style utilities, kivuli 0.20.0 — closed

Branch: main (PRP-025 on prp-025-style_utilities, merged)

### WHAT WAS DONE
- PRP-025 approved and built. The style groups moved into `src/styles/` (`fonts`, `boxes`, `drawing`; `drawing`, not `effects`, which clashed with the `effects` module as E0314); `styles.la` keeps `Styling`, `Sheet`, `applied`, `State`, `When`; DECISION-025 widened to `styles` ↔ `styles/`. Static utilities on every group (`Font.h2()`, `Text.dim()`, `Spacing.padding(16)`, `Border.rounded(12)`, `Shadow.md()`, `Effects.opacity(0.5)`, …). Test: utilities (new, 10); styles, states, spacing, sizing, shadows, sheets, sheets_border, theming, lib still pass; every example built. hello and borders use utilities. Docs updated. kivuli 0.20.0, tagged locally.

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.18.0, v0.19.0, v0.20.0. The full suite wasn't run: run it before the push.
- For the user to check by hand: `lughac run examples/borders.la`: the new "Shadow.md(), in one line" card.
- Next: motion; the file dialog (013); test/live_forms.la split; DECISION-007 to close.
- The user's examples/search_bar.la is untracked; `.gitignore` and `reports/.gitkeep` changes are the user's; left alone.

---

## SESSION 32 — 2026-10-20 — src/ in folders — closed

Branch: refactor-folders (not merged: waiting for the full suite)

### WHAT WAS DONE
- The user approved a folder layout: `src/view/`, `src/styles/` (+ values, theming, sheets, box_rules, paint), `src/core/` (diff, effects, live_tree, paths), `src/render/` (the GTK callers); lib, testing and styles stay at the root (DECISION-026). Imports rewritten by script; every example builds; every test file type-checks; fmt clean.
- `test/` stays flat: a test in a subfolder can't import `test/fixtures.la` (DECISION-027, open, needs lughac).
- The user stopped the full suite run (it ran in the background); they have the command to run it themselves.

### STILL OPEN AT CLOSE
- Merge `refactor-folders` into main once the full suite passes (`lughac test test/ examples/tasks.la examples/browser.la examples/controls.la`).
- Not pushed: main and tags v0.18.0–v0.20.0.
- DECISION-027: a lughac change, then `test/` into folders.

---

## SESSION 33 — 2026-10-20 — Merge src/ in folders — closed

Branch: main

### WHAT WAS DONE
- Merged `refactor-folders` into main on the user's go (the full suite's result wasn't reported here; the command was handed over in session 32).

### STILL OPEN AT CLOSE
- Not pushed, waiting for the user's go: main and tags v0.18.0, v0.19.0, v0.20.0. Run the full suite before the push.
- DECISION-027: a lughac change, then `test/` into folders.
- Next: motion; the file dialog (013); test/live_forms.la split (562 lines); DECISION-007 to close.

---

## SESSION 34 — 2026-10-20 — Components (shadcn-like): discovery, PRP-026 draft — closed

Branch: main

### WHAT WAS DONE
- The user asked for beautiful components (shadcn) beside the native look. Plan: tokens and palettes, then motion, then a first `ui` component set. Interview: a library module `kivuli.ui` (A); named tokens with a palette per app (A); the palette colours the window, native widgets stay GTK's (A); follow the system, `.scheme(…)` forces (A); five shadcn bases, `.with`, a radius token (A).
- GTK 4.22 probe (C): custom properties, `prefers-color-scheme`, `prefers-reduced-motion`, `color-mix`, transitions, `calc(var(…) - 4px)` all parse; `gtk-interface-color-scheme` reads 2 (default).
- PRP-026 (palettes and tokens) drafted.

### STILL OPEN AT CLOSE
- PRP-026 awaits the user's approval.
- Not pushed: main and tags v0.18.0–v0.20.0; run the full suite first.

---

## SESSION 35 — 2026-10-20 — PRP-026: palettes and colour tokens, built, not merged — closed

Branch: prp-026-palettes (not merged: its tests haven't run)

### WHAT WAS DONE
- lugha-gtk 0.1.32: `lgtk_set_color_scheme`, `lgtk_reset_color_scheme`, `lgtk_color_scheme` (src/c/settings.c); settings and clamp tests pass; pushed and tagged under the standing go (the user stopped its full run).
- PRP-026 built: `Token` as `Paint.Token` (`var(--kv-…)`), `Colorish` (Color, Token, Paint), `Palette` (five shadcn bases, `.with`, `.radius`), `Radius.Small/Medium/Large` and `Border.rounded_sm/md/lg`, `.palette` and `.scheme` (root only), the root's `kv-pal-…` class with its dark block, zinc's tokens on every window, `render/schemes.la`. `stylesheet()` gives whole rules. examples/palette.la. Docs; kivuli 0.21.0 (not tagged).
- The user is fixing slow test runs: tests written and type-checked only (palettes, live_palette, theming, live_type).
- A stray `git push` in kivuli ran by mistake in a chained command; it failed (no upstream), nothing left the machine.

### STILL OPEN AT CLOSE
- Run palettes, live_palette, theming, live_type (and the full suite), then merge prp-026-palettes and tag v0.21.0.
- `src/render/live.la` is at 500 lines: propose a split before it grows.
- Not pushed: main and tags v0.18.0–v0.20.0.
- `.gitignore`'s `/examples/search_bar` line is the user's, uncommitted.

---

## SESSION 36 — 2026-10-20 — PRP-026 fix: dark mode never applied — closed

Branch: prp-026-palettes (not merged)

### WHAT WAS DONE
- The user ran examples/palette.la: palettes and the brand colour worked, Dark changed nothing. Cause (C probe): an app's GtkCssProvider keeps `prefers-color-scheme` at "default" whatever GTK's setting; binding it to `gtk-interface-color-scheme` fixes it, live.
- lugha-gtk 0.1.33: `lgtk_css_provider_new_for_scheme`, `lgtk_css_provider_color_scheme`; settings and clamp tests pass; pushed and tagged.
- kivuli's stylesheet is made with it (live.la stays at 500 lines); live_palette checks the stylesheet turns dark (type-checked). The Yaru-dark warnings are the system theme's (its gtk.css imports a resource this GTK lacks), not kivuli's.

### STILL OPEN AT CLOSE
- The user to rerun `lughac run examples/palette.la` and try Dark.
- Run palettes, live_palette, theming, live_type, then merge prp-026-palettes and tag v0.21.0.
- `src/render/live.la` at 500 lines: split proposed (render/windows.la), waiting for the user.
- Not pushed: main and tags v0.18.0–v0.20.0.

---

## SESSION 37 — 2026-10-20 — Merge PRP-026; split render/live.la — closed

Branch: main (the split on refactor-live-split, merged)

### WHAT WAS DONE
- The user saw dark mode work and said to merge now: PRP-026 merged without running its tests (the user's call), tagged v0.21.0 locally. The user's uncommitted `.gitignore` line was stashed for the checkout and restored.
- `src/render/live.la` (500 lines) split: `window_of`, `show_window`, `show_maximized` and `click_away` moved to `src/render/windows.la`; live.la is 439 lines. No behaviour change: examples build, the live tests touching it type-check, fmt clean.

### STILL OPEN AT CLOSE
- Tests not run since session 32 (the user is fixing slow test runs): run the full suite, then push main and tags v0.18.0–v0.21.0 on the user's go.
- Next in the shadcn plan: motion (transitions), then the first `kivuli.ui` component set.

---

## SESSION 38 — 2026-10-20 — Motion: discovery, PRP-027 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: transitions plus named animations (B); a `Motion` group with property-group utilities, easing and delay (A); reduced motion stops decorative motion, keeps spin and pulse (A); entrances only, played on a new widget (A).
- GTK 4.22 probe: transitions (lists, curves, delays, steps), `@keyframes`, `prefers-reduced-motion` all parse; `gtk-interface-reduced-motion` exists (4.22), and a provider's `prefers-reduced-motion` needs binding like the colour scheme.
- PRP-027 (motion) drafted.

### STILL OPEN AT CLOSE
- PRP-027 awaits the user's approval.
- Tests not run since session 32; main and tags v0.18.0–v0.21.0 not pushed.

---

## SESSION 39 — 2026-10-20 — PRP-027: motion, built, not merged — closed

Branch: prp-027-motion (not merged: waiting for the user's hand check)

### WHAT WAS DONE
- lugha-gtk 0.1.34: reduced motion (set, reset, read; the scheme provider follows `prefers-reduced-motion` too); tests pass; pushed and tagged.
- PRP-027 built: `Motion` (styles/motion.la: transitions by group, easing, delay, `fade_in`, `slide_in`, `spin`, `pulse`), its CSS (styles/motion_rules.la: one transition and one animation rule, keyframes, reduced-motion blocks through `rule_text`), checks; `Sheet.motion`. GTK parsed every generated rule (C probe). examples/motion.la. Docs; kivuli 0.22.0 (not tagged).
- Tests written and type-checked only (motion, live_motion), as the user asked.

### STILL OPEN AT CLOSE
- The user to try `lughac run examples/motion.la` (eases, the tile, toasts sliding in, spinner, skeletons; then Reduce Animation on).
- Then merge prp-027-motion and tag v0.22.0; tests to run once the user's fix is in.
- Not pushed: main and tags v0.18.0–v0.21.0.

---

## SESSION 40 — 2026-10-20 — Merge PRP-027 — closed

Branch: main (prp-027-motion merged)

### WHAT WAS DONE
- Merged PRP-027 (motion) on the user's go, without running its tests (the user's call); tagged v0.22.0 locally. The user's `.gitignore` line stashed for the checkout and restored.

### STILL OPEN AT CLOSE
- Tests not run since session 32; main and tags v0.18.0–v0.22.0 not pushed (the user's go, after a full run).
- Next in the shadcn plan: the first `kivuli.ui` component set (a discovery, then a PRP).

---

## SESSION 41 — 2026-10-20 — kivuli.ui, first component: discovery, PRP-028 draft — closed

Branch: main

### WHAT WAS DONE
- The user asked for one component first, the easiest. Interview: Badge (A); variants as style values through `.style` (C); shadcn v4's look, corners from the palette (A).
- PRP-028 (kivuli.ui, Badge) drafted: `kivuli.ui` module, `Look` (a bundle of style values), `badge(label)`, `Badge.secondary/outline/destructive()`.

### STILL OPEN AT CLOSE
- PRP-028 awaits the user's approval.
- Tests not run since session 32; main and tags v0.18.0–v0.22.0 not pushed.

---

## SESSION 42 — 2026-10-20 — PRP-028: kivuli.ui, Badge, built, not merged — closed

Branch: prp-028-ui-badge (not merged: waiting for the user's look)

### WHAT WAS DONE
- PRP-028 built: `Look`/`look` in styles.la (exported); `kivuli.ui` (src/ui.la re-exporting src/ui/badges.la): `badge(label)`, `Badge.secondary/outline/destructive()`. Component files import the entry's modules directly, not `lib`. examples/components.la (variants, beside text, adjusted, on a card; light/dark, palettes). Docs; 0.23.0 (not tagged).
- test/badges.la written and type-checked only (the user's call).

### STILL OPEN AT CLOSE
- The user to look: `lughac run examples/components.la`; then merge and tag v0.23.0.
- Next component: Button (variants default, secondary, outline, ghost, destructive, link; sizes), on the same convention.
- Tests not run since session 32; main and tags v0.18.0–v0.22.0 not pushed.

---

## SESSION 43 — 2026-10-20 — CLAUDE.md: narrow example builds — closed

Branch: prp-028-ui-badge

### WHAT WAS DONE
- The user asked why every example was rebuilt and wants to work faster. CLAUDE.md's TESTING rule and COMMANDS done-line now say: check only the examples that use what changed, build only the one the user will run, build them all before a push. Saved as memory too.

### STILL OPEN AT CLOSE
- As session 42: the user to look at examples/components.la, then merge PRP-028 and tag v0.23.0.

---

## SESSION 44 — 2026-10-20 — badge_with; no shadcn in comments; no co-author line — closed

Branch: prp-028-ui-badge (not merged)

### WHAT WAS DONE
- The user liked the badges and asked for icons: `badge_with(content)` (A): the badge's look on a row, 4 px apart, icons at 12 px unless sized. Added to PRP-028; a "With icons" section in examples/components.la (only that example built). Tests type-checked.
- Code comments no longer name shadcn (src, examples).
- Commits carry no Co-Authored-By trailer from now on (the user's request; saved as memory). `.claude/settings.json` changed in the working tree by the user: left alone.

### STILL OPEN AT CLOSE
- The user to look at the icon badges; then merge PRP-028 and tag v0.23.0.
- Tests not run since session 32; main and tags v0.18.0–v0.22.0 not pushed.

---

## SESSION 45 — 2026-10-20 — Merge PRP-028 — closed

Branch: main (prp-028-ui-badge merged)

### WHAT WAS DONE
- The user approved the icon badges: PRP-028 merged without running its tests (the user's call), tagged v0.23.0 locally.

### STILL OPEN AT CLOSE
- Next component: Button (a discovery, then a PRP), on the badge's convention.
- Tests not run since session 32; main and tags v0.18.0–v0.23.0 not pushed.

---

## SESSION 46 — 2026-10-20 — kivuli.ui Button: discovery, PRP-029 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: `ui.button` beside the native `button` (A); six variants, four sizes, `button_with` (A); `Token.X.alpha(x)` for hovers and the focus halo (A); disabled at half opacity, no press look, colours and shadow ease 150 ms (A).
- PRP-029 drafted. Found in the design: GTK's own `button:hover/:active/:disabled` rules outrank a plain class, so each variant restates its colours in states.

### STILL OPEN AT CLOSE
- PRP-029 awaits the user's approval.
- Tests not run since session 32; main and tags v0.18.0–v0.23.0 not pushed.

---

## SESSION 47 — 2026-10-20 — PRP-029: kivuli.ui Button, built, not merged — closed

Branch: prp-029-ui-button (not merged: waiting for the user's look)

### WHAT WAS DONE
- `Paint.Alpha` / `Token.X.alpha(x)` (color-mix), its classes and check. `kivuli.ui`'s `button`, `button_with`, `Button`'s six variants and three sizes (src/ui/buttons.la); each variant restates its colours in hover, press and disabled to outrank GTK's button rules. GTK parsed the generated CSS (101 lines). A Button section in examples/components.la (only it built). Docs; 0.24.0 (not tagged).
- Tests (buttons_ui, palettes) written and type-checked only.

### STILL OPEN AT CLOSE
- The user to look at the buttons (hover, press, Tab focus, disabled, light/dark); then merge and tag v0.24.0.
- Tests not run since session 32; main and tags v0.18.0–v0.23.0 not pushed.

---

## SESSION 48 — 2026-10-20 — Button fix: a box inside a box — closed

Branch: prp-029-ui-button (not merged)

### WHAT WAS DONE
- The user's screenshot: styled buttons drew a second box inside and were twice as padded. Cause: `style_label` gave the label every class of its button. Now only text-decoration classes. test/live_buttons_ui.la (type-checked). components.la rebuilt. (Opened late: the fix was made before this entry was written.)
- Then the icon-only button's icon sat at its left edge: `ui.button_with`'s content row filled the button. It's `.align(Align.Center)` now (a button's content aligns across its width). Test updated, type-checked.
- Seen in the screenshot, not kivuli's: native GTK buttons draw as bare text because the system theme (Yaru-dark) can't load in this GTK.

### STILL OPEN AT CLOSE
- The user to look again; then merge PRP-029 and tag v0.24.0.

---

## SESSION 49 — 2026-10-20 — Merge PRP-029 — closed

Branch: main (prp-029-ui-button merged)

### WHAT WAS DONE
- PRP-029 (Button, Token.alpha, the label and centring fixes) merged on the user's go without running its tests; tagged v0.24.0 locally.
- A first try failed at the checkout (an uncommitted `lughac fmt` of test/palettes.la) and, chained with `;`, still committed a false "merged" entry on the branch; it was undone (soft reset, local only) and the formatting committed before merging.

### STILL OPEN AT CLOSE
- Raised to the user: text inside a stretched label is centred (GTK's default) and kivuli can't left-align it; worth a small PRP before Card.
- Next components: Card, Input, … one at a time.
- Tests not run since session 32; main and tags v0.18.0–v0.24.0 not pushed.

---

## SESSION 50 — 2026-10-20 — Text alignment: discovery, PRP-030 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: `Text.start()/center()/end()` style values (A); start as the default everywhere, breaking (B). GTK's CSS has no `text-align`: it's `gtk_label_set_xalign` and `set_justify`, which lugha-gtk lacks.
- PRP-030 drafted.

### STILL OPEN AT CLOSE
- PRP-030 awaits the user's approval.

---

## SESSION 51 — 2026-10-20 — PRP-030: text alignment, built, not merged — closed

Branch: prp-030-text-align (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.35: label xalign and justify; its test passes; pushed and tagged.
- PRP-030 built: `TextAlign`, `Text.align`, `Text.start/center/end()`, merged; refused in states; `styling` sets xalign/justify on text nodes every restyle; default start (breaking). Tests type-checked; components built. Docs; 0.25.0 (not tagged).

### STILL OPEN AT CLOSE
- The user to look (components: headings and the card's text now start); then merge, tag v0.25.0.
- Next component: Card.

---

## SESSION 52 — 2026-10-20 — Merge PRP-030 — closed

Branch: main (prp-030-text-align merged)

### WHAT WAS DONE
- PRP-030 (text alignment) merged on the user's go without running its tests; tagged v0.25.0 locally.

### STILL OPEN AT CLOSE
- Next component: Card (a discovery, then a PRP).
- Tests not run since session 32; main and tags v0.18.0–v0.25.0 not pushed.

---

## SESSION 53 — 2026-10-20 — kivuli.ui Card: discovery, PRP-031 draft — closed

Branch: main

### WHAT WAS DONE
- Read the user's reference (the current shadcn card). Interview: parts as functions (A); one size now, small later when a container's spacing can reach its parts (A).
- PRP-031 drafted (`card`, `card_header`, `card_header_with`, `card_content`, `card_footer`; `Radius.XLarge` proposed).

### STILL OPEN AT CLOSE
- PRP-031 awaits the user's approval.

---

## SESSION 54 — 2026-10-20 — PRP-031: kivuli.ui Card, built, not merged — closed

Branch: prp-031-ui-card (not merged: waiting for the user's look)

### WHAT WAS DONE
- `Radius.XLarge` / `Border.rounded_xl()`; src/ui/cards.la (`card`, `card_header`, `card_header_with`, `card_content`, `card_footer`); the card keeps the theme's text size (scales compound). DECISION-028 open. Card section in examples/components.la (only it built). GTK parsed the CSS. Docs; 0.26.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look; then merge and tag v0.26.0.

---

## SESSION 55 — 2026-10-20 — Merge PRP-031 — closed

Branch: main (prp-031-ui-card merged)

### WHAT WAS DONE
- PRP-031 (Card, Radius.XLarge) merged on the user's go without running its tests; tagged v0.26.0 locally.

### STILL OPEN AT CLOSE
- Next components: Input (`ui.input`), then Separator, Select, Checkbox, Slider … one at a time.
- Tests not run since session 32; main and tags v0.18.0–v0.26.0 not pushed.

---

## SESSION 56 — 2026-10-20 — kivuli.ui Input: discovery, PRP-032 draft — closed

Branch: main

### WHAT WAS DONE
- The reference page shows no class string; the look follows shadcn's published input. Interview: `ui.input` and `Input.invalid()` only, Field next (A); a new `State.FocusWithin` (`:focus-within`), as a GTK field's focus is on the text inside it (A).
- PRP-032 drafted.

### STILL OPEN AT CLOSE
- PRP-032 awaits the user's approval.

---

## SESSION 57 — 2026-10-20 — PRP-032: kivuli.ui Input, built, not merged — closed

Branch: prp-032-ui-input (not merged: waiting for the user's look)

### WHAT WAS DONE
- `State.FocusWithin` (`:focus-within`, rank 3; Press, Checked, Disabled shift up one); src/ui/inputs.la (`input`, `Input.invalid()`), disabled border, background and text restated. GTK parsed the CSS. An Input section in components.la, the login card on `ui.input`. Docs; 0.27.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look (type in a field: the ring; type without "@": invalid); then merge and tag v0.27.0. Next: Field.

---

## SESSION 58 — 2026-10-20 — Input fixes: bare text centred, a heavy ring — closed

Branch: prp-032-ui-input (not merged)

### WHAT WAS DONE
- From the user's screenshot: a bare `text(…)` stayed centred (only styled text was aligned; `build` now places every new label at the start); the focus ring was a dark band (`Ring` was the text colour; now the other scheme's muted text, as the current reference bases have it). Tests updated (palettes), type-checked; components rebuilt. (Opened late: the fixes were made before this entry.)

### STILL OPEN AT CLOSE
- The user to look again; then merge PRP-032 and tag v0.27.0.

---

## SESSION 59 — 2026-10-20 — Merge PRP-032 — closed

Branch: main (prp-032-ui-input merged)

### WHAT WAS DONE
- PRP-032 (Input, State.FocusWithin, the bare-text and ring fixes) merged on the user's go without running its tests; tagged v0.27.0 locally.

### STILL OPEN AT CLOSE
- Next component: Field (a label and a description around a control).
- Tests not run since session 32 (two bugs since were ones the unrun tests cover); main and tags v0.18.0–v0.27.0 not pushed.

---

## SESSION 60 — 2026-10-20 — kivuli.ui Field: discovery, PRP-033 draft — closed

Branch: main

### WHAT WAS DONE
- The user asked about icons in inputs; chose Field first, then the input group (B); GTK's own entry icons later. Read the reference Field page. Interview: parts and a group, `Field.invalid()` (A). Corrected my sketch: horizontal is `field_horizontal`, since a style can't turn a column into a row.
- PRP-033 drafted.

### STILL OPEN AT CLOSE
- PRP-033 awaits the user's approval. Then the input group.

---

## SESSION 61 — 2026-10-20 — PRP-033: kivuli.ui Field, built, not merged — closed

Branch: prp-033-ui-field (not merged: waiting for the user's look)

### WHAT WAS DONE
- src/ui/fields.la (`field`, `field_horizontal`, `field_label`, `field_description`, `field_error`, `field_group`, `Field.invalid()`); a live Field section in components.la and the login card on fields. Docs; 0.28.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.28.0. Next: the input group (the user's order).

---

## SESSION 62 — 2026-10-20 — Merge PRP-033 — closed

Branch: main (prp-033-ui-field merged)

### WHAT WAS DONE
- PRP-033 (Field) merged on the user's go without running its tests; tagged v0.28.0 locally.

### STILL OPEN AT CLOSE
- Next: the input group (the user's order), then GTK's own entry icons, Separator, Select, Checkbox, Slider …
- Tests not run since session 32; main and tags v0.18.0–v0.28.0 not pushed.

---

## SESSION 63 — 2026-10-20 — components.la scrolls; input group discovery, PRP-034 draft — closed

Branch: main

### WHAT WAS DONE
- examples/components.la scrolls (the user's request): the page in a `scroll`, palette and scheme on it.
- Read the reference input group page. Interview: `input_group(start, control, end)` with parts (A); headers, footers and text areas later.
- PRP-034 drafted.

### STILL OPEN AT CLOSE
- PRP-034 awaits the user's approval.

---

## SESSION 64 — 2026-10-20 — PRP-034: kivuli.ui input group, built, not merged — closed

Branch: prp-034-ui-input-group (not merged: waiting for the user's look)

### WHAT WAS DONE
- src/ui/input_groups.la (`input_group`, `group_input`, `group_text`, `group_button`, `group_button_with`, `InputGroup.invalid()`); an Input group section in components.la. GTK parsed the CSS (116 lines). Docs; 0.29.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look; then merge and tag v0.29.0. Next: GTK's entry icons, or Separator, Select, Checkbox, Slider.

---

## SESSION 65 — 2026-10-20 — Merge PRP-034 — closed

Branch: main (prp-034-ui-input-group merged)

### WHAT WAS DONE
- PRP-034 (input group) merged on the user's go without running its tests; tagged v0.29.0 locally.

### STILL OPEN AT CLOSE
- Next: GTK's entry icons, or Separator, Select, Checkbox, Slider (the user to choose).
- Tests not run since session 32; main and tags v0.18.0–v0.29.0 not pushed.

---

## SESSION 66 — 2026-10-20 — Field icons: dropped — closed

Branch: main

### WHAT WAS DONE
- Began a discovery for GTK's built-in entry icons; the user pointed out the input group already gives icons and buttons inside a field's frame. Dropped: no PRP.

### STILL OPEN AT CLOSE
- Next component: Separator, then Checkbox, Select, Slider.

---

## SESSION 67 — 2026-10-20 — kivuli.ui Separator: discovery, PRP-035 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: `ui.separator()` (the native one, which orients by its parent, in `Border`) and `separator_with(label)` (A). Corrected myself: a component can't know its parent, so `separator_with` in a row isn't refused, only documented.
- PRP-035 drafted.

### STILL OPEN AT CLOSE
- PRP-035 awaits the user's approval.

---

## SESSION 68 — 2026-10-20 — PRP-035: kivuli.ui Separator, built, not merged — closed

Branch: prp-035-ui-separator (not merged: waiting for the user's look)

### WHAT WAS DONE
- src/ui/separators.la (`separator`, `separator_with`); a Separator section in components.la. Docs; 0.30.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look; then merge and tag v0.30.0. Next: Checkbox, Select, Slider.

---

## SESSION 69 — 2026-10-20 — Merge PRP-035 — closed

Branch: main (prp-035-ui-separator merged)

### WHAT WAS DONE
- PRP-035 (Separator) merged on the user's go without running its tests; tagged v0.30.0 locally.

### STILL OPEN AT CLOSE
- Next: Checkbox, Select, Slider (native controls whose inner parts GTK draws).
- Tests not run since session 32; main and tags v0.18.0–v0.30.0 not pushed.

---

## SESSION 70 — 2026-10-20 — kivuli.ui Checkbox: discovery, PRP-036 draft — closed

Branch: main

### WHAT WAS DONE
- Probed GTK 4.22: descendant part rules (`.kv-x check`, under a state too), `-gtk-icon-source` for the tick, slider and placeholder parts all parse. Interview: a public `Inside(Part, …)` value (A); the reference checkbox look, with `Mark.icon(name, px)` (yes).
- PRP-036 drafted.

### STILL OPEN AT CLOSE
- PRP-036 awaits the user's approval.

---

## SESSION 71 — 2026-10-20 — PRP-036: Checkbox and widget parts, built, not merged — closed

Branch: prp-036-ui-checkbox (not merged: waiting for the user's look)

### WHAT WAS DONE
- `Part`, `PartStyle`, `Inside`, `Sheet.parts` (styles.la); `Mark` (drawing.la); part rules, state placement before the part, the focus tail fix, mark CSS and checks (sheets.la); paint splits a class at a space. src/ui/checkboxes.la. A Checkbox section in components.la. GTK parsed the CSS. Docs; 0.31.0 (not tagged). Tests type-checked only.
- Found: native check boxes can't be disabled. Raised, not built.

### STILL OPEN AT CLOSE
- The user to look; then merge and tag v0.31.0. The user to decide on disabled check boxes.

---

## SESSION 72 — 2026-10-20 — Split sheets.la; disabled check boxes PRP-037 draft — closed

Branch: prp-036-ui-checkbox (the split; PRP-036 still not merged)

### WHAT WAS DONE
- sheets.la (514 lines) split on the user's approval: its checks are styles/checks.la (239); sheets.la 303. Unused imports trimmed; importers updated; type-checks.
- PRP-037 (disabled check boxes) drafted: `View.Checkbox`'s toggle becomes an `Option`, as rule 6 says.

### STILL OPEN AT CLOSE
- PRP-037 awaits the user's approval. PRP-036 awaits the user's look and a merge (PRP-037 builds on it).

---

## SESSION 73 — 2026-10-20 — Merge PRP-036; PRP-037: disabled check boxes, built, not merged — closed

Branch: prp-037-disabled-checkboxes (not merged)

### WHAT WAS DONE
- PRP-036 merged on the user's go (tests not run), tagged v0.31.0.
- PRP-037 built: `View.Checkbox`'s toggle is `Option`; `.disabled` takes it; build and `SetOnToggle` set sensitivity; `kivuli.testing.toggle` gives `Silent`. A disabled check box in components.la. 0.32.0 (not tagged). Tests type-checked only.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.32.0. Next: Select or Slider.
- Tests not run since session 32; main and tags v0.18.0–v0.31.0 not pushed.

---

## SESSION 74 — 2026-10-20 — Merge PRP-037 — closed

Branch: main (prp-037-disabled-checkboxes merged)

### WHAT WAS DONE
- PRP-037 (disabled check boxes) merged on the user's go without running its tests; tagged v0.32.0 locally.

### STILL OPEN AT CLOSE
- Next: Select or Slider (with `Inside(Part, …)`: new parts).
- Tests not run since session 32; main and tags v0.18.0–v0.32.0 not pushed.

---

## SESSION 75 — 2026-10-20 — kivuli.ui Slider: discovery, PRP-038 draft — closed

Branch: main

### WHAT WAS DONE
- Found: native sliders can't be disabled either. Interview: the slider, its parts and disabled sliders in one PRP (B).
- PRP-038 drafted.

### STILL OPEN AT CLOSE
- PRP-038 awaits the user's approval.

---

## SESSION 76 — 2026-10-20 — PRP-038: kivuli.ui Slider, disabled sliders, built, not merged — closed

Branch: prp-038-ui-slider (not merged: waiting for the user's look)

### WHAT WAS DONE
- `Part.Trough`, `Highlight`, `Knob`; `View.Slider`'s handler an `Option` through view, map, `.disabled`, build, diff, patch, testing; src/ui/sliders.la. A Slider section in components.la. GTK parsed the CSS. Docs; 0.33.0 (not tagged). Tests type-checked only.
- Noticed: the native slider draws its number beside the thumb; raised.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.33.0. Next: Select.

---

## SESSION 77 — 2026-10-20 — Merge PRP-038; Select discovery, PRP-039 draft — closed

Branch: main

### WHAT WAS DONE
- PRP-038 merged on the user's go (tests not run), tagged v0.33.0.
- Probed GTK's drop-down node tree (the pop-up is a CSS descendant; a `row` in the trigger too; the list's scrollbars have `trough`/`slider`). Read the reference select page (collapsed). Interview: trigger, list, items, placeholder, `Select.invalid()` (A); groups, disabled, align-with-trigger later.
- PRP-039 drafted.

### STILL OPEN AT CLOSE
- PRP-039 awaits the user's approval.

---

## SESSION 78 — 2026-10-20 — PRP-039: the name — closed

Branch: main

### WHAT WAS DONE
- The user asked why "dropdown" not "select": `select` is a Lugha keyword (no function can have the name), and the native widget is `dropdown`. The user chose `Dropdown` throughout (A): PRP-039's `Select.invalid()` is `Dropdown.invalid()`.

### STILL OPEN AT CLOSE
- PRP-039 awaits the user's approval.

---

## SESSION 79 — 2026-10-20 — PRP-039: kivuli.ui dropdown, built, not merged — closed

Branch: prp-039-ui-dropdown (not merged: waiting for the user's look)

### WHAT WAS DONE
- Parts `Trigger`, `Arrow`, `List`, `Item` as paths with their own class codes; states inside a part; the drop-down's placeholder (live `Choices`, styling syncs it); src/ui/dropdowns.la; a Dropdown section in components.la. GTK parsed the CSS. Docs; 0.34.0 (not tagged).
- Type-checked every test and example: three stale tests from earlier PRPs fixed. Saved as memory: type-check all tests when a shared type changes.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.34.0.
- Tests not run since session 32; main and tags v0.18.0–v0.33.0 not pushed.

---

## SESSION 80 — 2026-10-20 — Merge PRP-039 — closed

Branch: main (prp-039-ui-dropdown merged)

### WHAT WAS DONE
- PRP-039 (dropdown) merged on the user's go without running its tests; tagged v0.34.0 locally.

### STILL OPEN AT CLOSE
- All the planned first-set components are in. Next is the user's choice; the full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.34.0 not pushed.

---

## SESSION 81 — 2026-10-20 — Switch: discovery, PRP-040 draft — closed

Branch: main

### WHAT WAS DONE
- `switch` isn't a Lugha keyword; kivuli has no native switch and lugha-gtk no switch bindings. Interview: a native `switch` on GtkSwitch, then `ui.switch` (A).
- PRP-040 drafted.

### STILL OPEN AT CLOSE
- PRP-040 awaits the user's approval.

---

## SESSION 82 — 2026-10-20 — PRP-040: switch, built, not merged — closed

Branch: prp-040-switch (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.36 (switch bindings; test passes; pushed, tagged).
- The native switch: `View.Switch`, `Live.Switch`, build (notify::active), `diff_switch`, patch, map, `.disabled`, walk, `kivuli.testing.flip`, `test/live.la`'s `shown()` arm. src/ui/switches.la. A Switch section in components.la. Docs; 0.35.0 (not tagged). Every test and example type-checks; tests not run.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.35.0.

---

## SESSION 83 — 2026-10-20 — Merge PRP-040 — closed

Branch: main (prp-040-switch merged)

### WHAT WAS DONE
- PRP-040 (switch) merged on the user's go without running its tests; tagged v0.35.0 locally.

### STILL OPEN AT CLOSE
- Next is the user's choice (tabs, tooltip, progress, avatar, dialog; drop-down groups, disabled drop-downs, the slider's number). The full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.35.0 not pushed.

---

## SESSION 84 — 2026-10-20 — kivuli.ui Tabs: discovery, PRP-041 draft — closed

Branch: main

### WHAT WAS DONE
- Read the reference tabs page. Interview: tabs from the model, built of buttons (A). Drafted PRP-041 with two proposals: triggers as views (`tab`, `tab_with`) picked by position, and the line variant as `ui.tabs_line` (a row's style can't reach its buttons: DECISION-028).

### STILL OPEN AT CLOSE
- PRP-041 awaits the user's approval.

---

## SESSION 85 — 2026-10-20 — PRP-041: kivuli.ui Tabs, built, not merged — closed

Branch: prp-041-ui-tabs (not merged: waiting for the user's look)

### WHAT WAS DONE
- src/ui/tab_bars.la (`tab`, `tab_with`, `tabs`, `tabs_line`); a Tabs section in components.la (panels in a card, line, icons, a disabled trigger). GTK parsed the CSS. Docs; 0.36.0 (not tagged). Every test and example type-checks; tests not run.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.36.0.

---

## SESSION 86 — 2026-10-20 — Merge PRP-041 — closed

Branch: main (prp-041-ui-tabs merged)

### WHAT WAS DONE
- PRP-041 (Tabs) merged on the user's go without running its tests; tagged v0.36.0 locally.

### STILL OPEN AT CLOSE
- Next is the user's choice (tooltip, progress, avatar, dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number). The full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.36.0 not pushed.

---

## SESSION 87 — 2026-10-20 — Dropdown menu: discovery, PRP-042 draft — closed

Branch: main

### WHAT WAS DONE
- Read the reference page and its source (the parts' classes). Probed GTK's popover menu (modelbutton items, label.title, separator, accelerator, arrow.right, check.left; nested submenus possible). Interview: GTK's popover menu opened by a click (A); all but check and radio items (B).
- Found: a GMenu item can't carry a CSS class, so a destructive item can't be styled through it: put to the user in PRP-042.
- PRP-042 drafted.

### STILL OPEN AT CLOSE
- PRP-042 awaits the user's approval and the destructive-item choice.

---

## SESSION 88 — 2026-10-20 — PRP-042: dropdown menu, built, not merged — closed

Branch: prp-042-dropdown-menu (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.37 (menu items, submenus, nested popovers, gesture phase and state; pushed, tagged).
- Menu entries `Hinted`, `Label`, `Submenu`; items numbered depth first; the model in titled sections with nested submenus; nested popovers; `.dropdown_menu` (a style flag, a claimed primary click); five menu parts; src/ui/menu_looks.la (`Menu.look()`, `dropdown_menu`). A Dropdown menu section in components.la. Docs; 0.37.0 (not tagged). Every test and example type-checks; tests not run.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.37.0. Next: check and radio items (the PRP's follow-up).

---

## SESSION 89 — 2026-10-20 — Merge PRP-042 — closed

Branch: main (prp-042-dropdown-menu merged)

### WHAT WAS DONE
- PRP-042 (dropdown menus) merged on the user's go without running its tests; tagged v0.37.0 locally.

### STILL OPEN AT CLOSE
- Next: the user's choice (check and radio menu items; tooltip, progress, avatar, dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number). The full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.37.0 not pushed.

---

## SESSION 90 — 2026-10-20 — Check and radio menu items: discovery, PRP-043 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: `check_item(label, checked, on_toggle)` and `radio_group(selected, choices)`, from the model (A). GTK: stateful GActions (boolean, string with targets); lugha-gtk lacks GVariant bindings. Adwaita has `object-select-symbolic` and `media-record-symbolic` for the marks.
- PRP-043 drafted.

### STILL OPEN AT CLOSE
- PRP-043 awaits the user's approval.

---

## SESSION 91 — 2026-10-20 — PRP-043: check and radio menu items, built, not merged — closed

Branch: prp-043-check-radio-menu (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.38 (stateful actions, GVariant values, a menu item's target; pushed, tagged).
- `MenuEntry.Check`/`Radio`, `check_item`, `radio_group`; plain, check and radio actions numbered apart; `sync_checks`/`sync_radios` set states each refresh; the model's radio items with targets; `Part.MenuCheck`/`MenuRadio` and marks in `Menu.look()`. A View menu in components.la. Docs; 0.38.0 (not tagged). Every test and example type-checks; tests not run.

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.38.0.

---

## SESSION 92 — 2026-10-20 — Menu radio dot size — closed

Branch: prp-043-check-radio-menu (not merged)

### WHAT WAS DONE
- From the user's screenshot: the menu works (labels, hint, disabled item, the dot on the selected radio), but the dot was ~14 px: GTK draws a radio's mark at the node's size. Now an 8 px node in a 4 px margin. Test updated (type-checked); components rebuilt.

### STILL OPEN AT CLOSE
- The user to look again (and tick Status bar); then merge PRP-043, tag v0.38.0.

---

## SESSION 93 — 2026-10-20 — Merge PRP-043 — closed

Branch: main (prp-043-check-radio-menu merged)

### WHAT WAS DONE
- PRP-043 (check and radio menu items) merged on the user's go without running its tests; tagged v0.38.0 locally.

### STILL OPEN AT CLOSE
- Next: the user's choice (tooltip, progress, avatar, dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number). The full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.38.0 not pushed.

---

## SESSION 94 — 2026-10-20 — Tooltip look: discovery, PRP-044 draft — closed

Branch: main

### WHAT WAS DONE
- `.tooltip` exists; GTK puts tooltips in their own window off the widget's surface, so a node's style can't reach its tooltip. Interview: one look for every tooltip, set on the root through a sibling part (A). A C probe couldn't show a tooltip without a real pointer: the sibling placement is from the session-77 drop-down probe, to check by hand.
- PRP-044 drafted.

### STILL OPEN AT CLOSE
- PRP-044 awaits the user's approval.

---

## SESSION 95 — 2026-10-20 — PRP-044: tooltip look, built, not merged — closed

Branch: prp-044-tooltip-look (not merged: waiting for the user's look)

### WHAT WAS DONE
- `Part.Tooltip` (`~ tooltip`); src/ui/tooltip_looks.la (`Tooltip.look()`); components.la's root takes it, three buttons gain tooltips. GTK parsed the CSS. Docs; 0.39.0 (not tagged). Every test and example type-checks; tests not run.

### STILL OPEN AT CLOSE
- The user to hover the tooltips: whether the look reaches them is unverified. Then merge, tag v0.39.0.

---

## SESSION 96 — 2026-10-20 — Merge PRP-044 — closed

Branch: main (prp-044-tooltip-look merged)

### WHAT WAS DONE
- PRP-044 (tooltip look) merged on the user's go without running its tests; tagged v0.39.0 locally. The user didn't say whether the hover check showed the look: the sibling selector stays unconfirmed.

### STILL OPEN AT CLOSE
- Next: the user's choice (progress, avatar, dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number). The full test suite is overdue (none since session 32).
- main and tags v0.18.0–v0.39.0 not pushed.

---

## SESSION 97 — 2026-10-20 — Progress: discovery, PRP-045 draft — closed

Branch: main

### WHAT WAS DONE
- No progress bar in kivuli or lugha-gtk. Interview: a native `progress(value)` out of 100 on GtkProgressBar, then `ui.progress` (A). Found: the fill can't ease (GTK sizes it from the fraction, not CSS).
- PRP-045 drafted.

### STILL OPEN AT CLOSE
- PRP-045 awaits the user's approval.

---

## SESSION 98 — 2026-10-20 — The first full run: three stale tests fixed — closed

Branch: main

### WHAT WAS DONE
- The user ran `lughac test test/`: 421 tests, 418 passed, 3 failed, all stale expectations, no code bugs: palettes (colours carry alpha `#…ff`; a hover background's extra `bgi-none` rule and its `kv-hover-` class), live_type (a press rule's class repeats four times since FocusWithin). Fixed; palettes and live_type pass (17/17).

### STILL OPEN AT CLOSE
- The examples' own tests (`lughac test examples/tasks.la examples/browser.la examples/controls.la`) and every example's build, before a push.
- PRP-045 (progress) awaits the user's approval.
- main and tags v0.18.0–v0.39.0 not pushed.

---

## SESSION 99 — 2026-10-20 — PRP-045: progress, built, not merged — closed

Branch: prp-045-progress (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.39 (progress bars; test passes; pushed, tagged).
- `View.Progress`/`Live.Progress`, build (fraction clamped), `SetProgress`, map, `shown()` arm; `Part.Fill`; src/ui/progress_bars.la. A Progress section in components.la. Its tests run and pass (22). Docs; 0.40.0 (not tagged).

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.40.0. Before a push: the examples' tests and every example's build.

---

## SESSION 100 — 2026-10-20 — Merge PRP-045 — closed

Branch: main (prp-045-progress merged)

### WHAT WAS DONE
- PRP-045 (progress) merged on the user's go, its own tests run and passing; tagged v0.40.0 locally.

### STILL OPEN AT CLOSE
- Next: the user's choice (avatar, dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number).
- Before a push: the examples' own tests and every example's build (the test suite passed in session 98). main and tags v0.18.0–v0.40.0 not pushed.

---

## SESSION 101 — 2026-10-20 — Avatar: discovery, PRP-046 draft — closed

Branch: main

### WHAT WAS DONE
- Interview: `ui.avatar(image, initials)` as a picture layered over its fallback (A): a picture that can't load shows nothing, so the initials show through. PRP-046 drafted (`Avatar.size`, `avatar_group`).

### STILL OPEN AT CLOSE
- PRP-046 awaits the user's approval.

---

## SESSION 102 — 2026-10-20 — PRP-046: kivuli.ui Avatar, built, not merged — closed

Branch: prp-046-ui-avatar (not merged: waiting for the user's look)

### WHAT WAS DONE
- src/ui/avatars.la (`avatar`, `Avatar.size`, `avatar_group`); an Avatar section in components.la (a picture, initials, a missing file, a group, a large one). Its test passes; GTK parsed the CSS. Docs; 0.41.0 (not tagged).

### STILL OPEN AT CLOSE
- The user to look; then merge, tag v0.41.0.

---

## SESSION 103 — 2026-10-20 — Merge PRP-046 — closed

Branch: main (prp-046-ui-avatar merged)

### WHAT WAS DONE
- PRP-046 (Avatar) merged on the user's go; tagged v0.41.0 locally.

### STILL OPEN AT CLOSE
- Next: the user's choice (dialog; vertical tabs; drop-down groups; disabled drop-downs; the slider's number).
- Before a push: the examples' own tests and every example's build. main and tags v0.18.0–v0.41.0 not pushed.

---

## SESSION 104 — 2026-10-20 — Dialog: discovery, PRP-047 draft — closed

Branch: main

### WHAT WAS DONE
- One window only (anti-pattern 9): a dialog is an overlay over the page. Interview: a root `.dialog(Option<Dialog>)` that kivuli makes modal (A). Design: the overlay always there (open/close adds layers, never rebuilds the page); the page `.inert` while open; a backdrop button `.focusable(false)`; a ✕ with `.shortcut("Escape")`.
- PRP-047 drafted.

### STILL OPEN AT CLOSE
- PRP-047 awaits the user's approval.

---

## SESSION 105 — 2026-10-20 — PRP-047: dialog, built, not merged — closed

Branch: prp-047-dialog (not merged: waiting for the user's look)

### WHAT WAS DONE
- lugha-gtk 0.1.40 (can-target, can-focus; pushed, tagged).
- Style `inert`, `focusable`, `dialog_root`; `.inert`, `.focusable`, `.dialog` (body and `Dialog` in mapping.la); styling sets them; paint refuses `.dialog` below the root; src/ui/dialogs.la. A Dialog section in components.la. Tests run and pass (dialogs, live_dialog; 16 GTK tests re-run). Docs; 0.42.0 (not tagged).

### STILL OPEN AT CLOSE
- The user to look (open, Escape, backdrop, ✕, Tab stays in the dialog); then merge, tag v0.42.0.

---

## SESSION 106 — 2026-10-20 — Merge PRP-047 — closed

Branch: main (prp-047-dialog merged)

### WHAT WAS DONE
- PRP-047 (Dialog) merged on the user's go; tagged v0.42.0 locally.

### STILL OPEN AT CLOSE
- Next: the user's choice (vertical tabs; drop-down groups; disabled drop-downs; the slider's number; sheets or drawers from the dialog's pieces).
- Before a push: the examples' own tests and every example's build. main and tags v0.18.0–v0.42.0 not pushed.

---

## SESSION 107 — 2026-10-20 — Follow-ups: discovery, PRP-048 to PRP-052 drafts — closed

Branch: main

### WHAT WAS DONE
- The follow-ups in one go, one PRP each (the scope rule), all A: PRP-048 vertical tabs (`ui.tabs_vertical`, `ui.tabs_line_vertical`), PRP-049 drop-down groups (`dropdown_grouped`; a flatten list model's sections and a header factory, probed first; build.la's drop-down arm moves to controls.la, the split needing approval), PRP-050 disabled drop-downs (`.disabled`; choices hold `Option<Msg>`), PRP-051 `.show_value(on)` (`ui.slider` hides the number; views.la reaches ~500 lines), PRP-052 `ui.sheet(close, Edge, parts)` (a `Dialog`; the card places itself).

### STILL OPEN AT CLOSE
- The five drafts await approval; then "build all" builds them in order, merging each.

---

## SESSION 108 — 2026-10-20 — PRP-048: vertical tabs, built, paused on a file split — closed

Branch: prp-048-vertical-tabs (not merged)

### WHAT WAS DONE
- PRP-048 to PRP-052 approved ("build all"). PRP-048 built: `ui.tabs_vertical`, `ui.tabs_line_vertical`; a native button's `Text.start()`/`end()` places its content. Its tests pass; components built.

### STILL OPEN AT CLOSE
- "build all" paused: `examples/components.la` is 602 lines (over 500 since PRP-041) and `test/live_forms.la` 562. A split proposed (examples/gallery/: state, actions, choices, display); waiting for approval. Then merge PRP-048 and go on with 049–052.

---

## SESSION 109 — 2026-10-20 — File split (components, live_forms); PRP-048 merged — closed

Branch: prp-048-vertical-tabs → main

### WHAT WAS DONE
- The split (A): `examples/components.la` → itself (57 lines) and `examples/gallery/` (state, actions, choices, display; `import gallery.x;` from the main file's folder, probed first); `test/live_forms.la` → itself and `test/live_shortcuts.la` (6 tests pass). No file over 500 lines.
- PRP-048 merged; v0.43.0 tagged locally. GTK parsed the vertical tabs' rules.

### STILL OPEN AT CLOSE
- "build all" goes on: PRP-049 (drop-down groups) next.

---

## SESSION 110 — 2026-10-20 — PRP-049: drop-down groups, merged — closed

Branch: prp-049-dropdown-groups → main

### WHAT WAS DONE
- lugha-gtk 0.1.41 (groups: `src/c/groups.c`; pushed, tagged). `dropdown_grouped`, `ui.dropdown_grouped`, `Part.GroupHeading`/`GroupLine`; `View.Dropdown` and `SetChoices` carry groups; the drop-down's creation moved to controls.la. Tests pass; every test and example type-checks; GTK parsed the rules. Merged; v0.44.0 tagged locally.

### STILL OPEN AT CLOSE
- "build all" goes on: PRP-050 (disabled drop-downs) next.

---

## SESSION 111 — 2026-10-20 — PRP-050: disabled drop-downs, merged — closed

Branch: prp-050-disabled-dropdowns → main

### WHAT WAS DONE
- Drop-down choices carry `Option<Msg>`; `.disabled` clears them; insensitive while none sends; `choose` gives `Silent`; `ui.dropdown` fades. Tests pass; every test and example type-checks. Merged; v0.45.0 tagged locally.

### STILL OPEN AT CLOSE
- "build all" goes on: PRP-051 (`.show_value`) next.

---

## SESSION 112 — 2026-10-20 — PRP-051: a slider's number, merged — closed

Branch: prp-051-slider-value → main

### WHAT WAS DONE
- lugha-gtk 0.1.42 (`gtk_scale_get_draw_value`; pushed, tagged). Style `show_value`, `.show_value(on)`, styling sets it; `ui.slider` hides the number. Tests pass; every test and example type-checks. Merged; v0.46.0 tagged locally. views.la at 497 lines.

### STILL OPEN AT CLOSE
- "build all" goes on: PRP-052 (sheet) next.

---

## SESSION 113 — 2026-10-20 — PRP-052: sheet, merged; build all done — closed

Branch: prp-052-sheet → main

### WHAT WAS DONE
- `ui.sheet(close, edge, parts)`: a `Dialog` against an edge; a dialog's card places itself. A Sheet section in the gallery. Tests pass; every test and example type-checks; GTK parsed the rules. Merged; v0.47.0 tagged locally.
- "build all" (PRP-048 to PRP-052) done.

### STILL OPEN AT CLOSE
- By hand: vertical tabs, grouped and disabled drop-downs, the slider without its number, sheets from each edge (`lughac run examples/components.la`).
- Before a push: the full suite, the examples' own tests, every example built. main and tags v0.18.0–v0.47.0 not pushed.
- views.la at 497 lines: the next modifier needs a split (DECISION-018 keeps modifiers there).

---

## SESSION 114 — 2026-10-20 — Full check after build all; a lughac report — closed

Branch: main

### WHAT WAS DONE
- Full suite: 462 tests, 462 passed (exit 0). The examples' own tests, each alone: tasks 8, browser 5, controls 3, all pass. Every example built.
- `lughac test examples/tasks.la examples/browser.la examples/controls.la` (as CLAUDE.md writes it) fails: E0304 `main` belongs in the main file, "these test files check alone but not together; please report it". Recorded as DECISION-029 (a lughac report); run them one at a time until it's fixed.

### STILL OPEN AT CLOSE
- By hand: the five new pieces in `examples/components.la`. Push main and tags v0.18.0–v0.47.0 only on the user's go.

---

## SESSION 115 — 2026-10-20 — Published: main and tags v0.18.0–v0.47.0 — closed

Branch: main

### WHAT WAS DONE
- On the user's go ("push it"), after session 114's full run (462 pass): pushed main and every tag; GitHub now has v0.1.0–v0.47.0. Merged local branches deleted (only main left).

### STILL OPEN AT CLOSE
- By hand: the five new pieces in `examples/components.la`. Next component: the user's choice.

---

## SESSION 116 — 2026-10-20 — SVG pictures: discovery, PRP-053 draft — closed

Branch: main

### WHAT WAS DONE
- Interview (all A): SVG through `Image.File`/`Image.Bytes`, told apart by its bytes; GTK 4.22's `GtkSvg` with external resources, animations and extensions off (system fonts on); not in backgrounds (CSS `data:` URLs can't set the features). PRP-053 drafted.

### STILL OPEN AT CLOSE
- PRP-053 awaits the user's approval.

---

## SESSION 117 — 2026-10-20 — PRP-053: SVG images, stopped on a GTK crash — closed

Branch: prp-053-svg-images (nothing built)

### WHAT WAS DONE
- Probed GtkSvg first: sizes fine (width/height, viewBox-only, prolog); broken input reports an error and a 200 × 200 default size; a 100000 px claim is kept as is. **Crash**: any non-`data:` `<image>` with external resources off segfaults in `gtk_svg_load_from_bytes` (GTK 4.22.4). Recorded as DECISION-030; stopped.

### STILL OPEN AT CLOSE
- DECISION-030: the user's choice (A: refuse SVGs holding `<image>`; B: external resources on; C: wait for GTK).

---

## SESSION 118 — 2026-10-20 — PRP-053: SVG images, merged — closed

Branch: prp-053-svg-images → main

### WHAT WAS DONE
- DECISION-030 resolved (A). lugha-gtk 0.1.43 (GtkSvg, `lgtk_svg_load`; pushed, tagged). `kind_of`, SVG refused when it holds `<image>`; `textures` draws an SVG as GtkSvg; no SVG backgrounds. Tests pass; every test and example type-checks; controls built. Merged; v0.48.0 tagged locally (not pushed).

### STILL OPEN AT CLOSE
- By hand: `lughac run examples/controls.la` (the SVG at 32 and 96 px). Push v0.48.0 on the user's go. Reporting the GTK crash upstream: only on the user's go.

---

## SESSION 119 — 2026-10-21 — Pushed v0.48.0 — closed

Branch: main

### WHAT WAS DONE
- Full suite 467/467 (exit 0); tasks 8, browser 5, controls 3 (each alone, DECISION-029); every example built; fmt clean. On the user's go: pushed main and v0.48.0.

### STILL OPEN AT CLOSE
- By hand: the SVG in `examples/controls.la`. Reporting the GtkSvg crash (DECISION-030) to GTK: only on the user's go. Next component: the user's choice.

---

## SESSION 120 — 2026-10-21 — SVG confirmed by hand — closed

Branch: main

### WHAT WAS DONE
- The user ran `examples/controls.la`: the SVG is sharp at 32 and 96 px. Noted in PRP-053.

### STILL OPEN AT CLOSE
- Reporting the GtkSvg crash (DECISION-030) to GTK: only on the user's go. Next component: the user's choice.

---

## SESSION 121 — 2026-10-21 — rapidkoi's window controls look wrong: investigation — open

Branch: main

---

## NEXT SESSION START POINT

1. Open a new session in this file (step 1, before anything else) and commit it.
2. Read CLAUDE.md, MEMORY.md, DECISIONS.md, then this file's last session; go by `git log` for what's merged.
3. kivuli is at 0.48.0, pushed with tags up to v0.48.0 (session 119). Later tags wait for the user's go, after a full test run (none since session 32: the user is speeding tests up). lugha-gtk is at 0.1.43, pushed (standing go).
4. `src/` is in folders (view/, styles/, core/, render/; DECISION-026); `test/` stays flat (DECISION-027, open: a lughac change).
5. The plan (session 34): shadcn-like components. Done: palettes and tokens (PRP-026), motion (PRP-027), `kivuli.ui` with Badge (PRP-028: variants are `Look`s through `.style`; component files import the entry's modules, not `lib`) and Button (PRP-029: `ui.button` beside the native one; variants restate colours in states to outrank GTK's button rules; `Token.alpha`). Then text alignment (PRP-030: start by default, `Text.center()`) and Card (PRP-031: parts as functions; no font size on containers, since scales compound; DECISION-028 open) and Input (PRP-032: `State.FocusWithin`, as a GTK field's text child takes the focus). Field (PRP-033: parts, `field_horizontal`). the input group (PRP-034: the frame draws border and ring; `group_input` has none). Separator (PRP-035), Checkbox with widget parts (PRP-036: `Inside(Part, …)`, `Mark.icon`), disabled check boxes (PRP-037), Slider and disabled sliders (PRP-038), dropdown (PRP-039: parts as paths, states inside parts, a placeholder). Then a native switch and `ui.switch` (PRP-040), Tabs from the model (PRP-041), dropdown menus (PRP-042: `.dropdown_menu`, labels, submenus, hints, `Menu.look()`). Then check and radio menu items (PRP-043: stateful actions set from the view), the tooltip look (PRP-044: `Part.Tooltip` = `~ tooltip` on the root; unconfirmed by hand), progress (PRP-045), avatar (PRP-046: a picture layered over its initials), dialog (PRP-047: `.dialog` on the root, an always-there overlay, the page `.inert`, Escape on the ✕). Then vertical tabs (PRP-048), drop-down groups (PRP-049: a flatten model and a header factory, lugha-gtk's groups.c), disabled drop-downs (PRP-050), `.show_value` (PRP-051), sheets (PRP-052). `examples/components.la`'s sections are in `examples/gallery/`. Next is the user's choice (later: tooltip, dialog, progress, avatar; groups in drop-downs; disabled drop-downs) (GTK entry icons dropped: the input group covers them) (button variants, input, card, badge, separator, select, checkbox, slider), then widgets kivuli lacks (switch, tabs, tooltip, dialog, progress, avatar).
6. Gotchas: a named import binds the module's last name (E0311/E0314: no `styles.effects`); `lughac fmt` reflows text scripted edits expect: re-read first; Lugha strings interpolate `{…}`: write `{{`; an app's GtkCssProvider needs `lgtk_css_provider_new_for_scheme` for dark rules; files may not exceed 500 lines; gate any push on the test run's real exit status (the full suite last passed in session 98, 421 tests); never chain a bare `git push` in kivuli; no Co-Authored-By trailer on commits; no shadcn by name in code comments; check only affected examples and build only the one being run; commit `lughac fmt` output before switching branches; chain git steps with `&&`, never `;`.
