# PRP-001 — Multi-line text area

Status: approved by the user (session 3); building on branch prp-001-text_area
Discovery: session 3, 2026-10-17. Answers marked *(you)* were chosen in the interview; *(proposed)* are recommendations not yet asked, for you to confirm or change with this PRP.

## FEATURE: A scrolling, word-wrapping multi-line text field, `text_area(value, on_change)`.

## OBJECTIVE

An app can let the user write several lines (notes, descriptions, comments) the same way it uses `input`: the model holds the text, every edit sends `on_change(whole text)`, and kivuli keeps the cursor where the user is typing. The field scrolls inside itself and wraps long lines, so it never widens the window. Tests find it by a `.name(id)`.

```
fun view(m: Model): View<Msg> = column([
    input(m.title, Msg.Title).placeholder("Title"),
    text_area(m.notes, Msg.Notes).height(120).name("notes"),
]);

@test
fun typing_notes() {
    assert_eq(kivuli.testing.type_into(view(start()), "notes", "milk\neggs"), Ok(Msg.Notes("milk\neggs")));
}
```

## CONTEXT

- Starting state: `src/views.la` (View, Style, modifiers), `src/diff.la` (`SetValue`, `SetOnChange` for inputs), `src/live_tree.la`, `src/build.la`, `src/patch.la` (the input's same-text guard), `src/testing.la` (`type_into` by placeholder), lugha-gtk 0.1.14.
- Ending state: the files above changed, plus `src/lib.la` (export), tests in `test/lib.la`, `test/diff.la`, `test/testing.la`, `test/live_forms.la`; lugha-gtk 0.1.15 with `src/c/text.c`.
- Related existing code: `input` end to end (views → diff → build → patch → testing); `test/live_health.la` (the cursor test this mirrors); `../lugha-gtk/src/c/signals.c` and its `[link] c` entry (how a C wrapper ships).
- Related source files: none.
- Lugha spec sections relied on: callbacks with `data`/`release` (section 8), handles from C and `Option` of a handle (section 7–8), strings from C copied (milestone 89), `[link] c` files (section 8).
- GTK/GLib functions needed (all checked in `/usr/include/gtk-4.0`):
  - new externs in `gtk.raw`: `gtk_text_view_new`, `gtk_text_view_get_buffer` (returns a new handle type `GtkTextBuffer`), `gtk_text_view_set_wrap_mode` (`GTK_WRAP_WORD_CHAR` = 3), `gtk_text_buffer_set_text` (length -1), `connect_buffer` (`g_signal_connect_data` for a handler taking the buffer: `"changed"`), `g_free`.
  - new C wrappers (`src/c/text.c`), because GtkTextIter is a struct passed by pointer: `lgtk_text_get(buffer)` (all text, an owned C string as a handle), `lgtk_cstring_text(handle): string` (the copy Lugha keeps), `lgtk_text_cursor(buffer): i32` (the insert mark's offset), `lgtk_text_place_cursor(buffer, offset)`.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- `text_area(value: string, on_change: fun(string): Msg): View<Msg>` in `views.la`, re-exported by `lib.la`, with a doc comment and example. *(you)*
- A new `Style` field `name: string` ("" for none) and a `.name(id)` modifier on any node, not shown on screen. *(you)*
- Built as a `GtkTextView` with word wrapping (`GTK_WRAP_WORD_CHAR`) inside its own `GtkScrolledWindow`; `.height(n)`, `.width(n)`, `.expand()`, `.padding(n)`, `.align()` apply to the scrolled window. Without `.height`, a default minimum height of about four lines (80 px). *(you: scrolls and wraps; proposed: the 80 px default)*
- Every edit sends `on_change(whole text)`; Enter inserts a line; `.on_submit` and `.placeholder` are ignored on text areas. *(you)*
- Diff: `(TextArea(a, _), TextArea(b, f))` → `SetValue` when the text differs, `SetOnChange` always — the same patches and meaning as for `input`. *(you)*
- Patch: `SetValue` on a text area writes the buffer only when its text differs (as for input); when it writes, the cursor goes back to the same character offset, clamped to the new length. *(you)*
- Echoes: the buffer's `"changed"` handler is connected once at creation, reads its function from a `Ref`, and its echoes during patching are dropped by the existing `patching` flag. *(proposed: as input)*
- `kivuli.testing.type_into(v, id, text)` finds a text field by its placeholder or its name, and a text area by its name; `texts` doesn't list text-area contents (they're input, not shown text). *(you: by name; proposed: the `texts` rule)*
- `View.map` maps a text area's `on_change`. *(proposed)*

### Must NOT Do
- No placeholder drawn on text areas (GtkTextView has none; the user chose not to draw one).
- No change to how `input` behaves; its tests must pass unchanged.
- No Ctrl+Enter or other shortcuts (a later keyboard PRP).
- No change to lughac or to lugha-gtk's existing API (additions only, released as 0.1.15).
- No text formatting (bold, size) inside the text area: those modifiers are ignored on it.

## ERROR HANDLING REQUIREMENTS

- `text_area` and `.name` can't fail.
- `kivuli.testing.type_into` returns `Err(Lookup.NotFound("text field \"id\""))` or `Err(Lookup.Ambiguous(…, n))` as today; a name or placeholder matching both an input and a text area counts as ambiguous.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch (programmer error, as for the others).

## SECURITY CONSIDERATIONS

- Each new extern matches its header prototype exactly; the C wrappers compile with GTK's flags through the existing `[link]` table.
- Text from C: `lgtk_text_get` returns an owned C string as a handle; Lugha copies it (`lgtk_cstring_text`) and frees the original (`g_free`), so no keystroke leaks memory. The wrapper keeps no pointer to Lugha memory.
- The `"changed"` closure is pinned with `data`/`release` and connected once per widget; the existing `on_finalized` check covers its release.
- No new libraries: GTK only.

## TESTS TO WRITE

All pure ones run with no display:
- [ ] `test/lib.la`: `text_area("a", f)` builds the node; `on_change("b")` is `f("b")`; `.name("n")` lands in the style.
- [ ] `test/diff.la`: changed text gives `SetValue` + `SetOnChange`; same text gives only `SetOnChange`; a text area that becomes an input is `Replace`.
- [ ] `test/testing.la`: `type_into` finds a text area by name; an input by name; an input still by placeholder; a name shared by two fields is `Ambiguous`; an unknown name is `NotFound`; `texts` leaves text-area contents out.
- [ ] `test/map.la`: a mapped text area's `on_change` sends the mapped message.
- [ ] Live GTK (`test/live_forms.la`, skipped without a display):
  - typing (setting the buffer from outside a dispatch) sends `on_change` with the whole text, and the model sees it;
  - an update with the same text leaves the cursor where it was (mutation-checked: without the guard it moves);
  - the model setting a shorter text clamps the cursor to the end; a longer one keeps its offset;
  - a model-driven change sends no echo message;
  - the text area wraps (`gtk_text_view_get_wrap_mode` = 3) and sits in a scrolled window with the requested height.
- [ ] lugha-gtk `test/text.la`: set, get (no leak path), cursor get/place, `"changed"` fires.

## ROLLBACK PLAN

- Branch: `prp-001-text_area`, from `main` at the approved PRP. *(you)*
- To abandon: leave the branch unmerged (or delete it); `main` stays as it is. lugha-gtk 0.1.15 is additions only, so leaving it published is harmless.

## ACCEPTANCE CRITERIA
- [ ] `text_area(value, on_change)` works in an app, scrolls, wraps, and keeps the cursor while typing.
- [ ] `.name(id)` finds a text area (and an input) from `kivuli.testing`.
- [ ] All tests above written first and passing; existing tests unchanged and passing.
- [ ] `lughac test test/` and `lughac test examples/todo.la` pass
- [ ] Every example passes `lughac check` and `lughac build`
- [ ] `lughac fmt --check src examples test` is clean
- [ ] Every `public` item has a `/** … */` doc comment
- [ ] No file over 500 lines
- [ ] CHANGELOG.md and README.md (widget table) updated
- [ ] Merged into `main` from `prp-001-text_area`

## VALIDATION
- `lughac fmt --check src examples test`
- `lughac test test/` and `lughac test examples/todo.la`
- `for f in examples/*.la; do lughac build "$f" -o /tmp/x || exit 1; done`
- In lugha-gtk: `lughac test test/`
- For the user: `examples/notes.la` (a small notes app), run it, type several lines, check wrapping, scrolling and that the cursor never jumps.
