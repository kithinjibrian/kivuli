# PRP-043 — Check and radio menu items

Status: approved (session 91)
Discovery: session 90, 2026-10-20 (PRP-042's follow-up; the reference's `DropdownMenuCheckboxItem` and `RadioGroup`/`RadioItem`: text indented 32 px, a 16 px tick or an 8 px dot at the start). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `check_item(label, checked, on_toggle)` and `radio_group(selected, choices)` menu entries, ticked from the model, for `.menu` and `.dropdown_menu`; their look in `Menu.look()`.

## OBJECTIVE

```
.dropdown_menu([
    menu_label("Appearance"),
    check_item("Status bar", m.status_bar, Msg.StatusBar),
    check_item("Activity bar", m.activity, Msg.Activity).disabled(m.locked),
    menu_separator(),
    menu_label("Panel position"),
    radio_group(label_of(m.panel), [("Top", Msg.Panel(Pos.Top)), ("Bottom", Msg.Panel(Pos.Bottom))]),
])
assert_eq(kivuli.testing.pick(view(m), "View", "Status bar"), Ok(Msg.StatusBar(false)));
```

*(you: A)*

## REQUIREMENTS

### The entries
- `check_item<Msg>(label: string, checked: bool, on_toggle: fun(bool): Msg): MenuEntry<Msg>`: picking it sends `on_toggle(!checked)`. `.disabled(true)` greys it (no message, rule 6); `.hint(…)` works on it *(proposed)*.
- `radio_group<Msg>(selected: string, choices: (string, Msg)[]): MenuEntry<Msg>`: one entry showing an item per choice, the one labelled `selected` dotted; picking a choice sends its message (picking the selected one sends it again, as GTK would) *(proposed)*. A `selected` matching no label dots none.
- `map` maps both; `kivuli.testing.pick` finds a check item and each radio choice by label.

### On GTK
- **A check item** names a stateful action (`kv.check{n}`, a boolean state); **a radio group** one stateful action (`kv.radio{n}`, a string state) its items name with their label as the target. Made once per position and kept (rule 5); every refresh sets each state from the view, so what's ticked follows the model; activating one sends its message only (an `activate` handler is connected, so GTK doesn't flip the state itself).
- **lugha-gtk 0.1.38**: `GVariant` values (`g_variant_new_boolean`, `g_variant_new_string`, a C helper reading a string variant), `GVariantType` (`g_variant_type_new`, `_free`), `g_simple_action_new_stateful`, `g_simple_action_set_state`, `g_menu_item_set_action_and_target_value`; checked against `glib/gvariant.h`, `gio/gsimpleaction.h`, `gio/gmenu.h`. Pushed under the standing go.

### The look
- **`Part.MenuCheck`** (`popover.menu check`) and **`Part.MenuRadio`** (`popover.menu radio`). `Menu.look()` gains: both 16 × 16 at the item's start with no border or background of their own; checked (their own `:checked`, a state inside the part), the check a 16 px `object-select-symbolic`, the radio an 8 px `media-record-symbolic` (a filled dot, both in Adwaita's icons), in the item's colour.

### Must NOT Do
- No mixed (indeterminate) state, no check or radio items outside menus (check boxes are those).

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [ ] `test/menus.la`: the two entries, `.disabled`, `.hint` on a check item, `map`; `layout` shows them.
- [ ] `test/testing.la`: `pick` of a check item (the toggle) and of a radio choice.
- [ ] `test/live_dropdown_menu.la`: a check item's and a radio group's action states follow the model across an update; activating one sends its message and leaves the state to the model.
- [ ] `test/menus_ui.la`, `test/parts.la`: the two parts and the look's marks.
- [ ] lugha-gtk: a stateful action's state set and read back; a radio item's target.
- [ ] GTK parses the generated rules; the Dropdown menu section in `examples/components.la` gains an Appearance group; only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.38.0; merged, tagged; pushed only on your go.
