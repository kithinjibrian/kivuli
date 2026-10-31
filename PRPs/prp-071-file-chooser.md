## FEATURE: Commands that open the system's file chooser (open, save, folder) and send back the chosen path

## OBJECTIVE

An app asks for a file with a command, and hears back in one message which path the person chose, or that they didn't choose one. The dialog is the desktop's own (the xdg-desktop-portal chooser where there is one), modal to the app's window. kivuli reports the path and nothing else: reading or writing the file is the program's job.

```
Open => (m, Cmd.open_file(
    "Open a picture",
    [FileFilter { name: "Pictures", patterns: ["*.png", "*.jpg", "*.jpeg"] }],
    Msg.Opened,                       // Opened(Option<string>)
)),
SaveAs => (m, Cmd.save_file("Save the note", "note.txt", [], Msg.SavedTo)),
Import => (m, Cmd.choose_folder("Import a folder", Msg.Imported)),
Opened(Some(path)) => (m, Cmd.task(fun() = read(path), Msg.Read)),
Opened(None) => (m, Cmd.none()),     // cancelled: not an error
```

In a test, with no display:

```
let cmd = update(m, Msg.Open).1;
assert_eq(chooser(cmd).map(fun(c) = c.title), Ok("Open a picture"));
assert_eq(answer_chooser(cmd, Some("/tmp/x.png")), Ok(Msg.Opened(Some("/tmp/x.png"))));
```

## CONTEXT

- **Request:** Rapidkoi DECISION-089 (`../rapidkoi/docs/specs/files.md`, M1), pasted into session; it extends `kivuli-requests.md` § 7 (DECISION-020). Resolves kivuli DECISION-013.
- **Read first:** `src/core/effects.la` (`Cmd`, `map`, `commands`), `src/render/live.la` (`perform`, `Running`, `mount`), `src/render/windows.la` (`copy_text`), `src/testing.la` (`Lookup`), `src/lib.la` (exports).
- **Creates:** `src/render/choosing.la` (builds and shows the dialog), `test/choosing.la` (pure), `test/live_choosing.la` (GTK).
- **Changes:** `src/core/effects.la` (329 → ~390 lines), `src/render/live.la` (474 → ~480: one `perform` arm, one `Running` field), `src/testing.la` (407 → ~460), `src/lib.la`, `lugha.toml` (gtk 0.1.60), `examples/controls.la` (167 lines) and its tests, `README.md`, `CHANGELOG.md`, `DECISIONS.md`.
- **GTK:** `/usr/include/gtk-4.0/gtk/gtkfiledialog.h` (GTK 4.22.4 installed; the API is 4.10+): `gtk_file_dialog_new`, `set_title`, `set_modal`, `set_initial_name`, `set_filters` (a `GListModel` of `GtkFileFilter`), `open` / `save` / `select_folder` (each `(self, GtkWindow *parent, GCancellable *, GAsyncReadyCallback, gpointer)`) and their `_finish` (`(self, GAsyncResult *, GError **)` → `GFile *`). `gtkfilefilter.h`: `gtk_file_filter_new`, `set_name`, `add_pattern`. All confirmed in the headers.
- **lugha-gtk 0.1.59 has none of it** (checked: no `file_dialog`, `GAsync`, `_finish`; it has `GFile`, `g_file_get_path`, `OwnedText`). Precedent for the shape: `src/c/opening.c` (a hook struct with handler, data and release; results kept on the object and read back with a getter, since a C string can't be a closure argument).
- **Decisions:** DECISION-013 is open, blocked on lugha-gtk (below). DECISION-005 (dialogs) is about windows `view` describes; a system chooser opened by a command, owning no view, doesn't need it. DECISION-007 (non-blocking I/O) isn't needed: no I/O happens in kivuli.

## REQUIREMENTS

### Must Do

- `Cmd.open_file(title, filters, chosen)`, `Cmd.save_file(title, suggested, filters, chosen)` and `Cmd.choose_folder(title, chosen)`, `chosen: fun(Option<string>): Msg`.
- **Answered exactly once:** `chosen(Some(path))`, the absolute local path (`g_file_get_path`), or `chosen(None)` when the person cancels, the dialog fails, or the choice has no local path. The one exception: the app quits while the dialog is open (no message: the app has ended).
- **Modal to the app's window**, opened from the main loop after the update is shown: a 0 ms GLib timeout, so `init`'s command opens it after the window is presented (`mount` runs `init`'s commands before `gtk_window_present`).
- **Filters:** a name and glob patterns each, in order; the first is selected when the dialog opens; none means every file. Documented as guidance only: the file may be of any type.
- **Save:** `suggested` is the initial name (`""`: none). GTK's own overwrite confirmation stays.
- **One at a time per window:** a chooser asked for while another is open is answered `chosen(None)` at once (from the main loop, never inside `update`), and documented. Queueing was rejected: a request the person didn't see yet may be stale by the time the first closes, and a silent queue hides a program bug.
- `Cmd.map` maps `chosen`.
- **`kivuli.testing`:**
  - `chooser(cmd): Result<Chooser, Lookup>`: the chooser a command opens, found inside batches; `Err(NotFound)` for none, `Err(Ambiguous)` for two or more.
  - `answer_chooser(cmd, chosen: Option<string>): Result<Msg, Lookup>`: the message it sends for that answer; the same errors.
- Public data, so tests and apps can read it: `Chooser { mode, title, filters, suggested }`, `Choosing { Open, Save, Folder }`, `FileFilter { name, patterns }`, exported through `import kivuli;`. (`Filter` is taken: it's a style value.)

### Must NOT Do

- Read, write or check the chosen file, or trust its type: the program's job.
- Put the path anywhere but the message (no widget, no log).
- Work around the missing lugha-gtk bindings with local externs or C here (CLAUDE.md; DECISION-013).
- Several files at once, an initial folder, an accept label: not asked for; `Chooser` has room for them later.
- Take a separate "disabled" or cancellation API: cancelling is `None`.

## DESIGN

**Data (`core/effects.la`, pure):**

```
public enum Choosing { Open, Save, Folder }
public struct FileFilter { public name: string, public patterns: string[] }
public struct Chooser {
    public mode: Choosing,
    public title: string,
    public filters: FileFilter[],
    /** Save: the name the dialog starts with; `None` otherwise. */
    public suggested: Option<string>,
}
Cmd.Choose(Chooser, fun(Option<string>): Msg)
```

Three named constructors rather than one `Cmd.choose_file(Chooser { … }, f)` (the request's sketch): each case reads as one line, and `Folder` takes no filters or name. `Chooser` is still public data, so tests read it.

**Rendering (`render/choosing.la`, new):**
- `file_dialog(chooser): GtkFileDialog`: title, modal, initial name, filters. A live test builds one and reads it back without showing it.
- `choose(window, chooser, answer)`: shows it by mode through lugha-gtk's helper; the helper calls back once; kivuli reads the path, and `answer` sends `chosen(path)` through `sender(app)`.
- `Running` gains `choosing: Ref<bool>`; `perform`'s arm is one line calling into `choosing.la`. The busy check is a small function taking the `Ref` and the open function, so it's tested with a stand-in, with no dialog.

**From lugha-gtk 0.1.60 (its PRP-001, tagged):** `GtkFileDialog` (`gtk_file_dialog_new`, `unref`, `set_title`, `set_modal`, `set_initial_name` and their getters), `GtkFileFilter` (`gtk_file_filter_new`, `unref`, `set_name`, `add_pattern`), `lgtk_file_dialog_add_filter`, `lgtk_file_dialog_filter_count`/`filter` (for the live test), and `lgtk_file_dialog_choose(dialog, parent, mode, cancellable, fun(Option<string>))`: mode 0 open, 1 save, 2 folder; the closure is called exactly once on the main loop, never during the call; it holds the dialog, so kivuli unrefs it at once.

**Security:** externs only through lugha-gtk, checked there against the headers. One pinned closure per dialog, released by the helper after its one answer. No new dependency (lugha-gtk version bump). The path comes from the portal and goes to the program unchecked: documented as the program's to validate.

**Version:** kivuli 0.66.0 (minor: new commands, a new `Cmd` variant).

## TESTS

- [x] `test/choosing.la` (pure): each constructor's `Chooser` (mode, title, filters, suggested; `save_file` with `""` → `None`); `map` maps `chosen`; `commands` keeps a `Choose`.
- [x] Same file: `chooser` finds one in a batch, `Err(NotFound)` for none, `Err(Ambiguous)` for two; `answer_chooser` with `Some(path)` and `None`, and its two errors.
- [x] Same file: the busy check answers `None` while one is open, opens when none is, and frees after the answer (a stand-in for the dialog).
- [x] `test/live_choosing.la` (skipped without a display): `file_dialog` for each mode reads back its title, initial name, modal, filter count and names.
- [x] `examples/controls.la`'s tests: its Open… / Save as… buttons give the right `Chooser`, and an answer shows in the model.

## ACCEPTANCE

- [x] lugha-gtk 0.1.60 tagged with the bindings (in lugha-gtk's repo), `lugha.toml` moved to it
- [x] All tests above pass
- [x] `lughac test test/` (in batches), the examples' tests and `lughac fmt --check src examples test` pass; every example builds
- [x] README's command table and "Testing an app" updated; CHANGELOG.md updated; DECISION-013 resolved
- [x] For the user, by eye (`lughac run examples/controls.la`): Open… shows the desktop's chooser over the window, modal, with the filters listed; cancel shows "nothing chosen"; a choice shows its path; Save as… starts with the suggested name and asks before overwriting; Choose folder… gives a folder's path
