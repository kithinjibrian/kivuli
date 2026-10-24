# PRP-060 — Knowing when the window comes to the front

Status: approved and built (2026-10-24)
Request: Rapidkoi's DECISION-054, section 2 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `Sub.window_active(f)` sends `f(true)` when the app's window becomes active and `f(false)` when it stops being active.

## OBJECTIVE
An app hears when its window comes back to the front, as Rapidkoi needs to check for upgrades on return:

```
fun subscriptions(m: Model): Sub<Msg> = Sub.batch([
    Sub.every(15 * 60 * 1000, Msg.Tick),
    Sub.window_active(Msg.WindowActive),   // WindowActive(bool)
]);
```

## CONTEXT
- Read first: `src/core/effects.la` (`Sub`, `timers`, `receivers`), `src/render/live.la` (`Running`, `mount`, `resubscribe`, `sender`), `src/render/windows.la` (window-level GTK, `click_away`).
- GTK (`gtk/gtkwindow.h`): GtkWindow's read-only `is-active` property, notified as `notify::is-active`; `gboolean gtk_window_is_active (GtkWindow *window)`. lugha-gtk 0.1.47 has `connect_notify`, not `gtk_window_is_active`: lugha-gtk 0.1.48 adds it.
- `src/render/live.la` is at 458 lines: the GTK wiring goes in `windows.la` (76 lines), so `live.la` grows only by its listeners `Ref`.
- Decisions: DECISION-041 doesn't block features in today's GTK renderer (recorded with PRP-059). None other applies.

## REQUIREMENTS

### Must Do
- `Sub.WindowActive(fun(bool): Msg)`, built with `Sub.window_active(f)`; `Sub.map` and `Sub.batch` carry it; a pure `active_listeners(sub)` collects them in order, as `receivers` does.
- Changes only: no message when the subscription starts; never the same value twice in a row (kivuli keeps the last value it saw, read with `gtk_window_is_active` at mount).
- `notify::is-active` connected once, at mount (rule 5), in `windows.la`. Its handler reads the listeners from a `Ref` that `resubscribe` sets after every update; a dropped subscription sends nothing; several listeners each get one message, in batch order, each through the normal `dispatch`.
- Docs: README (subscriptions), CHANGELOG 0.56.0; `examples/browser.la` (an `application`) shows a line while its window is in the background, for the check by eye.

### Must NOT Do
- No change to `Sub.every` or `Sub.receive`; no window state kept anywhere but the app's model.
- No promise beyond GTK's: what makes a window inactive (another window in front, minimising, a desktop switch) is the desktop's call.

## DESIGN
- `windows.la`: `watch_active(window, on_change: fun(bool))` connects `notify::is-active` and calls `on_change` only when `gtk_window_is_active` differs from the last value; `Running` keeps `listening: Ref<(fun(bool): Msg)[]>`. The handler `fun(active) = for f of listening.get() { send(f(active)) }` lives in `live.la`'s `mount`.
- The change check is a small function (`active_changed(last: Ref<bool>, now: bool): bool`) so it's tested without activating a window: a hidden test window never becomes active.
- No new errors or panics.
- Security: one read-only extern; one closure, connected once per window.

## TESTS
- [ ] `test/effects.la` (pure): `Sub.window_active`'s value; `map` turns its message; `active_listeners` finds them in a batch, in order, and none in `Sub.none()`.
- [ ] `test/live_window_active.la` (GTK, skipped without a display): `active_changed` reports a change once and a repeat never; a mounted app whose subscriptions return two listeners gets one message from each per change, none after the subscription is dropped.
- [ ] lugha-gtk: `gtk_window_is_active` on a window never shown is false.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles and its tests pass
- [ ] The full suite (in batches), the examples' tests and `fmt --check` pass; every example type-checks
- [ ] For the user, by eye: in `examples/browser.la`, switching to another window shows the background line; coming back hides it
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve its DECISION-054

## NOTES FROM THE BUILD

- **The repeat check moved into `live.la`'s `on_active(app, now)`**, with the last value in `Running.active`, instead of a separate `active_changed` in `windows.la`: one function the test drives covers the repeat check, the listeners and a dropped subscription together. `windows.la`'s `watch_active` only connects the notify and reads `gtk_window_is_active`.
- `src/render/live.la` is at 489 lines: the next addition there needs a split first.
- lugha-gtk 0.1.48 (`gtk_window_is_active`) edited from here, pushed and tagged, on the user's yes. The first `lughac update` failed to fetch the new tag (E0325); the second worked.
- Mutation check: without the repeat check, `test/live_window_active.la` fails.
- The process chore approved with this PRP (`PRPs/DISCOVERY.md`, CLAUDE.md: research first, ask only real decisions) is its own commit on this branch.
