# PRP-049 — Groups in drop-downs

Status: approved (session 107, "build all")
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
- [ ] `test/dropdowns.la` (new, pure): `dropdown_grouped`'s node; empty groups left out; `selected` across groups; `map`; `dropdown` is one unnamed group; the diff's `SetChoices` on a change in a group or a heading, none without.
- [ ] `test/testing.la`: `choose` finds a choice in a later group.
- [ ] `test/live_choices.la`: a grouped drop-down's model has a section per group; picking a choice in the second group sends its message; regrouping on update keeps the widget; `dropdown`'s model is unchanged.
- [ ] `test/dropdowns_ui.la`, `test/parts.la`: the heading's look; `Part.GroupHeading`'s selector.
- [ ] lugha-gtk: a flatten model's sections and a header factory set and read back.
- [ ] GTK parses the generated rules; `examples/components.la`'s Dropdown section gains a grouped one; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.44.0, CLAUDE.md; merged, tagged; pushed only on your go.
