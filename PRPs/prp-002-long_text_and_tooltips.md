# PRP-002 — Long text and tooltips

Status: draft, for the user's approval
Discovery: session 3, 2026-10-17. Answers marked *(you)* were chosen in the interview; *(proposed)* are recommendations for you to confirm or change with this PRP.

## FEATURE: Modifiers for text that doesn't fit (`.wrap()`, `.wrap_at(chars)`, `.ellipsize()`), selectable text (`.selectable()`), and hover text on any node (`.tooltip(s)`).

## OBJECTIVE

An app can show long text without it widening the window: wrapped onto several lines, cut with "…", or both (wrapped, at most three lines, the last cut). Text the user may want to copy can be made selectable. Any widget can carry a tooltip, and tests can click a button by its tooltip, which tells apart icon-like buttons such as the to-do app's ✕.

```
checkbox(t.title, t.done, fun(on: bool) = Msg.Toggle(t.id, on)).ellipsize().tooltip(t.title)
button("✕", Msg.Delete(t.id)).style(ButtonStyle.Subtle).tooltip("Delete {t.title}")
text(m.description).wrap_at(50)
text(m.error).wrap().selectable()

@test
fun deleting_by_tooltip() {
    assert_eq(kivuli.testing.click(view(m), "Delete milk"), Ok(Msg.Delete(1)));
}
```

## CONTEXT

- Starting state: `src/views.la` (Style, modifiers; 460 lines), `src/build.la` (`apply_style`, `style_text`, check box creation), `src/patch.la` (`SetLabel`), `src/live_tree.la` (`Checkbox(GtkWidget, Ref<…>)`), `src/testing.la` (`click`, `is_disabled`), lugha-gtk 0.1.15.
- Ending state: those files changed, `src/views.la` split (see Must Do), tests in `test/lib.la`, `test/testing.la`, `test/live_style.la`, `test/live_forms.la`; lugha-gtk 0.1.16; the to-do app uses the new modifiers.
- Related existing code: `style_text` and `style_button` in `build.la` (how a style field reaches GTK); `SetLabel` in `patch.la`; `ticked()` in `test/live_forms.la` and `shown()` in `test/live.la` (they read check box labels today).
- Related source files: none.
- Lugha spec sections relied on: nothing new.
- GTK functions needed (all in GTK 4.22's headers), new externs in `gtk.raw`:
  - `gtk_label_set_wrap` / `get_wrap`, `gtk_label_set_wrap_mode` (`PANGO_WRAP_WORD_CHAR` = 2), `gtk_label_set_ellipsize` / `get_ellipsize` (`PANGO_ELLIPSIZE_END` = 3, `NONE` = 0), `gtk_label_set_lines` / `get_lines`, `gtk_label_set_max_width_chars` / `get_max_width_chars`, `gtk_label_set_selectable` / `get_selectable`;
  - `gtk_widget_set_tooltip_text` (`Option<string>`; `None` removes it) / `get_tooltip_text`;
  - `gtk_check_button_new`, `gtk_check_button_set_child` / `get_child`.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- Modifiers, each a `Style` field, ignored where they don't fit *(you)*:
  - `.wrap()`: long text flows onto more lines at word boundaries (breaking a word only if it's longer than a line). Uncapped: the label may ask for its whole one-line width. *(you)*
  - `.wrap_at(chars)`: wraps, and asks for at most `chars` characters of width (`max-width-chars`); `.wrap_at` implies `.wrap`. *(you)*
  - `.ellipsize()`: one line, cut at the end with "…". *(you)*
  - Both wrap and ellipsize: wrap, at most 3 lines, the last cut with "…". *(you)*
  - `.selectable()`: the user can select and copy the text. Text only. *(you)*
  - `.tooltip(s)`: hover text on any node; `""` (the default) means none. *(you)*
- Reach: wrap, wrap_at and ellipsize apply to `text` and to `checkbox` labels; selectable to `text` only. *(you)*
- Every check box shows a `GtkLabel` child kivuli owns (`gtk_check_button_new` + `set_child`), so the text modifiers reach it; `SetLabel` on a check box sets that label's text. Clicking the label still toggles the box. *(proposed: for every check box, not only styled ones, so relabelling has one path)*
- `apply_style` applies all of these on create and on `SetStyle`, defaults included, so a dropped modifier is undone (as for the other fields).
- `kivuli.testing`: `click(v, s)` and `is_disabled(v, s)` find a button whose label **or** tooltip is `s` (one button matching both counts once); new `tooltip(v, label): Result<string, Lookup>` gives the tooltip of the button labelled `label` (`""` when it has none). *(you)*
- `src/views.la` is 460 lines and this adds about 80: move the modifiers (`impl View` with `restyle`) into a new `src/modifiers.la` first, as its own commit, with no behaviour change. *(proposed: per the file size rule)*
- The to-do app: task titles `.ellipsize().tooltip(t.title)`, the ✕ button `.tooltip("Delete {t.title}")`, and its tests click ✕ by tooltip. *(proposed: the showcase)*

### Must NOT Do
- No other ellipsis positions or line counts (`.ellipsize()` is end-only; wrap+ellipsize is 3 lines): not chosen.
- No wrapping or tooltips-as-names for buttons' labels beyond tooltips (button labels stay plain GTK button labels).
- No change to how `input`, `text_area` or `button` are built.
- No change to lughac; lugha-gtk gets additions only (0.1.16).

## ERROR HANDLING REQUIREMENTS

- The modifiers can't fail. `.wrap_at(n)` with `n <= 0` means no cap (as `.wrap()`). *(proposed)*
- `kivuli.testing.click` / `is_disabled` / `tooltip` return `Lookup.NotFound` / `Ambiguous` / `Silent` as today; `Ambiguous` counts distinct buttons.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- Each new extern matches its header prototype; tooltip text from C (`get_tooltip_text`, for tests) is copied, `NULL` as `None`.
- No new closures handed to C; no new libraries.

## TESTS TO WRITE

Pure (no display):
- [ ] `test/lib.la`: each modifier lands in the style (`wrap`, `wrap_at` setting both fields, `ellipsize`, `selectable`, `tooltip`); they merge with others.
- [ ] `test/diff.la`: adding a tooltip or `.wrap()` is a `SetStyle`, not a rebuild.
- [ ] `test/testing.la`: `click` by tooltip; label and tooltip on one button count once; two buttons sharing a tooltip are `Ambiguous`; `is_disabled` by tooltip; `tooltip(v, label)` gives the text, `""` when none, `NotFound` for no such button.

Live GTK (skipped without a display):
- [ ] `test/live_style.la`: a text with `.wrap()` reads back wrap on and `WORD_CHAR`; `.wrap_at(40)` → max-width-chars 40; `.ellipsize()` → `END` and wrap off; both → wrap on, `END`, lines 3; `.selectable()` → selectable; dropping each (a message) resets it; a tooltip set, changed and removed reads back from the widget.
- [ ] `test/live_forms.la`: a check box's label is kivuli's `GtkLabel` child; `.ellipsize()` reaches it; relabelling (SetLabel) changes that label's text; emitting `toggled` still works.
- [ ] lugha-gtk `test/label.la`: each new extern set and read back.
- [ ] The to-do app's own tests: delete by tooltip.

## ROLLBACK PLAN

- Branch: `prp-002-long_text_and_tooltips` from `main`. *(you, PRP-001)*
- To abandon: leave it unmerged; lugha-gtk 0.1.16 is additions only.

## ACCEPTANCE CRITERIA
- [ ] Long text can wrap, wrap at a width, be cut with "…", or both, on text and check box labels; text can be selectable; any node can have a tooltip.
- [ ] Tests click buttons by tooltip.
- [ ] The tests above written first and passing; the full suite and the examples' tests pass before the merge.
- [ ] Every example builds; `lughac fmt --check` clean; every `public` item documented; no file over 500 lines.
- [ ] CHANGELOG.md and README.md (modifier table) updated.
- [ ] Merged into `main`.

## VALIDATION
- While building: only the tests covering each step (the user's rule).
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, `lughac test examples/todo.la examples/notes.la`, every example built; in lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/todo.la`, add a task with a very long title: it's cut with "…", its full title shows on hover, and hovering ✕ says "Delete …".
