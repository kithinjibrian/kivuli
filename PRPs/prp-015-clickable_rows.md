# PRP-015 — Clickable rows: a button holding any view

Status: draft, for the user's approval (session 16)
Discovery: session 16, 2026-10-18 (Rapidkoi's DECISION-020, item 0; its PRP-005 info panel rows). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `button_with(content, msg)` is a button whose content is any view, so a whole row (label, summary, arrow) is one click target.

## OBJECTIVE

Rapidkoi's info panel rows show a label, a summary and an arrow, but only the arrow is clickable: kivuli buttons hold a label and an icon, nothing more. With `button_with`, the whole row is a real GTK button. The theme draws its hover and pressed looks, it takes keyboard focus, Enter and Space press it, and screen readers call it a button. Everything buttons already have works on it: `.disabled`, `.button_style`, `.tooltip`, `.shortcut`, `.menu`, and `click` in tests. *(you)*

```
button_with(
    row([text("Account").expand(), text(summary).dim(), icon("go-next-symbolic")]).spacing(8),
    Msg.PanelGo(Page.Account),
).button_style(ButtonStyle.Subtle)

@test
fun account_opens_its_page() {
    assert_eq(kivuli.testing.click(view(m), "Account"), Ok(Msg.PanelGo(Page.Account)));
}
```

## CONTEXT

- Starting state: kivuli 0.9.0. `View.Button(label, Option<Msg>, Option<Image>)` with kivuli's own content (a label and an image); `Live.Button` (its message `Ref`, shortcut controller, label, image); button looks, shortcuts and tooltips in `styling.la`; `kivuli.testing.click` / `is_disabled` / `tooltip` / `press`, which find buttons by label or tooltip; the shortcut duplicate check in `keys.la`. `src/build.la` is at 473 lines.
- Ending state: `View.ButtonWith(content, Option<Msg>)`, `Live.ButtonWith`; `button_with` in `widgets.la`, re-exported by the entry; a new `src/buttons.la` (see "First"); tests in `test/lib.la`, `test/diff_button_with.la` (new), `test/testing.la`, `test/map.la`, `test/keys.la`, `test/theming.la`, `test/live_button_with.la` (new); `examples/frame.la`'s info panel; kivuli 0.10.0.
- Related existing code: `src/build.la` (`create_in`'s Button arm), `src/styling.la` (`style_button`, `apply_shortcut`), `src/patch.la` (`SetMsg`), `src/testing.la` (`buttons`), `src/keys.la`, `src/walk.la`, `src/paint.la`; Rapidkoi's `src/panel.la` (`info_row`, read only).
- GTK functions needed: `gtk_button_new_with_label` / `gtk_button_set_child` (bound since lugha-gtk 0.1.22); no new externs expected. If one turns out to be needed, it goes to lugha-gtk first.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### First: make room in `build.la` (own commit, no behaviour change) *(proposed: file size rule)*
- `build.la` (473) would pass 500 with the new arm. The button-making code (the Button arm's body, about 45 lines: the GtkButton, kivuli's label and image, the click handler and the shortcut controller) moves to a new `src/buttons.la` as `create_button`, next to the new `create_button_with`. Both share the click handler and shortcut controller. `buttons.la` calls GTK, so it joins rule 4's list.

### Must Do
- **`button_with(content: View<Msg>, msg: Msg): View<Msg>`**: a button showing `content` and sending `msg` on each click. *(you)*
- **The content fills the button** (not centered like a plain button's label and icon), so `.expand()` and spacers inside a row lay it out edge to edge. The button's own padding and looks are the theme's. *(proposed)*
- **The button modifiers work as on `button`**: `.disabled(true)` (no message: greyed out, sends nothing, decision 6), `.button_style(…)` (`Subtle` is flat, for rows in a card), `.tooltip`, `.shortcut`, `.menu`, and every style modifier. `.image` / `.icon` are ignored: the content is the app's. *(proposed)*
- **The content is diffed as the button's child (path index 0)**: a changed summary sets one label's text, and the button and its handler stay. A content of another kind is replaced inside the same button. Only a `button_with` that becomes something else is rebuilt. *(proposed)*
- **Nothing interactive inside** *(you)*: content holding a button (plain or `button_with`), a text field, text area, check box, drop-down or slider stops the app with a message naming it, for example `kivuli: a button_with's content can't hold a button ("Delete")`. Text, icons, images, spacers, separators, columns, rows, wraps, overlays, scrolls, keyed lists and lazy nodes are allowed. The check runs where kivuli already walks every view: the paint pass before every render, and the testing helpers (so an app's pure tests hit it too).
- **Tests find it by its tooltip, its `.name`, or any text inside it** *(you)*: `click`, `is_disabled` and `tooltip` (and `pick`'s owner, for a menu on it) match a `button_with` when the asked-for string is its tooltip, its `.name`, or the text of any `text` node in its content. Two matches are `Ambiguous`, as now. `press` finds it by its shortcut, and the duplicate-shortcut check counts it.
- `View.map`, `walk`, lazy resolution, `paint` and `path_of_name` handle it.
- **Showcase: `examples/frame.la`'s info panel** gets a card of two rows in place of plain text: "Back ‹page›  ›" (sends Back; disabled on a tab's first page) and "New tab  home  ›" (sends NewTab), flat (`Subtle`). *(proposed)*
- Docs: README (the widget, how tests find it, the rule on content), docs/DESIGN.md (the GtkButton with a child). Release 0.10.0; push on your go.

### Must NOT Do
- No `.on_click` on arbitrary nodes (not a button: no theme feedback, focus or keyboard). *(you)*
- No change to `button(label, msg)` or how tests find it.
- No list-row selection (`GtkListBox` rows, a selected row): a row is a button that sends a message; the app keeps any "selected" state in its model and shows it with `.style`.

## ERROR HANDLING REQUIREMENTS

- Interactive content is a programmer error. The check is a pure function returning `Result<(), string>` (the message), tested directly. The paint pass turns it into `Err` (which stops the app with the message, as a misplaced `.header` does), and `walk.nodes` panics with it (so testing helpers stop on it).
- Testing helpers keep their `Lookup` errors: `NotFound`, `Ambiguous`, `Silent` (a disabled `button_with`).
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- No new externs expected; the click handler is a closure pinned with `data`/`release`, connected once per widget and reading its message from a `Ref`, as a plain button's is.
- No new libraries.

## TESTS TO WRITE

Pure:
- [ ] `test/lib.la`: `button_with` makes its node; `.disabled(true)` takes its message; `.tooltip`, `.button_style`, `.shortcut` land in its style; `.icon` leaves it unchanged.
- [ ] `test/diff_button_with.la`: the message is always set (`SetMsg`); a text change in the content patches at `[0, …]` and keeps the button; content of another kind is `Replace [0]`; a `button_with` becoming a `button` is `Replace []`.
- [ ] `test/testing.la`: `click` by a text inside (a row's label, its summary), by tooltip, by `.name`; `NotFound`; `Ambiguous` (two rows sharing a summary); `Silent` (disabled); `is_disabled` and `tooltip` on one.
- [ ] `test/keys.la`: a `button_with`'s shortcut counts in the duplicate check, and `press` finds it.
- [ ] `test/map.la`: its message and its content map.
- [ ] `test/theming.la` (the check): content with a button, a text field, a check box, a nested `button_with`, inside a row or a lazy node: each an `Err` naming it; text, icons, rows, spacers: `Ok`; the paint pass returns the same `Err`.
- [ ] Before the split: the existing button tests pass unchanged after it.

Live GTK (skipped without a display), `test/live_button_with.la`:
- [ ] a `GtkButton` whose child is the content's widget (a box with the row's labels); clicking sends the message; disabled is insensitive and sends nothing.
- [ ] changing the summary updates the label in place (same button, same label widget); replacing the content keeps the button; becoming a plain button replaces it.
- [ ] `.button_style(Subtle)` gives the `flat` class; the shortcut presses it.
- [ ] How rows look (hover, focus ring, fill): checked by hand in `examples/frame.la`.

## ROLLBACK PLAN
- Branch `prp-015-clickable_rows` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [ ] A whole row is one button: clicks, keyboard and the theme's looks work on all of it; disabled rows send nothing.
- [ ] Interactive content stops the app with a message naming it, in pure tests too.
- [ ] Tests find rows by their text, tooltip or name.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines.
- [ ] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md updated; 0.10.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built (benchmark included).
- For the user: `lughac run examples/frame.la`, open Info: each row highlights as a whole on hover, clicking anywhere on it works, Tab focuses it and Enter presses it; Back is greyed out on a tab's first page.
