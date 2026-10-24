# PRP-063 — Accessible roles, states and labels

Status: approved (2026-10-24: a style group; lugha-gtk 0.1.51; `diff.la` split first)
Request: Rapidkoi's DECISION-050, section 4 of `../rapidkoi/docs/specs/kivuli-requests.md` (contract 7: `role`, `checked`, `selected`, `label` on `box` and `text`)

## FEATURE: an `Accessible` style group: a node's role, checked and selected states and label, as a screen reader hears them.

## OBJECTIVE

```
button_with(row([square, text("Buy milk")]), Msg.Toggle(3))
    .style([Accessible.role(Role.Checkbox), Accessible.checked(true), Accessible.label("Buy milk")])
text("Notes").style([Accessible.role(Role.Heading)])
input(m.query, Msg.Query).style([Accessible.label("Search notes")])
```

A screen reader says "Buy milk, checkbox, ticked" instead of "button".

## CONTEXT
- A style group, as `Overflow` (the user, 2026-10-24): plain data, merged and reused; no `views.la` split.
- GTK 4.22 (`gtkwidget.c`): the role can be set until an assistive technology has read the widget, then GTK refuses with a critical. kivuli sets it when it builds the widget; a changed role rebuilds the node (the diff). Disabled is announced from sensitivity already (`gtkwidget.c:6238`), so kivuli's `.disabled` covers it (rule 6).
- Probed (C, GTK 4.22): a check button and a switch keep their own checked state; a GtkLabel and a check button their own label (their text). So states apply only where roles do, and an unset label on a text is its text again.
- lugha-gtk 0.1.51: `lgtk_accessible_set_role`/`_label`/`_checked`/`_selected`, resets, and test read-backs (GTK's public `gtk_test_accessible_*`).
- `diff.la` split first (its list helpers to `core/lists.la`, the user's go).

## REQUIREMENTS

### Must Do
- `src/styles/accessible.la`: `Role { Theme, Button, Checkbox, Switch, Tab, TabList, Heading, List, ListItem, Group, Separator }` (contract 7's names); `Accessible { role, checked: Option<bool>, selected: Option<bool>, label: Option<string> }`; `Accessible.role(r)`, `.checked(on)`, `.selected(on)`, `.label(text)`; the last value setting a field wins.
- `Sheet` gains `accessible` (breaking for a full `Sheet { … }` literal).
- Roles on `column`/`row`, `button_with` and `text`; ignored on other nodes. Checked and selected where roles go; reset when unset.
- A label on those and on `input` (the entry) and `text_area` (its editor); unset: reset, except a text, whose label is its text again.
- The diff: a changed role is a `Replace`, not a `SetStyle`.
- Refused under `When(…)` and inside `Inside(…)`.

### Must NOT Do
- No accessible "disabled" of its own (rule 6); no states or labels on GTK's own check buttons and switches; no relations, descriptions or heading levels (not asked).

## TESTS
- [ ] `test/accessible.la` (pure): the utilities' values; the merge; refusals; a changed role is a `Replace`, a changed label a `SetStyle`.
- [ ] `test/live_accessible.la` (GTK): a `button_with` styled as a checkbox has role 6, checked, its label; restyled unchecked and unlabelled, the same widget; a text as a heading; an input's and a text area's labels; a native checkbox keeps its own checked state.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; tests pass; the full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] For the user, by ear if they wish (Orca): a styled `button_with` read as a checkbox
- [ ] Once merged and tagged, the user is told, so Rapidkoi can resolve DECISION-050
