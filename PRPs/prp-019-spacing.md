# PRP-019 — Spacing through `.style`: padding, margin, gap

Status: draft, for the user's approval (session 21)
Discovery: session 21, 2026-10-19 (DECISION-023's next group). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: a `Spacing` value in `.style([...])`: padding inside the border, margin outside it, and the gap between a box's children; `.padding(n)` and `.spacing(n)` go.

## OBJECTIVE

Today's `.padding(n)` is a margin: it sets GTK's widget margins, the room outside a node, so with a border the room lands outside the line and the content touches it. Spacing becomes a style value like `Border`, with `all` and each side, and the gap between children joins it, so all the room around and between things goes through the one door. *(you)*

This is the target. Each example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Spacing, Edges, Width, Border, Side, Line, Radius, Paint, column, row, text };

fun hairline(): Side = Side { width: Width.Px(1), color: Paint.Faded(0.15), style: Line.Solid };

/** The same room on every side. */
fun pad(n: i64): Edges = Edges { all: Width.Px(n), ..Edges.theme() };

fun view(m: Model): View<Msg> = column([
    // Padding is inside the border: the rows don't touch the line.
    column(rows).style([
        Border { all: hairline(), radius: Radius.All(12), ..Border.theme() },
        Spacing { padding: pad(16), gap: Width.Px(8), ..Spacing.theme() },
    ]),
    // Margin is outside: room below this row only; a side's own value beats all.
    row([text("a"), text("b")]).style([Spacing {
        padding: Edges { all: Width.Px(4), left: Width.Px(12), ..Edges.theme() },
        margin: Edges { bottom: Width.Px(8), ..Edges.theme() },
        gap: Width.Px(6),
    }]),
    // A negative margin pulls a node toward its neighbour.
    text("tucked").style([Spacing { margin: Edges { top: Width.Px(-4), ..Edges.theme() }, ..Spacing.theme() }]),
]).style([Spacing { padding: pad(24), gap: Width.Px(12), ..Spacing.theme() }])
```

How it resolves, as `Border` does *(you)*: within one value a side's own field beats `all`; across values the later wins, so a later `all` resets sides an earlier value set; a field nothing sets stays the theme's.

## CONTEXT

- Starting state: kivuli 0.13.0. `.padding(n)` (sets `gtk_widget_set_margin_*`, a margin) and `.spacing(n)` (a box's spacing, a wrap's row and column spacing) are modifiers storing `Style.padding` / `Style.spacing`. `src/styles.la` is 421 lines, `src/sheets.la` 453.
- Ending state: `Edges` and `Spacing` beside the other style values; `Sheet.spacing`; padding and margin as atomic CSS; the gap applied through GTK's spacing properties; `.padding`, `.spacing`, `Style.padding`, `Style.spacing` gone; every example, test and doc migrated; kivuli 0.14.0.
- GTK 4.22 CSS used (parsed with no error in session 21, a C probe): `padding-{side}`, `margin-{side}` (negative too). The gap uses `gtk_box_set_spacing` and `gtk_flow_box_set_row_spacing` / `set_column_spacing` (bound already): GTK's flow box doesn't read a CSS gap.
- No new externs or libraries.
- Open decisions: none (this continues DECISION-023).

## IMPLEMENTATION REQUIREMENTS

### First: make room (own commit, no behaviour change) *(proposed: file size rule)*
- **`src/styles.la` → `src/styles.la` + `src/style_values.la`.** `Sheet` holds every group and each group's `apply` takes a `Sheet`, so the groups must stay together (moving them out would make modules import each other). What moves is the plain values the groups are made of, which never mention `Sheet`: `Family`, `Size`, `Weight`, `FontStyle`, `LineHeight`, `Decoration`, `Transform`, `Gradient`, `Stop`, `Fill`, `Width`, `Line`, `Radius`, `Side` (about 190 lines). `styles` imports `style_values`; nothing imports back. The entry re-exports from both as now.
- **`src/sheets.la` → `src/sheets.la` + `src/box_rules.la`.** The border, outline (and now padding and margin) CSS moves: `line_rules`, `keyword`, `sides`, the corner rules (about 120 lines). `sheets` imports `box_rules`; nothing imports back.

### The value *(you)*
```
public struct Edges { all: Width, top: Width, right: Width, bottom: Width, left: Width }
public struct Spacing { padding: Edges, margin: Edges, gap: Width }
Edges.theme(), Spacing.theme()   // every field the theme's
```
- `Spacing` implements `Styling`; `Sheet` gains `spacing` (each side resolved, `all` folded in).
- **Padding** is CSS `padding-{side}`: inside the border, around the content.
- **Margin** is CSS `margin-{side}`: outside the border; it may be negative.
- **Gap** is the room between a column's, row's or keyed column's children, and in a wrap both across and down; set through GTK's spacing properties, so a node that isn't a box or a wrap ignores it.

### What goes *(you)*
- The `.padding(n)` and `.spacing(n)` modifiers, and `Style.padding` / `Style.spacing`. kivuli stops setting widget margins itself (they stay GTK's 0; margin is CSS now).
- `.width`, `.height`, `.expand`, `.align` and `.place` stay modifiers: where and how big, not how much room.

### CSS *(proposed classes)*
- `kv-p-{side}-{n}` → `padding-{side}: {n}px`; `kv-m-{side}-{n}` / `kv-m-{side}-m{n}` → `margin-{side}: {n}px` (negative).

### Checks (stop the app with a message) *(proposed wording)*
- `Spacing.padding: Width.Px(-4): padding can't be negative`; `Spacing.gap: Width.Px(-2): a gap can't be negative`. Margins may be negative.

### Migration *(proposed)*
- Every example uses `.style([Spacing { … }])`; small helpers (`pad(n)`, `gap(n)`) in the examples where they repeat. `examples/borders.la` gains a "Padding and margin" section: padding inside a border, margin outside, a negative margin.
- Rapidkoi is not touched; the CHANGELOG lists the change: `.padding(n)` becomes `.style([Spacing { padding: Edges { all: Width.Px(n), ..Edges.theme() }, ..Spacing.theme() }])` (or `margin`, to keep today's room outside the node), and `.spacing(n)` becomes `Spacing { gap: Width.Px(n), .. }`.

### Must NOT Do
- No `auto` margins (centring is `.align`), no percentages, no gap that differs across and down.

## ERROR HANDLING REQUIREMENTS

- Bad values are programmer errors: `paint` returns `Err(message)` and the app stops (`kivuli: …`).

## SECURITY CONSIDERATIONS

- No new externs, libraries or strings in CSS.

## TESTS TO WRITE

Pure:
- [ ] Before the splits: the existing style and sheet tests pass unchanged after them.
- [ ] `test/styles.la`: `Edges.theme()`, `Spacing.theme()` set nothing; `all` folds into each side; a side's own field beats `all`; a later `all` resets earlier sides; padding, margin and gap merge separately; the OBJECTIVE's `view` reads as written.
- [ ] `test/sheets_border.la` (box rules): CSS for each padding and margin side, a negative margin; each check message.
- [ ] `test/theming.la`: `paint` gives a spacing's classes and refuses its bad values.
- [ ] Every test that used `.padding` or `.spacing`: rewritten to `Spacing`, same intent.

Live GTK (skipped without a display):
- [ ] `test/live_style.la`: padding and margin land as classes (and widget margins stay 0); a gap sets a box's spacing and a wrap's row and column spacing, follows the model, and goes back to 0 when it goes.
- [ ] How the room looks (padding inside a border): checked by hand in `examples/borders.la`.

## ROLLBACK PLAN
- Branch `prp-019-spacing` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [ ] The OBJECTIVE's code compiles and its tests pass unchanged.
- [ ] No `.padding(` or `.spacing(` modifier left in kivuli, its examples, tests or docs.
- [ ] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines.
- [ ] README, docs/DESIGN.md, CHANGELOG (Breaking), CLAUDE.md, DECISIONS updated; the tutorial page; 0.14.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built with `-o examples/<name>`.
- For the user: `lughac run examples/borders.la`'s "Padding and margin" section, and any example: the room looks as before.
