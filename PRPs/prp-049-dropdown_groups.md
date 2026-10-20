# PRP-049 — Groups in drop-downs

Status: built (session 110); merged into main in session 110 (its own tests run and pass)
Discovery: session 107, 2026-10-20 (the reference's select groups: a small muted heading over each group's items, a line between groups). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `dropdown_grouped(groups, selected)` and `ui.dropdown_grouped(groups, selected, placeholder)`: a drop-down whose open list shows its choices under headings.

## OBJECTIVE

```
dropdown_grouped([
    ("Fruits", [("Apple", Msg.Pick("apple")), ("Banana", Msg.Pick("banana"))]),
    ("Vegetables", [("Carrot", Msg.Pick("carrot"))]),
], m.picked)
ui.dropdown_grouped(groups, m.picked, "Pick a food")
assert_eq(kivuli.testing.choose(view(m), "food", "Carrot"), Ok(Msg.Pick("carrot")));
```

*(you: A)*

## REQUIREMENTS

### The view
- **`dropdown_grouped<Msg>(groups: (string, (string, Msg)[])[], selected: string): View<Msg>`**: the choices of every group, in order; `selected` picks the first matching label across groups (as `dropdown` does in one); a group with no choices is left out; a heading `""` shows the separating line without a heading *(proposed)*.
- `View.Dropdown` holds groups: `Dropdown((string, (string, Msg)[])[], string)`; `dropdown(choices, selected)` is one group with heading `""`, shown as today (no header row). `map`, `walk`, `kivuli.testing`, `diff` (`SetChoices` carries the groups) and `test/live.la`'s `shown()` follow.

### On GTK
- The model is a `GtkFlattenListModel` over a `GListStore` of one `GtkStringList` per group (a section per group, GTK 4.12); headers come from a `gtk_drop_down_set_header_factory` signal factory whose `GtkListHeader` shows the group's heading (found by `gtk_list_header_get_start`), with a `GtkSeparator` above every group after the first. One group with `""` gets no header factory, so `dropdown` is unchanged.
- **Verified first by a C probe** that the flatten model gives a drop-down sections and the popup shows headers, and what the header's CSS node is (`popover listview header` expected). If GTK doesn't: stop and report.
- **lugha-gtk 0.1.41**: `gtk_flatten_list_model_new`, `g_list_store_new`/`_append`, `gtk_drop_down_set_header_factory`, `gtk_list_header_get_start`/`set_child`, and the header factory's `setup`/`bind` (checked against `gtkflattenlistmodel.h`, `gio/gliststore.h`, `gtkdropdown.h`, `gtklistheader.h`). Pushed under the standing go.
- **File size (needs your approval with this PRP)**: `src/render/build.la` is at 491 lines. The drop-down's arm (its creation and `notify::selected` handler) moves to `src/render/controls.la` as `create_dropdown(…)`, beside `sync_choices`; build.la calls it.

### The look
- **`Part.GroupHeading`** (the header's node). `ui.dropdown_grouped`: `ui.dropdown`'s look, the headings `Font.xs()`, `MutedForeground`, padding 6 × 8 *(proposed)*, the line in `Border`.

### Must NOT Do
- No nested groups, no headings that can be picked, no disabled choices (PRP-050 is the whole drop-down).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] `test/dropdowns.la` (new, pure): `dropdown_grouped`'s node; empty groups left out; `selected` across groups; `map`; `dropdown` is one unnamed group; the diff's `SetChoices` on a change in a group or a heading, none without.
- [x] `test/testing.la`: `choose` finds a choice in a later group.
- [x] `test/live_choices.la`: a grouped drop-down's model has a section per group; picking a choice in the second group sends its message; regrouping on update keeps the widget; `dropdown`'s model is unchanged.
- [x] `test/dropdowns_ui.la`, `test/parts.la`: the heading's look; `Part.GroupHeading`'s selector.
- [x] lugha-gtk: a flatten model's sections and a header factory set and read back.
- [x] GTK parses the generated rules; `examples/components.la`'s Dropdown section gains a grouped one; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.44.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **Probed first** (C, `scratchpad/probe/sec.c`): a `GtkFlattenListModel` over a `GListStore` of string lists is a section model, and a drop-down with a header factory shows a `header` node before each section in `popover … listview`.
- **The node is flat, plus the groups**: `View.Dropdown((string, Msg)[], string, (string, i64)[])`, each group a heading and its size, rather than nested groups, so the pick, `choose`, the live `Choices` and the handler stay flat (positions count through every group, as GTK's do).
- **lugha-gtk 0.1.41**: a C helper (`src/c/groups.c`) builds the groups and the header factory (its `setup`/`bind` take a `GtkListHeader`, which Lugha can't name); `lgtk_drop_down_label`/`heading` read a drop-down back for tests. Pushed, tagged.
- **The split**: the drop-down's creation and handler moved from `build.la` to `controls.la` (`create_dropdown`).
- **`Part.GroupLine`** added beside `Part.GroupHeading` for the line's colour.
- **Tests run and pass**: dropdowns (4), dropdowns_ui, parts, testing, diff, lib (131 in all, pure); live_groups, live_choices, live (7, GTK). Every test and example type-checks; components built; GTK parsed the rules (85 lines, 0 errors).
- **Two test lines differ from the plan**: the GTK test is its own file, `test/live_groups.la` (not live_choices); and a drop-down always gets a `SetChoices` (its messages are re-pointed every render, as before), so there's no "none without" case.
