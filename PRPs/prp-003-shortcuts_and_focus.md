# PRP-003 — Keyboard shortcuts and focus

Status: approved by the user (session 3); building on branch prp-003-shortcuts_and_focus
Discovery: session 3, 2026-10-17. *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: Keyboard shortcuts on buttons (`.shortcut("Ctrl+N")`), and focus: `.autofocus()` and `Cmd.focus(name)`.

## OBJECTIVE

An app can give a button a key combination that presses it from anywhere in the window, start the cursor in the right field, and move the cursor back to a field after an action, as desktop apps do. Tests press shortcuts without GTK.

```
fun view(m: Model): View<Msg> = column([
    input(m.draft, Msg.Draft).name("task").autofocus(),
    button("Add", Msg.Add).shortcut("Ctrl+N"),
    button("Clear done", Msg.ClearDone).shortcut("Ctrl+Shift+Delete").disabled(!any_done(m)),
]);

// in an application's update
Add => (Model { draft: "", ..m }, Cmd.focus("task")),

@test
fun ctrl_n_adds() {
    assert_eq(kivuli.testing.press(view(m), "Ctrl+N"), Ok(Msg.Add));
}
```

## CONTEXT

- Starting state: `src/views.la` (Style, modifiers; 408 lines), `src/build.la` (`apply_style`, button creation), `src/live_tree.la` (`Button(GtkWidget, Ref<Option<Msg>>)`), `src/live.la` (`perform`, `mount`, `dispatch`), `src/effects.la` (`Cmd`), `src/testing.la`, lugha-gtk 0.1.16.
- Ending state: those changed, plus a new pure `src/keys.la` (key strings → GTK's form, and the duplicate check); tests in `test/lib.la`, `test/keys.la` (new), `test/testing.la`, `test/effects.la`, `test/live_forms.la`; lugha-gtk 0.1.17; examples use it.
- Related existing code: tooltips and `.name` (a style field reaching GTK, and finding a node by name); `Cmd.after` in `perform` (a command run after an update); the timers test (running the main loop until something happens).
- Related source files: none.
- Lugha spec sections relied on: nothing new.
- GTK functions needed (all in GTK 4.22), new externs in `gtk.raw`: handle types `GtkEventController`, `GtkShortcut`, `GtkShortcutTrigger`, `GtkShortcutAction`; `gtk_shortcut_controller_new`, `gtk_shortcut_controller_set_scope` (`GTK_SHORTCUT_SCOPE_GLOBAL` = 2), `gtk_shortcut_controller_add_shortcut`, `gtk_shortcut_controller_remove_shortcut`, `gtk_widget_add_controller`, `gtk_shortcut_new`, `gtk_shortcut_trigger_parse_string` (`Option`: `NULL` when it can't parse), `gtk_activate_action_get`, `gtk_shortcut_action_activate` (tests), `gtk_widget_grab_focus`, `gtk_root_get_focus` (tests).
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **`.shortcut(keys)`** on a button: the keys press it from anywhere in the window (a `GtkShortcutController` with global scope on the button, its shortcut triggering GTK's `activate` action, so the button shows the press). Ignored on other nodes. A style field (`shortcut: string`, `""` none), so changing or dropping it is a `SetStyle` that replaces or removes the shortcut. *(you; proposed: the mechanism)*
- A disabled button's shortcut sends nothing (it presses an insensitive button whose handler has no message). *(you)*
- **Key syntax**, parsed by kivuli's own pure code (`src/keys.la`): modifiers `Ctrl`, `Shift`, `Alt`, `Super` joined with `+`, then one key: a letter or digit, or `Enter`, `Escape`, `Delete`, `Backspace`, `Tab`, `Space`, `Home`, `End`, `PageUp`, `PageDown`, `Up`, `Down`, `Left`, `Right`, `F1`–`F12`, or a punctuation key (`Plus`, `Minus`, `Comma`, `Period`, `Slash`). Case-insensitive names; `Ctrl+N` and `ctrl+n` are the same. Turned into GTK's form (`<Control>n`). *(you; proposed: the exact key list)*
- A key string kivuli can't parse panics, naming it, when the view is built into widgets — and in `kivuli.testing.press`, so plain tests catch typos. *(you)*
- Two **enabled** buttons with the same shortcut in one view panic, naming the keys (checked when the view is shown, each update). *(you)*
- **`.autofocus()`** on any node: the widget takes keyboard focus when it's first shown — when the window opens or when a patch adds it. For a text area, its editor gets focus. Done just after the widget is placed in the window (on the main loop's next turn). *(you; proposed: the timing)*
- **`Cmd.focus(name)`**: after the update, focus moves to the node with that `.name`; a name not in the view is ignored. `Cmd.map` keeps it. *(you)*
- **`kivuli.testing.press(v, keys)`**: the message of the enabled button with that shortcut; `Err(NotFound)` when none has it, `Err(Silent)` when only a disabled one does, `Err(Ambiguous)` when two enabled ones do. *(you)*
- Showcase: the to-do app: the new-task field `.autofocus()`, Add `Ctrl+N`, Clear done `Ctrl+Shift+Delete`. *(proposed)*

### Must NOT Do
- No shortcuts without a button (`Sub.key`): not chosen.
- No shortcuts on other widgets (check boxes, fields).
- No change to how Enter works in text fields (`.on_submit`).
- No change to lughac; lugha-gtk additions only (0.1.17).

## ERROR HANDLING REQUIREMENTS

- `keys.parse(s): Result<string, string>` (the GTK form, or what's wrong): pure, tested on its own; the panics above wrap it, with messages like `kivuli: can't read the shortcut "Ctrl+Nope": unknown key "Nope"`.
- Duplicates: `keys.duplicate(view): Option<string>` names a repeated shortcut among enabled buttons; the runtime panics on `Some`.
- `Cmd.focus` with an unknown name: no error, nothing happens.

## SECURITY CONSIDERATIONS

- Each new extern matches its header; `gtk_shortcut_trigger_parse_string` returns `NULL` on bad input, declared `Option`, though kivuli only passes strings it produced.
- No new closures handed to C (the shortcut's action is GTK's own `activate`); focus on the next loop turn uses the existing timer binding (`g_timeout_add_full`, 0 ms, its closure released after it runs).

## TESTS TO WRITE

Pure:
- [ ] `test/keys.la`: `Ctrl+N` → `<Control>n`; `Ctrl+Shift+Delete` → `<Control><Shift>Delete`; `Escape`, `F5`, `Alt+Up`, `Super+Space`; case-insensitive; errors for `""`, `Ctrl+`, `Ctrl+Nope`, `Ctrl+N+M`, `Hyper+N`; `duplicate` finds two enabled buttons sharing keys, ignores a disabled one and different keys.
- [ ] `test/lib.la`: `.shortcut`, `.autofocus` land in the style.
- [ ] `test/testing.la`: `press` → message; disabled → `Silent`; none → `NotFound`; two enabled → `Ambiguous`; a bad key string panics (checked by its parse error, since a test can't expect a panic).
- [ ] `test/effects.la`: `Cmd.focus("x")` flattens and maps.

Live GTK (skipped without a display):
- [ ] `test/live_forms.la`: a button's shortcut is a global-scope controller whose trigger reads back `<Control>n`; activating it presses the button and sends its message (main loop run until it arrives); disabled → nothing sent; changing the keys replaces the trigger, dropping them removes it.
- [ ] `test/live_forms.la`: `.autofocus()` gives the field focus once the window has run a turn; `Cmd.focus("notes")` moves focus to a text area's editor; an unknown name changes nothing.
- [ ] lugha-gtk `test/shortcuts.la`: parse, controller, activate; grab and read focus.

## ROLLBACK PLAN
- Branch `prp-003-shortcuts_and_focus` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.17 is additions only.

## ACCEPTANCE CRITERIA
- [ ] Buttons take shortcuts that press them from anywhere in the window; disabled buttons' shortcuts do nothing; bad keys and duplicates panic with clear messages.
- [ ] `.autofocus()` and `Cmd.focus(name)` move keyboard focus.
- [ ] `kivuli.testing.press` works without GTK.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] Every example builds; fmt clean; public items documented; no file over 500 lines.
- [ ] CHANGELOG.md and README.md updated. Merged into `main`.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: run the to-do app; the cursor starts in the new-task field; Ctrl+N adds; Ctrl+Shift+Delete clears done tasks (and does nothing when none are done).
