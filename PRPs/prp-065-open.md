# PRP-065 — Opening from outside

Status: approved (2026-10-24: a field on `App`, breaking; open only, no `NON_UNIQUE`; `live.la` split; lugha-gtk 0.1.53)
Request: Rapidkoi's DECISION-014, section 6 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: `App.on_open`: what an app is asked to open, from its command line or a second launch, arrives as a message.

## OBJECTIVE

```
fun main(): i32 = application(App { id: "app.rapidkoi.Rapidkoi", init, update, view, subscriptions, on_open: Some(Msg.Open) })
// Open(items): "/home/…/app.rkb", "rapidkoi://notes.dafifi.net/"
```

`./main other.rkb` while Rapidkoi runs opens it in the running window; the second launch exits.

## CONTEXT
- The correct shape, breaking changes aside (the user, 2026-10-24): the `App` is the program's description (Elm's `Browser.application` record), and GTK fixes `G_APPLICATION_HANDLES_OPEN` when the application is made, so it can't be a subscription. Every `App { … }` gains `on_open` (kivuli's examples and tests, Rapidkoi's `main`).
- GLib (`gioenums.h`, `gapplication.h`): `HANDLES_OPEN` (4); on the first launch `open` comes instead of `activate`; a later launch's items go to the running app's `open` and that launch exits. lugha-gtk 0.1.53: `lgtk_application_on_open` (items kept as strings: a local file's absolute path, else its URI as GIO normalises it, `rapidkoi://host` gaining a trailing `/`), argument vectors.
- With `on_open` set, kivuli gives GTK the real command line (`args()`), as GTK apps do; an app with subcommands handles those before `application`. Without it, nothing changes: GTK sees no arguments.
- `run` moved to `render/launch.la` first (the split, approved).

## REQUIREMENTS

### Must Do
- `App.on_open: Option<fun(string[]): Msg>`; `sandbox` has none.
- `on_open` set: the application made with `HANDLES_OPEN`; the window made and the app mounted on whichever of `activate` and `open` comes first; each `open` presents the window and sends `on_open(items)`, after `init`'s command.
- Once mounted, a later `activate` (a plain second launch) presents the same window rather than mounting a second.
- README, CHANGELOG 0.61.0 (breaking: the new field).

### Must NOT Do
- No `NON_UNIQUE` (not needed: request 8 keeps one program); no command-line options of kivuli's own; no desktop file or URI scheme registration (Rapidkoi's packaging).

## TESTS
- [ ] `test/live_open.la` (GTK): an app with `on_open` run with `["prog", "/tmp/…rkb", "rapidkoi://notes.dafifi.net"]` mounts and gets `Open([path, "rapidkoi://notes.dafifi.net/"])`, then quits with status 0; an app without `on_open` run with arguments mounts through `activate` and gets nothing.
- [ ] Every `App { … }` in kivuli compiles with `on_open`.

## ACCEPTANCE
- [ ] Tests pass; the full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] For the user, by eye: `examples/browser` running, a second `./examples/browser some-address` opens a tab in the first; the second exits
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve DECISION-014
