## FEATURE: a message that leaves a lazy's key unchanged costs nothing for that lazy's size

## OBJECTIVE
An app writes nothing new. Today a lazy part whose key is unchanged is reused and left undiffed, but every message still walks the whole of it five more times. After this PRP, a message that leaves a lazy part's key unchanged does no work proportional to that part's size: opening a popover over a 2,000-node lazy screen costs about what it costs over an empty one.

```la
// Rapidkoi's content area, unchanged (../rapidkoi/src/ui/content.la, app_view):
lazy(screen_key(id, state, m.theme, own, first, last, a.pictures),
     fun() = screen_as(fill(pictured(d.root, a), first, last), m.theme, own))
// A popover elsewhere in the window opens: the screen is neither rebuilt, diffed nor walked.
```

## CONTEXT

**Where the request comes from:** Rapidkoi's DECISION-067 (§ 17 of `../rapidkoi/docs/specs/kivuli-requests.md`, which Rapidkoi is adding). Opening a popover in Rapidkoi's address box is slow when the guest app's screen is big and instant when it's small. Rapidkoi wraps each screen in `lazy`. Its own update, commands and view take under 10 ms per popover click and no longer grow with the screen.

### Investigation: measured headlessly

A scratch test (`test/zz_bench_scratch.la`, to be deleted, never committed) times each step of `dispatch` (`src/render/live.la:193`) separately. The view is a `column` holding a popover (6 nodes) and a `lazy` screen of rows. Each row has a bold or normal text and a button with a varied letter spacing. Only the popover's `open` changes, so the lazy's key holds every time. Mean of 10 messages, `std.time.monotonic_ms`:

| lazy nodes | view | resolve | painted | stylesheet | background_pictures | duplicate | diff | **total** |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 58 | 0 | 0 | 0 | 5 | 2 | 2 | 0 | **~9 ms** |
| 508 | 0 | 0 | 11 | 42 | 11 | 25 | 0 | **~89 ms** |
| 2,008 | 0 | 0 | 32 | 169 | 74 | 68 | 0 | **~343 ms** |

One `nodes()` walk alone takes 1, 15 and 79 ms. There were 8 distinct rules, 3 patches and no pictures at every size.

**Confirmed (lead 1).** `view`, `resolve` and `diff` already skip the unchanged lazy (0 ms at every size). Everything else is linear, or worse, in the lazy's size, though none of it changes:
- `painted` (`src/styles/paint.la:79`) re-paints the reused `built`, which is already painted. It recomputes `rules(sheet)` and the checks for every node.
- `stylesheet` (`paint.la:166`) walks `nodes()` twice and rebuilds every node's rule text (`rule_text`), only for `add_rules` to throw away the rules it already has. At about 4× the cost of a walk, it's the heaviest step.
- `background_pictures` (`paint.la:201`) is one more full walk.
- `duplicate` (`src/view/keys.la:104`), through `check_shortcuts`, is one more `nodes()` walk. With `check = true`, it re-runs `check_content` on every `button_with` inside, and that walks each one's content again.
- `nodes`/`walk` (`src/view/walk.la:18`) builds a fresh array of `(View, Style)` per subtree and copies it into its parent's. That's why a walk costs ~40 µs a node and grows a little faster than the node count.

At Rapidkoi's ~600-node Notes screen, that's roughly 100 ms of kivuli's work per message, before GTK does anything. That's enough to feel. A click that sends more than one message pays it each time.

**Ruled out for the steady state (lead 2).** Classes are atomic: one per property and value, and a class's rule never changes (`src/styles/sheets.la:1`). `add_rules` keeps the classes it has seen for the app's life (`styles.classes`), and reloads the CSS only when a class is new. The rule count stayed at 8 across all 10 messages. So reopening a popover never reloads the stylesheet. Its first opening may, once, if its card uses a value nothing else on screen has. A reload does happen whenever a lazy's key changes and its new build uses a new value. That isn't this symptom, but its GTK cost is measured below.

**Ruled out (lead 3, the kivuli side).** The popover's patches are 3 per message (`SetOpen` and the buttons' `SetMsg`). `apply` touches only the popover's widgets. `show_window` and `show_scheme` read the root's style only (`src/render/windows.la:36`, `src/render/schemes.la:15`). `resubscribe` works from the model, not the tree.

### Measured on GTK, with the user (2026-10-31)

`test/zz_live_bench_scratch.la` (scratch, never committed) was run by the user on GTK 4 under Wayland. It opens three windows in turn, each a popover over a lazy screen of 50, 600 or 2,000 nodes. In each it sends 10 messages 300 ms apart: open and close three times, then twice an open whose card uses a never-seen letter spacing (a stylesheet reload). For each message it times:
- **dispatch**: kivuli plus GTK's synchronous part;
- **after**: from the end of `dispatch` to the next `G_PRIORITY_LOW` tick, roughly GTK's restyle, layout and drawing;
- **kivuli pure**: the same pure steps `dispatch` runs (resolve, painted, stylesheet, background_pictures, duplicate, diff) over a mirror of the app's model and tree, just before `dispatch`, in the same process.

| lazy nodes | dispatch | kivuli pure (of which rules / duplicate) | after | after, with a reload |
|---:|---:|---:|---:|---:|
| 50 | 25–37 ms | 19–56 ms (13–38 / 4–19) | 1–5 ms | 11–13 ms |
| 600 | 132–160 ms | 142–239 ms (89–172 / 15–52) | 1–8 ms | 17–21 ms |
| 2,000 | 327–494 ms | 357–526 ms (160–351 / 48–174) | 3–11 ms | 57–58 ms |

What this shows:
- **The time in `dispatch` is kivuli's.** Its pure steps alone take as long as the whole `dispatch`, or longer, so GTK's synchronous share (popup, popdown, the 3 patches) is within the noise.
- **The walks run 2–4× slower in a GTK process than headless**, and they spike (a 115 ms `diff`, a 176 ms `paint`) where the headless run had none. That fits garbage-collection pauses under a larger heap: the walks allocate a fresh array per subtree. Removing the walks removes both the work and the allocation that triggers the pauses.
- **GTK's own work after a message is small and nearly flat** (1–11 ms).
- **A stylesheet reload does grow with the window**: about 12 ms at 50 nodes, about 20 ms at 600, about 58 ms at 2,000, as GTK restyles every widget. It happens only when a message brings a value nothing on screen has used yet, which is never on a reopened popover. It's left alone here and noted as a possible later improvement (more than one provider, say), not part of this PRP.

### Files

- Read first: `src/render/live.la` (dispatch, add_rules, check_shortcuts), `src/view/lazies.la`, `src/styles/paint.la`, `src/view/walk.la`, `src/view/keys.la`, `src/core/diff.la:97`, `src/view/mapping.la:95`, `src/render/build.la:273`, `test/lazy.la`, `test/live_lazy.la`, DECISION-002.
- Changes: `src/view/views.la` (the `Lazy` variant and the import of `Found`: the file goes from 499 to 500 lines), `src/view/lazies.la`, `src/styles/paint.la`, `src/view/walk.la`, `src/view/keys.la`, `src/core/diff.la` (pattern arity only), `src/view/mapping.la`, `src/render/build.la`, `src/view/widgets.la`'s `lazy`.
- New: `src/view/findings.la` (the summary type, ~30 lines; not `found.la`, since a module named `found` would forbid `found` as a variable name, E0311), `test/lazy_found.la` (the tests below).
- Depends on DECISION-002 (resolved: keys, reuse per place) and on core decision 9. Nothing open. This PRP raises DECISION-046 (open) for the design choice below.
- No new externs, GTK calls, dependencies or pinned closures.

## REQUIREMENTS

### Must Do
- A lazy reused with its key unchanged is neither painted, rule-collected, picture-collected nor shortcut-walked again. Its subtree is used exactly as the last dispatch left it.
- Every class on screen keeps its rule: after a reuse, after a rebuild (a new key), and for lazies nested in lazies (an outer rebuilt around a reused inner). Background pictures' rules too.
- Duplicate shortcuts are still caught across the boundary: an enabled button inside an unchanged lazy against a new one outside it, and two inside one lazy built fresh. The message still names the shortcut as the first of the two wrote it.
- `painted`'s refusals (`.palette`, `.scheme` and `.dialog` below the root, `.size` on text, `.header` below the root, a window part on a node, interactive content in a `button_with`) are still caught whenever a lazy is built or rebuilt.
- `kivuli.testing`'s finders still see inside lazies, built or not. They walk unpainted views and are unchanged.

### Must NOT Do
- Change how an app writes `lazy`, or DECISION-002's reuse rules (key, place, keyed rows). Only the internal cache changes.
- Make `nodes()`/`walk` faster in general (they're the testing helpers' walk). That's a separate chore if wanted.
- Touch GTK-side cost: measured as small, apart from a reload's restyle, which is out of scope (above).

## DESIGN

**The cache: what a painted lazy contains, kept on its node.** `View.Lazy` gains a fourth field:

```la
// src/view/views.la
Lazy(string, fun(): View<Msg>, Option<View<Msg>>, Option<Found>),
```

`Found` (new, `src/view/findings.la`, has no `Msg` and imports no `views`, so there's no cycle) is what the dispatch walks need from a painted subtree:

```la
struct Found {
    rules: (string, string)[],                     // stylesheet(): (class, the whole rule), each once
    pictures: (string, Image, string, string)[],   // background_pictures()
    shortcuts: (string, string)[],                 // enabled buttons' shortcuts, in order: (GTK keys, as written)
    interactive: Option<string>,                   // check_content's finding: the first interactive thing, if any
}
```

The rule: **`Some(found)` means the built subtree is painted and `found` describes it.** `None` with a build means built but not yet painted.

- **`resolve`** (`lazies.la`): an unchanged key keeps the old `built` *and* its `found`. A new build gets `found: None`. Its inner lazies are matched as today, so an inner one whose key holds keeps its `found`.
- **`paint`** (`paint.la`): a lazy with `Some(found)` is returned as is, with no walk. One with `None` is painted, with every refusal checked as today, and its `Found` is computed from the painted subtree. That's one walk, of the new build only.
- **A shallow walk** (`walk.la`, `nodes_painted`, replacing `nodes_unchecked`): it's `nodes`, except that a lazy carrying `Found` comes out as one node and isn't looked into. `stylesheet`, `background_pictures`, `duplicate` and `check_content` walk with it, and at such a node they read its `Found`: its rules, pictures and shortcuts, merged with the same `seen` sets as now. `Found` is computed with these same four functions over the lazy's painted content, so a nested lazy's `Found` is folded into its parent's. The cost is that summary's length, never the subtree's.
- **`duplicate`**: it collects `(GTK keys, written)` in tree order from nodes and from each `Found.shortcuts`, and remembers the first spelling of each, so `first_written`'s second walk goes too.
- **`diff`, `build`, `mapping`**: arity only. `map` carries `found` over unchanged, since it holds no messages, and mapping doesn't change which buttons are enabled.

**What a message still costs:** the nodes outside unchanged lazies, which the app's `view` rebuilt anyway, plus, per unchanged lazy, the length of its summary. That's its distinct classes, pictures and shortcuts, typically tens, each a set lookup. That remainder can't be avoided without a "reused this time" marker the outer walks would trust. It's left in because it's microseconds and it keeps `add_rules` pure over the tree it's given.

**Alternatives weighed:**
- *A "reused" marker from `resolve`, with each walk skipping marked lazies*: cheaper by the summary's length, but the shortcut check needs what's inside, so it needs a summary anyway. Rejected.
- *A cache outside the tree* (in `Running`, by path and key): keyed rows move, places shift, and it duplicates `resolve`'s place matching. It also goes stale on its own. Rejected.
- *The summary in a `Styled` wrapper's `Style` around `built`*: avoids changing `View`, but it hides data in a public type that means something else. Rejected.

**Public surface (called out):** `View`'s `Lazy` variant changes from three fields to four. Apps build lazies only through `lazy(key, build)`, which is unchanged. An app that *matches* `Lazy(k, b, x)` would no longer compile. Rapidkoi doesn't (`grep "Lazy(" ../rapidkoi/src` finds nothing), and neither do the examples. `Found` isn't exported from the entry. Version **0.64.0**, since a public variant changes shape.

**Errors:** none new. The "lazy reached the diff unresolved" panics keep their text.

## TESTS
- [ ] Pure, `test/lazy_found.la`, each failing first:
  - an unchanged lazy across three resolve+paint rounds keeps its `Found`, and painting it calls no build and no `rules` (a build counter, and `Found` compared by its printed form before and after);
  - an unchanged lazy's subtree comes out of `painted` identical, printed, to the last dispatch's;
  - a rebuilt lazy (new key) gets a fresh `Found` with its new classes; `stylesheet` over the whole tree holds every class on screen; nested: an outer rebuilt around a reused inner keeps the inner's classes;
  - `background_pictures` finds a picture inside an unchanged lazy;
  - `duplicate` finds a clash between an enabled button in an unchanged lazy and a new one outside it, named as written first; it finds two inside one fresh lazy; and a disabled button inside clashes with nothing;
  - each refusal (`.palette`, `.size` on text, `.header`, interactive `button_with` content) inside a lazy is caught on its first build and on a rebuild;
  - `check_content` on a `button_with` whose content is a painted lazy uses its `Found` (interactive inside → refused).
- [ ] Pure, scaling: a test times resolve → painted → stylesheet → background_pictures → duplicate → diff over a 2,000-node unchanged lazy and asserts it's under a small bound (e.g. 5 ms), against the ~340 ms now. It's kept as a regression guard, with a generous bound so it doesn't flake.
- [ ] Existing: `test/lazy.la`, `test/theming.la`, `test/keys.la`, `test/map.la` and `test/diff.la` updated for the fourth field.
- [ ] GTK (`test/live_lazy.la`, skipped without a display): a big unchanged lazy and a popover. Opening and closing it builds the lazy once, and the widgets under it are the same pointers before and after.

## ACCEPTANCE
- [ ] Opening a popover over a 2,000-node unchanged lazy: kivuli's steps total under 5 ms headlessly (from ~343 ms).
- [ ] Every correctness case above passes.
- [ ] `lughac test test/` (in batches), the examples' tests and `lughac fmt --check src examples test` pass, the examples using `lazy` pass `lughac check`, and every example builds.
- [ ] CHANGELOG.md updated (0.64.0). DECISION-046 resolved.
- [ ] For the user, by eye: re-run the live benchmark, whose dispatch column should now be flat across sizes, then open ⚙'s and the shield's cards in Rapidkoi over Notes.
- [ ] The note for Rapidkoi (DECISION-067, § 17), given to the user.
