# PRP-018 — Borders and outlines through `.style`

Status: approved by the user (session 20); building on branch prp-018-borders_and_outlines
Discovery: session 20, 2026-10-19 (DECISION-023's next group). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `Border` (each side, or all of them, and the corners' radius) and `Outline` values in `.style([...])`, merged like every other style value.

## OBJECTIVE

0.11.0 removed borders and corners with named looks. This brings them back through the one door, with each side on its own or all at once, every CSS line style, rounded corners that clip what's inside them, and an outline that marks a node (selected, a drop target) without hiding the theme's keyboard focus ring. *(you)*

This is the target. Each example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Border, Side, Outline, Width, Line, Radius, Paint, column, hex, text };

/** A hairline that follows light and dark. */
fun hairline(): Side = Side { width: Width.Px(1), color: Paint.Faded(0.15), style: Line.Solid };

fun view(m: Model): View<Msg> = column([
    column(rows).style([Border { all: hairline(), radius: Radius.All(12), ..Border.theme() }]).padding(16),
    text("Settings").padding(8).style([Border { bottom: hairline(), ..Border.theme() }]),
    text("Tab").style([Border {
        all: hairline(),
        bottom: Side { width: Width.Px(3), ..Side.theme() },
        radius: Radius.Corners(8, 8, 0, 0),
        ..Border.theme()
    }]),
    column([]).height(80).style([Border {
        all: Side { width: Width.Px(2), color: Paint.Color(hex("#2a5298")), style: Line.Dashed },
        ..Border.theme()
    }]),
    text("Selected").style([Outline { width: Width.Px(2), color: Paint.Color(hex("#2a5298")), style: Line.Solid, offset: 2 }]),
])
```

How a `Border` resolves *(you)*:
- Within one value, a side's own field beats `all`, field by field: `{ all: hairline(), bottom: thick }` gives a bottom of `thick`'s width with `hairline()`'s colour and style (where `thick` leaves them `Theme`).
- Across values, the later wins, as always: a later `Border { all: … }` sets all four sides, even ones an earlier value set one by one (as a later CSS `border` resets `border-bottom`).

## CONTEXT

- Starting state: kivuli 0.12.0. `src/styles.la` (`Styling`, `Sheet` with `font`, `text`, `background`); `src/sheets.la` (checks, atomic CSS); `src/paint.la`; `src/styling.la` (applies classes); one growing app stylesheet in `live`. 0.10's corner clipping (`gtk_widget_set_overflow` with a `kv-clip` marker, PRP-009) was removed with named looks in 0.11.0.
- Ending state: `Border`, `Side`, `Outline`, `Width`, `Line`, `Radius` beside the other style values; `Sheet.border`, `Sheet.outline`; their checks and CSS; rounded nodes clip their content again; `examples/borders.la` (new); kivuli 0.13.0.
- GTK 4.22 CSS used (each parsed with no error in session 20, a C probe): `border-{side}-width`, `-style` (every value below), `-color` (with `alpha(currentColor, x)`), `border-{corner}-radius`, `outline-width`, `-style`, `-color`, `-offset` (negative too), and `:focus-visible`.
- GTK functions: `gtk_widget_set_overflow` / `get_overflow` (bound since lugha-gtk 0.1.24). No new externs.
- Open decisions: none (this continues DECISION-023).

## IMPLEMENTATION REQUIREMENTS

### The values *(you)*
```
public enum Width { Theme, Px(i64) }
public enum Line { Theme, None, Solid, Dashed, Dotted, Double, Groove, Ridge, Inset, Outset }
public enum Radius { Theme, All(i64), Corners(i64, i64, i64, i64) }   // top-left, top-right, bottom-right, bottom-left

public struct Side { width: Width, color: Paint, style: Line }
public struct Border { all: Side, top: Side, right: Side, bottom: Side, left: Side, radius: Radius }
public struct Outline { width: Width, color: Paint, style: Line, offset: i64 }   // offset: pixels, negative inside; 0 the theme's

Side.theme(), Border.theme(), Outline.theme()   // every field the theme's
```
- `Border` and `Outline` implement `Styling`; `Sheet` gains `border` (its four sides resolved, `all` folded in) and `outline`.
- `Paint.Faded(x)` on a side or an outline is the theme's text colour at that opacity: a hairline that follows light and dark.
- A later `Radius.All(n)` replaces every corner; a later `Corners(…)` replaces all four corners too (a `Radius` is one field). *(proposed)*

### Showing *(proposed)*
- **A width needs a line**: in CSS a side with a width but no style draws nothing. A side (or outline) whose width is set and whose style is left `Theme` is drawn `Solid`. `Line.None` still hides it.
- **Rounded corners clip what's inside**: GTK rounds a node's own background and border but lets children draw past the corners. A node whose radius rounds any corner gets GTK's overflow clipping, marked so it's undone when the radius goes; a widget GTK itself clips is left alone (as 0.10 did, PRP-009).
- **Outlines don't hide focus** *(you)*: an outline's rules apply under `:not(:focus-visible)`, so the theme's focus ring still shows when the node has keyboard focus.

### CSS *(proposed classes; one per property and value, as for every style)*
- `kv-bw-{side}-{n}` → `border-{side}-width: {n}px`; `kv-bs-{side}-{line}` → `border-{side}-style`; `kv-bc-{side}-{rrggbbaa}` / `kv-bcfade-{side}-{x·1000}` → `border-{side}-color`.
- `kv-br-{corner}-{n}` → `border-{corner}-radius: {n}px` (four classes, one per corner, so a later radius always replaces each).
- `kv-ow-{n}`, `kv-os-{line}`, `kv-oc-…`, `kv-oo-{n}` / `kv-oo-m{n}` → `outline-width`, `-style`, `-color`, `-offset`, each rule written `.class:not(:focus-visible) { … }`.

### Checks (stop the app with a message, as for fonts) *(proposed wording)*
- `Width.Px(-1): a width can't be negative`; `Radius.All(-4): a radius can't be negative` (and `Radius.Corners(…)` likewise); `Paint.Faded(…)` as now.

### Showcase *(proposed)*
- `examples/borders.la` (new): a specimen: each line style, one side at a time, `all` with one side overridden, each radius, a card that follows the theme, an outline next to a focusable button to show the focus ring still appears.

### Must NOT Do
- No border images, no box shadows (later, if wanted), no per-side radius beyond the four corners, no outline on one side (CSS has none).

## ERROR HANDLING REQUIREMENTS

- Bad values are programmer errors: `paint` returns `Err(message)` and the app stops (`kivuli: …`).

## SECURITY CONSIDERATIONS

- No new externs, libraries or strings in CSS (every value is a number, a colour or a fixed keyword).

## TESTS TO WRITE

Pure:
- [ ] `test/styles.la`: `Border.theme()`, `Side.theme()`, `Outline.theme()` set nothing; `all` folds into each side; a side's own field beats `all` within one value, field by field; a later `all` resets earlier sides; a later radius replaces every corner; outlines merge field by field; the OBJECTIVE's `view` reads as written.
- [ ] `test/sheets.la`: CSS for each side's width, style (every `Line`), colour (and faded), each corner's radius, a width with a `Theme` style drawn solid, `Line.None`; an outline's four properties under `:not(:focus-visible)`, a negative offset; each check message.
- [ ] `test/theming.la`: `paint` gives a border's and an outline's classes and refuses their bad values.

Live GTK (skipped without a display), `test/live_type.la`:
- [ ] a bordered, rounded node carries its classes and the stylesheet holds their rules (an outline's with `:not(:focus-visible)`); a rounded node clips its children (overflow hidden), and stops when the radius goes; a node GTK clips itself is left as it is.
- [ ] How borders and outlines look, and that the focus ring still shows: checked by hand in `examples/borders.la`.

## ROLLBACK PLAN
- Branch `prp-018-borders_and_outlines` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [ ] The OBJECTIVE's code compiles and its tests pass unchanged.
- [ ] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines (`styles.la` and `sheets.la` are proposed for a split first if either would pass it).
- [ ] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md, DECISIONS updated; the tutorial page; 0.13.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built with `-o examples/<name>`.
- For the user: `lughac run examples/borders.la`: each value as its label names it, in light and dark; Tab to the outlined button and its focus ring shows.
