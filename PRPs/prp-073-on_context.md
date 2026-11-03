## FEATURE: `.on_context(msg)`: a right-click, a long press or the Menu key sends a message, with no menu

## OBJECTIVE

An app answers a right-click with something it draws itself, such as a details card, instead of kivuli's popover menu. On any node, `.on_context(msg)` makes a right-click, a long press, or the Menu key or Shift+F10 while the node (or something in it) has focus send `msg`. Nothing opens: what happens next is the app's `update` and `view`.

```
tile(app).name("tile {app.id}").on_context(Msg.Details(app.id))
button("✕", Msg.Delete(t.id)).on_context(Msg.Toggle(t.id, !t.done))   // a left-click still deletes
```

In a test, with no display:

```
assert_eq(kivuli.testing.context(view(m), "tile 3"), Ok(Msg.Details(3)));
```

## CONTEXT

- **Request:** pasted into the session by the user, for Rapidkoi: a right-click on an app's tile opens that app's own details card. It's not yet in `../rapidkoi/docs/specs/kivuli-requests.md`, and there is no Rapidkoi DECISION number.
- **Read first:** `src/render/popovers.la` (`attach`: the secondary click at line 138, the long press at 149, the primary click for `.dropdown_menu`, the Menu-key shortcut; `refresh`), `src/view/mapping.la` (`with_menu`, and the `Menu` arms of the other `with_*`), `src/view/views.la` (`View.Menu`, `.menu`, `.dropdown_menu`), `src/core/diff.la` (`SetMenu`, `unmenu`), `src/render/patch.la` (`SetMenu`), `src/render/build.la` (the `Menu` arm), `src/core/live_tree.la` (`Popup`), `src/testing.la` (`pick`, `click`, `only`, `finds`), `src/view/walk.la` (`nodes`, `check_content`: a check that returns `Result`, with its panic in the caller).
- **Changed:** `src/view/menus.la` (new `Context` enum), `src/view/mapping.la`, `src/view/views.la`, `src/view/lazies.la`, `src/styles/paint.la`, `src/core/diff.la`, `src/render/patch.la`, `src/render/build.la`, `src/render/popovers.la`, `src/core/live_tree.la`, `src/testing.la`, `README.md`, `CHANGELOG.md`, `lugha.toml` (0.67.0 → 0.68.0), `examples/tasks.la`. Tests: `test/diff_menu.la`, `test/menus.la`, the printed forms in `test/lib.la`, `test/live_menu.la` and `test/live_dropdown_menu.la` where they print a `Menu`, and new files `test/context.la` and `test/live_context.la`.
- **Language:** `lughac spec` line 723: an `impl` attaches only to a type declared in the same file, so `.on_context` must go in `views.la`. A panic can't be caught (line 1042), so the conflict check is a pure function that returns a `Result` and is tested as one, the way `check_content` is.
- **GTK:** nothing new. `gtk_gesture_set_state` (`gtk/gtkgesture.h`, already declared in lugha-gtk and used by `popovers.la` for `.dropdown_menu`'s claim), `gtk_gesture_click_new`, `gtk_gesture_long_press_new` and the shortcut controller are all used as they are today. No new externs and no new dependencies.
- **Decisions:** core decision 5 (modifiers), 6 (disabled = no message), 13 (handler `Ref`s, signals connected once), DECISION-009 (one `Styled` wrapper), DECISION-015 (context menus, resolved), DECISION-018 (`views.la` full: bodies move to `mapping.la`). None of the decisions this depends on is open.

## REQUIREMENTS

### Must Do

- `.on_context(msg: Msg)` on `View`, available on any node. A right-click (button 3), a long press, or the Menu key or Shift+F10 while the node or something inside it has focus sends `msg` once. No popover opens.
- It uses the same three gestures and the same shortcut that `.menu` already attaches, made once per widget (core decision 13): a patch only changes what they do.
- **Applied again:** the new message replaces the old one. Leaving it out of the next view takes it away, keeping the widget (a `SetMenu` patch, as `.menu` gaining or losing entries does today).
- **On a button:** the primary click is the button's own. `.on_context` never changes what a left-click sends. The `.dropdown_menu` primary gesture opens only a menu and never sends the context message.
- **Innermost wins:** a gesture that sends a context message claims its event sequence (`gtk_gesture_set_state(g, 1)`, as `.dropdown_menu` does), so an enclosing node's `.menu` or `.on_context` doesn't answer the same right-click or long press. The Menu-key callback returns true when it sends, which stops the shortcut there.
- **`.menu` and `.on_context` on one node is refused:** kivuli panics when the view is built, naming the node and saying that both answer the same right-click. That covers `.dropdown_menu` too, because it is `.menu` plus a flag. Without the conflict, `.menu([])` takes a menu away, as it does today, and leaves an `.on_context` in place, so `.menu(if editable { entries } else { [] }).on_context(…)` is refused only while `editable`.
- **Disabled:** a disabled button (`on_click: None`) is insensitive in GTK and receives no events, so its `.on_context` never fires on screen. `kivuli.testing.context` reports it as `Err(Silent)`.
- `kivuli.testing.context(v, label): Result<Msg, Lookup>`: the message a right-click on the node found by `label` would send. Nodes are found as `pick` finds an owner: a button's label or tooltip, a `button_with`'s tooltip, name or a text inside, or any node's `.name`. It returns `Err(NotFound)` when no node with a context message matches, `Err(Ambiguous)` for two, and `Err(Silent)` when the match is a disabled button or a node whose right-click opens a `.menu` (use `pick` for that).
- README: `.on_context` in the modifier table next to `.menu`, and `context` among the testing helpers.
- CHANGELOG: a 0.68.0 entry. `lugha.toml`: `version = "0.68.0"`.

### Must NOT Do

- No pointer position in the message (see DESIGN, "Where the pointer was").
- No change to what `.menu` or `.dropdown_menu` do on their own. Their nesting (two menus answering one right-click) isn't changed here: the claim above is for `.on_context` only.
- No new `View` variant and no new `Live` variant (see DESIGN), so `test/live.la`'s `shown()` is untouched.
- No tag and no push: the user says when.

## DESIGN

**One slot for a node's right-click.** `View.Menu(View<Msg>, MenuEntry<Msg>[])` becomes `View.Menu(View<Msg>, Context<Msg>)`, with this in `view/menus.la`:

```
/** What a node's right-click does: open a menu of these entries (`.menu`), or send a message (`.on_context`). */
public enum Context<Msg> {
    Entries(MenuEntry<Msg>[]),
    Send(Msg),
}
```

It gets `map(f)` for `View.map`. `Patch.SetMenu(path, Context<Msg>)` carries it, with `Entries([])` for "none". Because a node has one slot, `.menu` and `.on_context` can't coexist by construction, and the refusal is about which one a modifier finds already there. The alternative, a new `View.Context(View<Msg>, Msg)` variant, would add an arm to about 20 matches, a new patch and a new `Live` arm for the same gestures. This also changes the printed form of a view with a menu (`Menu(Text("a"), Entries([Item("Open", Some(1))]))`) and of `SetMenu`, so kivuli's tests that print them are updated. Rapidkoi's sources and tests don't print `View.Menu` (checked: its `Menu(` matches are its own `Msg.Menu`).

**The modifiers** (`mapping.la`): `with_menu(v, entries)` and `with_context(v, msg)` both go through `set_context(v, ctx): Result<View<Msg>, string>`, which goes inside `Styled` and refuses an `Entries` with at least one entry where a `Send` is, and the reverse. The two modifiers `panic("kivuli: {problem}")` on `Err`. The message names the node by its `.name`, else a button's label, else the menu's first item: `kivuli: "tile 3" has both .menu and .on_context: they answer the same right-click; use one`. The `Lazy` arm stays as it is (DECISION-002). The other `with_*` keep passing through `Menu(inside, ctx)` unchanged.

**Render** (`popovers.la`): `attach(w, ctx, send)` and `refresh(p, ctx, send)` take a `Context`. `Popup` gains `sends: Ref<Option<Msg>>`. On `Send(m)`: `sends` holds `Some(m)` and the menu is refreshed with `[]`, so an open menu closes and its actions are disabled. On `Entries(e)`: `sends` holds `None` and the refresh works as it does today. The secondary-click and long-press handlers and the Menu-key callback call one `answer(g, x, y, …)`: with `Some(m)` it sends `m`, claims `g` (not for the key, which has no gesture) and returns true; otherwise it calls today's `open`. The primary-click handler calls only `open`, and only while `sends` is `None`, so a left-click never sends the context message.

**`views.la` goes past 500 lines, the user's call.** It was at 500, and `.on_context` adds 10 (9 with its comment, plus 1 import). The proposed split (moving `restyle` and `.window`'s body to `mapping.la`) freed only 7 lines, because `lughac fmt` wraps `.window`'s new call one argument per line. The user then said to ignore the split: `views.la` ends at about 510, with no split in this PRP. `.dropdown_menu`'s doc comment, which sat above `.inert` (lines 461–468), moves back above `.dropdown_menu`; the line count doesn't change.

**Where the pointer was** (the open question): **left out.** Rapidkoi centres its card. The Menu key and Shift+F10 have no pointer, so a position would have to be an `Option`, or the node's centre made up. GTK's coordinates are relative to the widget, while an app places things relative to its window or an overlay, so returning them would mean a coordinate space kivuli doesn't model yet. It stays open without breaking callers: a later `.on_context_at(fun(Option<Point>): Msg)` adds a `Context.SendAt(…)` variant beside `Send`, and `.on_context(msg)` keeps its meaning.

**Testing** (`testing.la`, 448 lines, to about 480): `context` walks `nodes(v)` and collects every `Menu(inside, ctx)` whose owner matches. Matching is `pick`'s owner test, moved into one shared `owns(inside, style, label)` used by both. A match gives `Send(m)` → `Some(m)`, or `None` when `inside` is a disabled `Button`/`ButtonWith`, and `Entries(_)` → `None`. Then `only(…)` and `Silent` for `None`, as `click` does. `pick` skips `Send` nodes.

## TESTS

- [ ] `test/context.la` (pure): `.on_context` sets a `Send`, and applied again the second replaces the first (printed form). Through `.bold()` and `.name()` in either order there's still one `Styled` wrapper. `View.map` maps the message. `set_context` returns `Err` naming the node for `.menu(…).on_context(…)`, `.on_context(…).menu(…)` and `.dropdown_menu(…).on_context(…)`. `.menu([])` keeps an `.on_context`.
- [ ] `test/context.la`, the finder: `Ok` by a button's label, by its tooltip, by a `button_with`'s text and by `.name`. `NotFound` for no match, and for a plain node with no context. `Ambiguous` for two. `Silent` for a disabled button and for a node with `.menu`. On a button, `click` still gives the button's own message and `context` gives the context one.
- [ ] `test/diff_menu.la`: `SetMenu` carries `Send`. Gaining or losing `.on_context` keeps the widget. Switching between `.menu` and `.on_context` across renders is one `SetMenu`, not a `Replace`.
- [ ] `test/menus.la`: `Context.map`.
- [ ] `test/live_context.la` (GTK, skipped with no display): emitting `pressed` on the popup's secondary click, on its long press, and running its Menu-key action each send the message once, with the popover not visible. On a button, emitting `clicked` sends the button's own message and the secondary click sends only the context message. With `.dropdown_menu([])` and `.on_context`, the primary gesture sends nothing. A re-render with a new message sends the new one. A re-render from `.menu` to `.on_context` closes an open menu, and the way back opens the menu again.
- [ ] Printed forms in `test/lib.la`, `test/live_menu.la` and `test/live_dropdown_menu.la` updated to `Entries(…)`.

## ACCEPTANCE

- [ ] The tests above pass. **You run them, as you asked:** `lughac test` in batches of 10 files (`test/` is too big for one run here), the examples' tests one at a time (`for f in tasks browser controls; do lughac test examples/$f.la || break; done`), and `lughac fmt --check src examples test`. Every example builds. I run only `lughac check` on the files I change while working.
- [ ] No file over 500 lines except `views.la` (about 510, the user's call: no split).
- [ ] README documents `.on_context` and `kivuli.testing.context`. CHANGELOG has a 0.68.0 entry. `lugha.toml` is at 0.68.0.
- [ ] A resolved DECISION entry records the one slot, the refusal and the pointer position left out.
- [ ] For you, by eye: in `examples/tasks.la`, the ✕ button gets `.on_context(Msg.Toggle(t.id, !t.done))`. Right-click ✕: the task toggles, no menu opens, and the row's own menu doesn't open either. Left-click ✕ still deletes. Focus ✕ and press Shift+F10: it toggles. Right-click elsewhere on the row: the row's menu opens as before. A long press on ✕ (touch, or holding the mouse button) toggles.
- [ ] Before any tag or push, I ask you.
