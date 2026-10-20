# PRP-042 — Dropdown menu

Status: built (session 88) on branch prp-042-dropdown-menu; destructive items left out (the user: a)
Discovery: session 87, 2026-10-20 (the reference: ui.shadcn.com/docs/components/base/dropdown-menu and its source; GTK's popover menu probed: `popover.menu > contents > … modelbutton` items, `label.title` section titles, `separator`, `accelerator` hints, `arrow.right` for a submenu, `check.left`; the popover a CSS descendant of the node it opens from). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `.dropdown_menu(entries)` opening kivuli's menu on a click; new entries (`menu_label`, `submenu`, `.hint(keys)`); menu parts; `ui.dropdown_menu(trigger, entries)` in the palette's look.

## OBJECTIVE

```
import kivuli { item, menu_label, menu_separator, submenu };
import kivuli.ui { Button, Menu };

ui.dropdown_menu(ui.button("Open", Msg.Nothing).style([Button.outline()]), [
    menu_label("My Account"),
    item("Profile", Msg.Profile).hint("Ctrl+P"),
    item("Billing", Msg.Billing),
    menu_separator(),
    submenu("Invite users", [item("Email", Msg.Email), item("Message", Msg.Message)]),
    menu_separator(),
    item("Log out", Msg.Logout),
])
tab(t).menu([item("Close", Msg.Close)]).style([Menu.look()])   // a right-click menu in the same look
```

*(you: A: GTK's popover menu; B: no check or radio items yet)*

## REQUIREMENTS

### Opening on a click
- **`.dropdown_menu(entries)`** on any view *(proposed: not `.dropdown`, which reads like the drop-down widget)*: a primary click opens the menu below the node; picking an item sends its message. On a button, the button's own message isn't sent then. The same machinery, actions and keyboard handling as `.menu` (right-click, the Menu key), which keeps working; a node can have both.

### New entries (for `.menu` and `.dropdown_menu` alike)
- `menu_label(text)`: a group's title: the entries after it, up to the next separator or label, are its group (a GMenu section with a label).
- `submenu(label, entries)`: an item opening `entries` beside the menu (GTK's nested menus: `gtk_popover_menu_new_from_model_full` with `GTK_POPOVER_MENU_NESTED`), a chevron at its end.
- `item(…).hint(keys)`: a shortcut shown at the item's end, in kivuli's key form (`"Ctrl+P"`, checked as `.shortcut` is), drawn by GTK. Only a hint: pressing it does nothing unless the app gives a button that `.shortcut` *(proposed)*.
- `MenuEntry` grows to hold these; `kivuli.testing.pick` finds items inside submenus too.

### The look: `Menu.look()` and `ui.dropdown_menu`
- **Parts** (paths, as the drop-down's are): `Part.MenuBox` (`popover.menu > contents`), `Part.MenuItem` (`popover.menu modelbutton`), `Part.MenuLabel` (`popover.menu label.title`), `Part.MenuSeparator` (`popover.menu separator`), `Part.MenuHint` (`popover.menu accelerator`).
- **`Menu.look()`**, from the reference's classes: the box `Popover` on `PopoverForeground`, a 1 px `Border` border, `Border.rounded_md()`, 4 px padding, a medium shadow, 128 px minimum width; items `Font.sm()`, padding 6 × 8, `Radius.Small`, hovered `Accent` on `AccentForeground`; labels `Font.sm()`, `Font.medium()`, padding 6 × 8; separators a 1 px `Border` line, 4 px above and below, out to the box's edges (−4 px across); hints `Font.xs()`, `MutedForeground`, wide letter spacing.
- **`ui.dropdown_menu(trigger, entries)`**: `trigger.dropdown_menu(entries).style([Menu.look()])`.

### lugha-gtk 0.1.37
- `gtk_popover_menu_new_from_model_full(model, flags)`, `g_menu_append_submenu`, `g_menu_item_new`, `g_menu_item_set_attribute` for a string (the `accel` hint: variadic in C, so a C helper `lgtk_menu_item_set_string`), `g_menu_append_item`, `g_menu_item_unref`; checked against `gio/gmenu.h` and `gtkpopovermenu.h`. Pushed under the standing go.

### For the user: destructive items
- **GTK's menu model can't mark one item for styling**: a GMenu item has no CSS class, so `item("Delete", …).destructive()` can't be coloured red through it. Choose one:
  - **(a) leave destructive items out** of this PRP *(recommended: no workaround; the reference's other demos all fit)*;
  - **(b) a destructive item as a custom child** (`gtk_popover_menu_add_child`): a kivuli-styled button placed in the menu, which then needs its own click and close handling outside GTK's menu items.

### Must NOT Do
- No check or radio items (the next PRP), no icons beside item text (GTK 4's popover menus don't show them), no menu bar.

## TESTS (narrow; type-check every test and example, run once the user's faster tests are in)
- [x] `test/menus.la`: the new entries and `.hint`'s key check; groups from labels; `pick` inside a submenu.
- [x] `test/parts.la`: the five menu parts' selectors.
- [x] `test/menus_ui.la` (new, pure): `Menu.look()`'s values; `ui.dropdown_menu` wraps its trigger.
- [x] `test/live_menu.la`: a click opens a `.dropdown_menu` (and a button's own message isn't sent); a label titles its section; a submenu opens nested; a hint shows; `.menu` still opens on a right-click.
- [x] lugha-gtk: the new bindings, a nested popover menu built and read back.
- [x] GTK parses the generated rules; `examples/components.la` gains a Dropdown menu section; only it built. How it looks and opens: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.37.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **A hinted item is its own variant, `MenuEntry.Hinted`**, so `Item`'s shape (and the tests that build it) stays.
- **"Opens on a click" is a style flag** (`Style.menu_on_click`), not a change to the menu node: styling hands it to the popup, whose primary-click gesture (capture phase, claimed) opens the menu below the node. The diff, patch and paths code for the menu node is unchanged.
- **The GTK test is `test/live_dropdown_menu.la`** (live_menu.la was near 200 lines): nested flags, a primary press opening a dropdown menu and not a context menu. That a button's own click isn't sent can't be shown with a synthetic press: by hand.
- **The look's file is `src/ui/menu_looks.la`**: `menus` is `view.menus`' name.
- **lugha-gtk 0.1.37**: `GMenuItem`, `g_menu_item_new`, `g_menu_append_item`, `g_menu_item_unref`, `lgtk_menu_item_set_string` (`src/c/menu.c`), `g_menu_append_submenu`, `gtk_popover_menu_new_from_model_full`/`get_flags`, `gtk_event_controller_set_propagation_phase`, `gtk_gesture_set_state`; its test passes; pushed, tagged.
- **Every test and example type-checks**; tests not run; GTK parsed the menu's CSS (53 lines, 0 errors); only components built.
