# PRP-014 — Context menus

Status: approved by the user (session 15); building on branch prp-014-context_menus
Discovery: session 15, 2026-10-18 (DECISION-015; Rapidkoi's DECISION-020, its tab actions). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `.menu([item(label, msg), menu_separator(), …])` gives any node a context menu, opened by a right-click, a long press or the Menu key; picking an item sends its message.

## OBJECTIVE

Rapidkoi keeps its tab actions in the info panel only (a placeholder): there's no right-click. A context menu on each tab puts "Duplicate tab", "Close tab", "Close other tabs" where people look for them, with the theme's native menu look, reachable by mouse, touch and keyboard. *(you)*

```
tab(t).menu([
    item("Duplicate tab", Msg.Duplicate(t.id)),
    item("Close tab", Msg.Close(t.id)),
    menu_separator(),
    item("Close other tabs", Msg.CloseOthers(t.id)).disabled(m.tabs.len == 1),
])

@test
fun closing_a_tab_from_its_menu() {
    assert_eq(kivuli.testing.pick(view(m), "news", "Close tab"), Ok(Msg.Close(1)));
}
```

## CONTEXT

- Starting state: kivuli 0.8.0. Buttons' `Option<Msg>` (disabled = no message, rule 6); node-rewriting modifiers (`.disabled`, `.on_submit`, `.image`) living in `src/views.la` (494 lines); `src/mapping.la` (`View.map`'s body, the one module allowed to import `views` back, DECISION-018); `kivuli.testing` (`click`, `Lookup`); lugha-gtk 0.1.27.
- Ending state: room made in `views.la` first (Must Do); a new pure `src/menus.la` (`MenuEntry`, `item`, `menu_separator`); `View.Menu`; `Live.Menu`; `kivuli.testing.pick`; tests in `test/menus.la` (new, pure), `test/lib.la`, `test/diff_menu.la` (new), `test/testing.la`, `test/map.la`, `test/live_menu.la` (new); `examples/frame.la`'s tabs; lugha-gtk 0.1.28; kivuli 0.9.0.
- GTK/GIO functions needed (checked against `/usr/include/gtk-4.0` and `gio/*.h` when declared), new in lugha-gtk 0.1.28: `GMenu` (`g_menu_new`, `g_menu_append` with a detailed action name, `g_menu_append_section`), `GSimpleActionGroup` (`g_simple_action_group_new`), `GSimpleAction` (`g_simple_action_new` without a parameter, `g_simple_action_set_enabled`, its `activate` signal), `g_action_map_add_action`, `gtk_widget_insert_action_group`; `GtkPopoverMenu` (`gtk_popover_menu_new_from_model`, `set_menu_model`), `gtk_popover_set_pointing_to` (a `GdkRectangle`), `set_has_arrow`, `gtk_popover_popup`, `gtk_widget_set_parent` / `unparent`; gestures (`gtk_gesture_click_new` with `gtk_gesture_single_set_button` 3, `gtk_gesture_long_press_new`, their `pressed` signals carrying the position: a C helper if the signal's `double` arguments need one); a shortcut for the Menu key and Shift+F10 on the node.
- Open decisions that must be resolved first: none (this resolves DECISION-015).

## IMPLEMENTATION REQUIREMENTS

### First: make room in `views.la` (own commit, no behaviour change) *(proposed: file size rule, DECISION-018's route)*
- The node-rewriting modifiers keep one-line methods in `views.la` (`.disabled`, `.on_submit`, `.image`, `.icon`, and the new `.menu`); their bodies move to `src/mapping.la` as free functions, next to `map`'s, in the one allowed `views` ↔ `mapping` cycle. About 35 lines out of `views.la`; `mapping.la`'s purpose widens to "rewriting nodes" (its doc and CLAUDE.md say so).

### The menu
- **`.menu(entries: MenuEntry<Msg>[])`** on any node: a right-click on it opens the menu at the pointer. Applied again, the new list replaces the old. *(you)*
- **Entries**: `item(label, msg)` (picking it sends `msg`), `menu_separator()` (a line between groups: a new section), and `.disabled(true)` on an item (greyed out, sends nothing: no message, rule 6). *(you; proposed: the name `menu_separator`, since `separator()` is already kivuli's line between views and Lugha has no overloading)*
- **Opening** *(you)*: a right-click (at the pointer), a long press on a touch screen (at the finger), and the Menu key or Shift+F10 while the node or something inside it has keyboard focus (at the node).
- **The look is native**: a `GtkPopoverMenu` built from a `GMenu`, so the theme draws it and arrow keys, Enter and Escape work as in every GTK app. *(proposed)*
- **Messages follow the view**: each item is a `GSimpleAction` in an action group on the node; their messages live in a `Ref` the actions read, re-pointed on every update (as buttons' are). The `GMenu` is rebuilt only when labels or separators change, and an item's enabled state follows `.disabled`. A menu that's open while its node updates stays open with the new messages. *(proposed)*
- **The diff**: `View.Menu(inner, entries)` wraps the node; its entries give `SetMenu(path, entries)` every time (messages can't be compared), its node diffs as itself. A node gaining or losing its menu is a `SetMenu` on the same widget, not a rebuild. *(proposed)*
- `View.map`, `walk`, lazy resolution and `paint` handle menus.
- **`kivuli.testing.pick(view, owner, item)`**: the message of the item labelled `item`, in the menu of the node found by `owner` (a button's label or tooltip, or a `.name`); `NotFound` (no such node or item), `Ambiguous` (two owners), `Silent` (a disabled item). *(you)*
- **Showcase: `examples/frame.la`'s tabs** get "Duplicate tab", "Close tab", a separator, and "Close other tabs" (disabled with one tab). *(proposed)*
- Docs: README (`.menu`, entries, `pick`), docs/DESIGN.md (the popover menu, actions, gestures). Release 0.9.0; push on your go.

### Must NOT Do
- No submenus, no check or radio items, no icons or keyboard accelerators shown in items, no menu bar. *(proposed)*
- No menus on other events (left-click menus: that's a button's job).

## ERROR HANDLING REQUIREMENTS

- `pick` returns `Lookup` errors as `click` does.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- New externs match their headers; action handlers are closures pinned with `data`/`release`, connected once per action (re-pointed through a `Ref`, never reconnected per update); the popover is parented to the node and unparented when the node goes.
- No new libraries (GIO is part of GTK's dependencies).

## TESTS TO WRITE

Pure:
- [ ] `test/menus.la`: `item`, `menu_separator`, `.disabled` make their entries.
- [ ] `test/lib.la`: `.menu` wraps a node; through a style wrapper.
- [ ] `test/diff_menu.la`: a menu always gives `SetMenu`; the wrapped node's own changes still diff at its path; gaining or losing a menu is `SetMenu`, not `Replace`.
- [ ] `test/testing.la`: `pick` by a button's label, by tooltip, by `.name`; unknown owner, unknown item, two owners, a disabled item.
- [ ] `test/map.la`: a menu's messages map.
- [ ] Before the split: the existing modifier tests pass unchanged after it.

Live GTK (skipped without a display), `test/live_menu.la`:
- [ ] a node with a menu has the action group and a popover menu whose model lists the labels (and a section for each separator); activating an item's action sends its message; a disabled item's action is disabled and sends nothing.
- [ ] an update re-points the messages (activating sends the new one) without rebuilding the menu model when labels are the same; new labels rebuild it.
- [ ] the gestures and the shortcut are on the node; emitting a right-click press pops the menu up (the popover is visible), and so does activating the Menu-key shortcut.
- [ ] removing the node frees its popover and actions (finalizations counted).
- [ ] lugha-gtk `test/menu.la`: each new extern.
- [ ] How the menu looks and where it opens: checked by hand in `examples/frame.la`.

## ROLLBACK PLAN
- Branch `prp-014-context_menus` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.28 is additions only.

## ACCEPTANCE CRITERIA
- [ ] Any node can have a context menu opened by right-click, long press or the Menu key; items send messages, disabled ones send nothing.
- [ ] `pick` works without GTK.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines.
- [ ] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md, DECISIONS (015 resolved) updated; 0.9.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built (benchmark included); lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/frame.la`: right-click a tab: Duplicate, Close, Close others; with one tab, Close others is greyed out; Shift+F10 on a focused tab opens it too.
