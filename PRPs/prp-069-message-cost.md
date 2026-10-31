## FEATURE: each message checks the painted view in one pass, without arrays, and an unchanged lazy part adds nothing to it

## OBJECTIVE
An app writes nothing new. Today every message runs three walks that build arrays of `(View, Style)` over everything outside an unchanged lazy part (`duplicate`, `stylesheet`, `background_pictures`), plus one per `button_with`. Each array element copies a whole `Style` (two `Sheet`s). The arrays are large, so they trigger collections that mark the whole heap, and an app with a big lazy part (Rapidkoi showing Notes) has a big heap. After this PRP those checks are one pass of plain recursion over the painted view. That pass computes CSS text only for classes the stylesheet hasn't seen, and an unchanged lazy part adds only its shortcuts.

## CONTEXT

**Where the request comes from:** Rapidkoi's DECISION-067, § 17 of `../rapidkoi/docs/specs/kivuli-requests.md`. On kivuli 0.64, Rapidkoi running Notes still takes 70–180 ms after its own view per message, and opening a popover sends five messages. A `perf` profile put about 40% of the time in the collector (`GC_mark_from`, reached through `lugha_rt_array_grow` → `GC_alloc_large`) and about 36% inclusive in `walk.with_children`. `sheets.rules`, `hexed`, `motion_css` and `add_rules`'s set inserts followed. The second part, `Sub.window_active` firing for the window's own popovers, needs lugha-gtk first: it's DECISION-047, not this PRP.

### Investigation: measured headlessly (scratch benchmarks, never committed)

**1. The frame outside the lazy part costs, not the lazy part.** View = a frame of styled nodes, a popover, and a lazy screen whose key holds. Times are per message:

| frame nodes | lazy | view | resolve | paint | shortcuts | add_rules (stylesheet, pictures, inserts) | diff | total |
|---:|---|---:|---:|---:|---:|---:|---:|---:|
| 300 | none | 3.2 | 4.7 | 7.9 | 13.2 | 30.1 | 6.1 | 65 ms |
| 300 | 3,000 nodes, 10 classes | 5.3 | 2.3 | 5.8 | 9.4 | 34.9 | 6.4 | 64 ms |
| 300 | 3,000 nodes, 1,000 classes | 7.8 | 2.6 | 6.3 | 7.3 | 38.1 | 6.9 | 69 ms |
| 30 | none | 0.2 | 0.2 | 0.6 | 0.6 | 1.4 | 0.9 | 4 ms |
| 30 | 3,000 nodes | 0.2 | 0.3 | 0.6 | 1.1 | 2.2 | 1.0 | 5.5 ms |

**2. The walks are the cost.** Over 900 frame nodes, one pass:
- `nodes_painted` (arrays of `(View, Style)`): **19.4 ms**.
- The same traversal as plain recursion: **0.5 ms**.
- `rules(sheet)` for each of the 600 styled nodes: **3.5 ms**.

Lugha structs are values held inline (`lughac spec`, "Value semantics"), and `Style` holds two `Sheet`s of a dozen nested structs each. So every element pushed copies a large struct. `with_children` copies each child's array into its parent's, and `plain()` builds a fresh `Style` per child.

**3. The heap multiplies it.** With 20 retained 3,000-node trees on the heap (Rapidkoi holds Notes' painted tree, the app's own view description and its state), the frame-30 case's checks go from about 2 ms to 11 ms per message. The allocations are the same, but each collection marks more. This is how a big lazy part slows a message it isn't part of, and why Rapidkoi's new tab, with the same frame and a small heap, is quick.

**4. The one-pass prototype.** `duplicate`, `stylesheet`, `background_pictures` and the known-set inserts against a single recursion (a `mut self` method), which reads each node's painted classes and calls `rules` only for a node with a class not seen before:

| case | today's checks | prototype |
|---|---:|---:|
| frame 150, tiny lazy, small heap | 21.3 ms | 0.3 ms |
| frame 150, lazy 3,000, small heap | 13.7 ms | 0.2 ms |
| frame 150, lazy 3,000, heap +20 trees | 15.6 ms | 0.2 ms |
| frame 30, lazy 3,000, heap +20 trees | 11.3 ms | 0.0 ms |

### The hypotheses, checked
- **The walks grow arrays, and those allocations bring full collections**: *confirmed, and the main cost.* They cost about 20 µs a node per walk against 0.5 µs for the traversal itself. There are three walks per message, plus `check_content` per `button_with`, all over the nodes outside unchanged lazies.
- **`stylesheet` re-adds every rule each painted lazy found, every message**: *real, but minor.* With 1,000 distinct classes in the lazy, about 1–3 ms of the total. Fixed anyway: a lazy part's rules reach the stylesheet once.
- **`add_rules` re-inserts known classes, and `Ref.get` + `insert` clones the set**: *ruled out.* A known set of 10,000 classes re-inserted in full costs 1.4 ms a message, the same as `has` alone, so no clone happens per message. With 2,000 classes it's 0.2 ms.
- **`sheets.rules` regenerates every frame node's CSS each message**: *confirmed*, twice (in `paint` and in `stylesheet`), at about 6 µs a node. The `stylesheet` call goes: CSS text is built only for classes not seen before. The call in `paint` stays (see Must NOT Do).

### Files
- Read first: `src/render/live.la` (`dispatch`, `add_rules`, `check_shortcuts`, `Styles`), `src/styles/paint.la`, `src/view/walk.la`, `src/view/keys.la`, `src/view/findings.la`, `src/view/lazies.la`, `test/lazy_found.la`, PRP-068.
- New: `src/styles/gathering.la` (the pass, about 150 lines), `test/gathering.la`.
- Changes:
  - `src/view/findings.la`: `Found.fresh`.
  - `src/view/lazies.la`: reuse clears `fresh`.
  - `src/styles/paint.la`: `found_in`, `stylesheet` and `background_pictures` through `gathering`.
  - `src/view/walk.la`: `nodes_painted` goes, and `interactive` becomes plain recursion.
  - `src/view/keys.la`: `shortcuts` and `duplicate` move out; a `clash(shortcuts)` over a list stays beside `parse`.
  - `src/render/live.la`: `Styles.seen`, and `dispatch` gathers once.
  - Tests: `test/keys.la` and `test/lazy_found.la` (imports, `fresh`).
- No new externs, dependencies or GTK calls. Depends on DECISION-046 (resolved). Nothing open.

## REQUIREMENTS

### Must Do
- Each message checks the painted view in **one** pass of plain recursion: no `(View, Style)` arrays and no `Style` copies. The pass collects:
  - every enabled button's shortcut;
  - the CSS rules the stylesheet doesn't have yet;
  - the background pictures it doesn't have yet.
- CSS text (`rules`, `rule_text`) is built only for a node with a class the stylesheet hasn't seen. A class's rule never changes (`sheets.la`), so a seen class needs nothing.
- A lazy part reused with its key unchanged adds only its found shortcuts, for the clash check. Its rules and pictures were added when it was painted.
- Everything PRP-068 keeps correct stays correct:
  - every class on screen has its rule, after a rebuild and nested;
  - pictures inside lazies get their rules;
  - a shortcut clash across a lazy's edge is caught;
  - `paint`'s refusals hold on build and rebuild;
  - a `button_with`'s content check holds.
- `stylesheet(v)` and `background_pictures(v)` stay, for tests and for `found_in`, giving everything a painted view needs.

### Must NOT Do
- Cache a node's classes across messages (`paint` still calls `rules` once per frame node: about 6 µs each, ~1 ms for 150 nodes). Matching nodes against the last view's would need a second `resolve`-like pass for a millisecond. Left out, and noted.
- Speed up `nodes()`, which only the testing helpers use.
- Change any public API. `Found` isn't exported from the entry.
- Touch `Sub.window_active` (DECISION-047, waiting on lugha-gtk).

## DESIGN

**`Found.fresh: bool`** (`findings.la`): true when `paint` has just computed it, false once `resolve` reuses it (`Found { fresh: false, ..f }`, a copy of four array handles). The rule: a found lazy that isn't fresh was in the last painted view, whose rules and pictures `add_rules` added, and the stylesheet never drops a rule. So skipping its rules and pictures is exact.

**`gathering.la`**:

```la
struct Gathering {
    known: Set<string>,     // rule keys and picture classes the stylesheet has
    seen: Set<string>,      // node classes whose rules it has
    every: bool,            // true: a lazy's rules count even when not fresh (stylesheet(v) for tests)
    rules: (string, string)[],
    pictures: (string, Image, string, string)[],
    shortcuts: (string, string)[],
}
impl Gathering { fun visit<Msg>(mut self, v: View<Msg>) … }
```

- A `Styled` node:
  - If any of its `classes` or `bar_classes` isn't `seen`, add its rules: the palette's, `rules(sheet)`, `rules(bar)`, and its picture, each only when not `known`. Then mark the classes seen.
  - If it's an enabled button with a shortcut, add the shortcut.
  - Then visit its children, `Menu` looked through as the walk does.
- A lazy with `Found`: its shortcuts always; its rules and pictures (those not known) when `fresh` or `every`. A lazy without one: its build, visited.
- `gathered(v, known, seen)` returns the `Gathering`.
- `stylesheet(v)` = `gathered(v, #{}, #{})` with `every` and its `rules`.
- `background_pictures(v)` likewise.
- `duplicate(v)` = `clash(gathered(…).shortcuts)`.
- `found_in(p)` takes `rules`, `pictures` and `shortcuts` from one pass, with `interactive(p)` beside it.

**`live.dispatch`**: one `gathered(tree, styles.classes, styles.seen)`. `check_shortcuts` reads its shortcuts. `add_rules` appends its new rules and pictures (building each picture's CSS from its bytes, as now) and stores `known` and `seen` when either grew. The stylesheet is reloaded only when something was added, as now.

**`walk.interactive`**: plain recursion. It returns the first interactive node, and reads a found lazy's `interactive`.

**Cost of a message after this:**
- the app's `view`;
- `resolve`, `paint` and `diff` over the nodes outside unchanged lazies;
- one recursion over those same nodes with a set lookup per class;
- per unchanged lazy, its shortcuts (typically a handful).

Nothing grows with what an unchanged lazy holds. The work is also much smaller, so far fewer collections are triggered.

**Errors:** none new. The shortcut panic keeps its text.

## TESTS
- [ ] Pure, `test/gathering.la`, each failing first:
  - a reused (not fresh) found lazy's rules and pictures aren't gathered, but its shortcuts are. This uses a hand-made `Found` with a rule the stylesheet lacks: gathered, it would show.
  - a fresh one's rules are gathered once and not again on the next message;
  - a node whose classes are all seen gets no rules gathered (a hand-made style whose classes are seen, but whose sheet would give a new key);
  - a new class on a frame node is gathered with its rule text;
  - the palette's and the header bar's rules on first sight;
  - a picture's on first sight;
  - a shortcut clash across a lazy's edge, through the gathered list.
- [ ] `test/lazy_found.la` and `test/keys.la`: updated (`fresh`; `duplicate`'s new module). They keep passing as they are otherwise.
- [ ] A regression guard: a 150-node frame with and without a 3,000-node unchanged lazy part, each message run as `live` runs it. With the lazy part, a message costs no more than a millisecond above the cost without it, and gathering stays under 1 ms (the checks it replaces took about 14 ms). A first bound of 5 ms for the whole message was wrong: the frame's own `view`, `resolve`, `paint` and `diff` take about 11 ms at 150 nodes, which this PRP leaves alone.
- [ ] The full suite, which covers `add_rules` on screen through `test/live_style.la`, `live_pictures.la`, `live_palette.la` and `live_header.la`.

## ACCEPTANCE
- [ ] Before-and-after numbers for each step, from the benchmark above, in the CHANGELOG entry and the note for Rapidkoi.
- [ ] `lughac test test/` (in batches), the examples' tests, `lughac fmt --check src examples test`, and `lughac check` on every example pass.
- [ ] CHANGELOG.md: 0.64.1 (no public API change).
- [ ] For the user, by eye: Rapidkoi on Notes, opening ⚙'s and the shield's cards.
- [ ] A note for Rapidkoi (§ 17, DECISION-067): the cost part, and that `window_active` waits on DECISION-047.

## RESULT (2026-10-31)

The same benchmark on 0.64.0 (in a worktree) and on this branch: a 150-node frame and a 3,000-node unchanged lazy part, ms per message, mean of 20.

| | view | resolve | paint | shortcuts, rules, pictures | diff | total |
|---|---:|---:|---:|---:|---:|---:|
| 0.64.0 | 1.0 | 5.8 | 2.1 | 14.4 | 2.8 | ~26 |
| 0.64.1 | 1.5 | 2.1 | 4.3 | 0.1 (gathering) | 2.6 | ~11 |

Without the lazy part, 0.64.1 measures the same (1.8, 2.1, 4.7, 0.1, 2.8). With no frame, a message beside the 3,000-node lazy part costs 0.1 ms. `resolve` and `paint` vary by a few ms between runs with when collections land. What's left grows with the nodes outside unchanged lazies (about 70 µs a node across view, resolve, paint and diff, mostly copies of `Style`), not with what a lazy part holds.

