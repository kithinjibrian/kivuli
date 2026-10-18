# PRP-012 — Wrapping grid: `wrap` and `keyed_wrap`

Status: built (session 13) on branch prp-012-wrapping_grid; merged into main
Discovery: session 13, 2026-10-18 (DECISION-016; Rapidkoi's DECISION-020, its home tiles). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: A container that lays children side by side and moves to a new line when a row runs out of room, re-flowing as the window resizes; positional (`wrap`) or keyed (`keyed_wrap`), with `.equal()` widths and `.max_per_line(n)`.

## OBJECTIVE

Rapidkoi's home shows app tiles in fixed rows today (a placeholder): they leave a gap on wide windows and squeeze on narrow ones. A wrapping grid re-flows them: four across when there's room, two when there isn't. *(you)*

```
keyed_wrap(m.apps.map(fun(a) = (a.id, tile(a)))).equal().max_per_line(6).spacing(12)
wrap(m.tags.map(chip)).spacing(6)
```

## CONTEXT

- Starting state: kivuli 0.6.0. `column`/`row` (`View.Column`/`Row`, `Live.Box`), `keyed_column` (`View.Keyed`, `diff_keyed`: Insert/Move/Remove by key); `src/views.la` 473 lines, `src/diff.la` 494, `src/build.la` 477 (both at the limit); lugha-gtk 0.1.25.
- Ending state: two splits first (Must Do); `View.Wrap`, `View.KeyedWrap`, `Live.Wrap`; `.equal()`, `.max_per_line(n)` (style fields); tests in `test/lib.la`, `test/diff_wrap.la` (new), `test/testing.la`, `test/map.la`, `test/live_wrap.la` (new); lugha-gtk 0.1.26; `examples/frame.la`'s home gets tiles; kivuli 0.7.0.
- GTK functions needed (`gtkflowbox.h`, to be checked against the header when declared), new in lugha-gtk 0.1.26: `gtk_flow_box_new`, `gtk_flow_box_insert` (wraps a widget in a `GtkFlowBoxChild`; `-1` appends), `gtk_flow_box_remove`, `gtk_flow_box_get_child_at_index`, `gtk_flow_box_set_homogeneous` / `get_homogeneous`, `set_max_children_per_line` / `get`, `set_row_spacing` / `set_column_spacing` (and getters), `set_selection_mode` (`GTK_SELECTION_NONE` = 0) / `get`; `gtk_widget_set_focusable` / `get_focusable` (if not bound); `gtk_widget_get_parent` (a child's wrapper).
- Open decisions that must be resolved first: none (this resolves DECISION-016).

## IMPLEMENTATION REQUIREMENTS

### Splits first, each its own commit with no behaviour change *(proposed: file size rule)*
- **`src/diff.la` → new pure `src/lazies.la`**: `resolve`, `counterpart`, `resolve_children` (lazy nodes built or reused per place, about 70 lines) move out; `live` calls `lazies.resolve`.
- **`src/build.la` → new `src/controls.la`**: the drop-down's and slider's helpers (`picked`, `sync_choices`, `labels`, `nearest`, about 70 lines) move out; `build` and `patch` import them. CLAUDE.md rule 4 lists `controls.la` among the modules that call GTK.

### The grid
- **`wrap(children: View<Msg>[])`**: children matched by position, like `row` and `column`; **`keyed_wrap(children: (string, View<Msg>)[])`**: matched by key, like `keyed_column` (adding, removing or reordering a child moves only that widget; duplicate keys panic, as for `keyed_column`). *(you)*
- Children keep their **natural widths**, like words in a sentence; **`.equal()`** makes every child as wide as the widest (GTK's homogeneous), for a tidy grid. Ignored on other nodes. *(you)*
- **`.max_per_line(n)`**: at most `n` children on a line however wide the window (`n <= 0`: no limit); at least one always fits (GTK's minimum, 1). Ignored on other nodes. *(you)*
- **`.spacing(n)`**: the gap between children, across and down (row and column spacing). *(proposed)*
- **No selection, focus only inside**: the grid only lays out; `GTK_SELECTION_NONE`, and its child wrappers aren't keyboard stops, so Tab goes straight to a tile's button. Clicks and messages come from the children, as everywhere in kivuli; a "selected" tile is the app's model plus a look. *(you)*
- A child's `.align` / `.expand` work as in a row (across); children are laid out top to bottom, line by line. *(proposed)*

### Diff and patches
- `wrap` diffs by position (`Append`, `Remove`, children's own patches); `keyed_wrap` by key, with `diff_keyed`'s `Insert` / `Move` / `Remove` (paths count children as they do in a keyed column). A wrap and a column are different kinds (`Replace`). *(proposed)*
- On the flow box: `Append` and `Insert` use `gtk_flow_box_insert` at the index; `Remove` removes the child's wrapper; `Move` removes and re-inserts, holding the child widget alive meanwhile (`gtk_widget_ref`); `Replace` swaps a child in place.
- `View.map`, `walk` (tests find buttons in a grid) and lazy resolution (`lazies`) handle both kinds.

### Docs, example, release
- README (the widgets and modifiers tables), docs/DESIGN.md (the flow box mapping, no selection). *(you)*
- **Showcase: `examples/frame.la`'s home page** shows app tiles in a `keyed_wrap(…).equal().max_per_line(6)`, with "Add a tile" and "Remove" buttons, so resizing the window re-flows them and adding or removing moves only that tile. Its own tests. *(proposed)*
- Release 0.7.0; tag once every check passes; push on your go. *(you)*

### Must NOT Do
- No selection, no filtering or sorting by GTK, no model binding (`bind_model`): children come from the view. *(you)*
- No minimum per line. *(you)*

## ERROR HANDLING REQUIREMENTS

- Duplicate keys in a `keyed_wrap` panic with the key, as `keyed_column` does.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- New externs match `gtkflowbox.h` / `gtkwidget.h`; no new libraries; no new closures handed to C.

## TESTS TO WRITE

Pure:
- [x] `test/lib.la`: `wrap`, `keyed_wrap` make their nodes; `.equal()` and `.max_per_line(n)` land in the style.
- [x] `test/diff_wrap.la`: positional appends and removes; keyed inserts, moves and removes (the same patches as a keyed column at that path); a wrap that becomes a column is `Replace`; a change of `.equal`/`.max_per_line` is `SetStyle`.
- [x] `test/testing.la`: `click` finds a button in a grid.
- [x] `test/map.la`: grids map their children's messages.
- [x] lazies: a lazy child in a keyed wrap is reused by key (as in a keyed column), in `test/lazy.la`.

Live GTK (skipped without a display), `test/live_wrap.la`:
- [x] a wrap is a `GtkFlowBox` holding the children in order (read back through `get_child_at_index`), selection none, the wrappers not focusable, a button inside still sending its message.
- [x] `.equal()`, `.max_per_line(4)` and `.spacing(8)` read back; dropping them resets them.
- [x] keyed: insert, move and remove change the order in place (the same widgets, checked by identity), nothing rebuilt; a moved child isn't finalized.
- [x] removed children are finalized (as `test/live_health.la` checks).
- [x] lugha-gtk `test/flowbox.la`: each new extern.
- [ ] The re-flow itself (lines change as the window resizes) needs a window on screen: checked by hand in `examples/frame.la`.

## ROLLBACK PLAN
- Branch `prp-012-wrapping_grid` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.26 is additions only.

## ACCEPTANCE CRITERIA
- [x] `wrap` and `keyed_wrap` lay children out in lines that re-flow; `.equal()`, `.max_per_line(n)`, `.spacing(n)` work; keyed changes move only the widgets that changed.
- [x] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md (new modules, rule 4), DECISIONS (016 resolved) updated; 0.7.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/frame.la`: resize the window and the tiles re-flow; add and remove tiles.

## NOTES FROM THE BUILD

- **Splits first, as proposed:** `diff.la` 494 → 416 (lazy resolution in `src/lazies.la`), `build.la` 477 → 418 (drop-down and slider helpers in `src/controls.la`, which CLAUDE.md rule 4 now lists among the modules calling GTK).
- **GTK caps a flow box's line at 7 children by default**: kivuli sets 65535 when no `.max_per_line` is given (and when one is dropped), so `wrap` has no surprise limit.
- **GTK's default selection is single**: kivuli sets none; each `GtkFlowBoxChild` is made not focusable right after the insert (`gtk_widget_get_parent` finds it).
- **Moves:** a flow box can't reorder, so a keyed move removes the child (GTK destroys its old wrapper) and inserts it at its new place, holding the widget with `gtk_widget_ref` meanwhile. Mutation-checked: without the reference, the moved tile is destroyed mid-move and drops out of the grid.
- **The live test reads GTK's own order** (`get_child_at_index` → the wrapper's widget, matched to kivuli's tile), not just kivuli's list.
- **Caught by the full run, not the narrow ones:** `examples/bench.la` imported `resolve` from `kivuli.diff`; it imports it from `kivuli.lazies` now.
- **The example:** `frame.la`'s home has eight app tiles in a `keyed_wrap(…).equal().max_per_line(6)`, with Add and Remove; tile names are capitalised ("Maps") so they don't clash with tab labels in `click`.
