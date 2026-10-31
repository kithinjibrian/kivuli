## FEATURE: `Sub.window_active` reports the window coming to and leaving the front, not its own popovers taking the keyboard

## OBJECTIVE
An app writes what it writes today:

```la
fun subscriptions(m: Model): Sub<Msg> = Sub.window_active(Msg.WindowActive)   // WindowActive(bool)
```

`f(true)` arrives when the window comes to the front, and `f(false)` when another window takes it or the window is hidden. Opening and closing one of the window's own popovers, menus or drop-downs sends nothing. Today one popover sends `false`, `true`, `false`, `true` (Rapidkoi's DECISION-067, § 17).

## CONTEXT
- **Where the request comes from:** Rapidkoi's DECISION-067 (§ 17 of `../rapidkoi/docs/specs/kivuli-requests.md`). Rapidkoi checks an app against its site each time the window comes back to the front (its DECISION-054), so every popover now starts two checks and four extra messages.
- **The cause (DECISION-047):** `GtkWindow:is-active` follows keyboard focus on the window's own surface (`gtk_window_handle_focus`, `gtk/gtkwindow.c`). An autohide popover is its own surface and takes the keyboard while it's open. Being in front is the toplevel's `GDK_TOPLEVEL_STATE_FOCUSED` (Wayland's *activated*, the window manager's focus on X11), which a popover leaves alone.
- **lugha-gtk 0.1.59** (`f8f1ec2`, tag `v0.1.59`, `src/c/front.c`) adds:
  - `lgtk_window_is_front(window): i32`: shown, and the toplevel `FOCUSED`;
  - `lgtk_window_on_front(window, handler: fun(GtkWidget, i32, data), user, destroy)`: called on each real change, never twice with one value, following the surface across realize. Hiding or unrealizing a window that's in front reports 0. Connected while realized, the current value is the start and isn't reported.
- **Its test** (`../lugha-gtk/test/front.la`), on GNOME Wayland with GTK 4.22.4 and through XWayland:
  - opening a popover notifies `is-active` 2–4 times while `on_front` stays unchanged;
  - hiding the window gives `on_front` `[1, 0]`.
- **Files:**
  - `src/render/windows.la`: `watch_active` and `is_active` read the front;
  - `lugha.toml`: gtk 0.1.59, with `lugha.lock` written by `lughac` itself;
  - `src/core/effects.la`: `Sub.window_active`'s doc;
  - `README.md:304`;
  - `test/live_window_active.la`: a new test;
  - CHANGELOG.
- **Unchanged:** `live.la`'s `on_active` (drops repeats, sends once per listener) and how subscriptions start and stop.
- **Depends on:** PRP-060 (`Sub.window_active`) and DECISION-047, which this resolves.

## REQUIREMENTS

### Must Do
- `Sub.window_active` follows `lgtk_window_on_front`, and the starting state comes from `lgtk_window_is_front`.
- Its doc says what it means now: the window comes to the front, or another window takes it or the window is hidden. Its own popovers, menus and drop-downs don't count.
- A live test: a presented window with a kivuli `popover`, opened and closed, sends no `window_active` message after the window has settled.

### Must NOT Do
- Debounce or filter `is-active` in kivuli. The front state is the right signal: no timers, no guessing.
- Rename `Sub.window_active`. The name still reads right ("active" = the front window), and Rapidkoi and the examples use it.

## DESIGN
- **`windows.la`:**
  - `watch_active(window, on_change)` calls `lgtk_window_on_front(window, fun(w, front, _) = on_change(front != 0), …)`, with no user data to release. It's connected once, at mount, as now.
  - `is_active(window)` becomes `lgtk_window_is_front(window) != 0`.
  - The doc comments say "front".
- **At startup:** mount runs before the window is realized, so the starting state is false. When the window is shown in front, `on_front` reports 1 and `f(true)` is sent. That's what `notify::is-active` did before, so startup behaves the same.
- **Hiding:** `on_front` reports 0 for a window that's hidden or unrealized while in front. Before, `gtk_window_unmap` reported `is-active` false too. Same.
- **Version:** `Sub.window_active`'s meaning changes, though not its type, so it's a minor version: 0.65.0. It carries 0.64.1 (PRP-069) with it if that isn't released first.
- **New externs:** the two above, written by lugha-gtk against `gdk/gdktoplevel.h` and `gtk/gtknative.h`. kivuli imports them by name (core decision 9). No pinned closures beyond the handler, which lugha-gtk keeps for the window's life.

## TESTS
- [ ] GTK (`test/live_window_active.la`, skipped without a display): a presented window with `Sub.window_active` and a kivuli `popover`. After it settles (500 ms), the popover is opened and closed by dispatching its messages, 300 ms apart. No `window_active` message arrives after the settle point. Before this PRP it would get 2–4 on GNOME Wayland, as lugha-gtk's test saw. If the compositor doesn't make the test window active, the test still passes (nothing is sent) and prints that it couldn't tell, as lugha-gtk's does.
- [ ] The existing test (`on_active` sends once per listener, nothing on a repeat, nothing once dropped) still passes.
- [ ] Headless: nothing changes. `Sub.window_active`'s handling of subscriptions is already tested in `test/effects.la`.

## ACCEPTANCE
- [ ] `lughac test test/` (in batches), the examples' tests, `lughac fmt --check src examples test`, and `lughac check` on every example pass. Every example builds, since a new lugha-gtk is linked.
- [ ] CHANGELOG.md: 0.65.0. DECISION-047 resolved.
- [ ] For the user, by eye: in `examples/browser.la` (it uses `Sub.window_active`), or in Rapidkoi, opening and closing a popover or a menu sends no `WindowActive`, while switching to another window and back sends `false` then `true`.
- [ ] A note for Rapidkoi: § 17's second half done, so DECISION-067 can resolve.
