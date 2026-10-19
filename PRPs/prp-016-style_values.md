# PRP-016 — Styles as values: one door, fonts and text first

Status: built (sessions 17–18) on branch prp-016-style_values; merged into main
Discovery: session 17, 2026-10-19, in conversation (DECISION-023). *(you)*: chosen in the conversation; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: every look goes through `.style([...])`, a list of typed values merged left to right (the last value that sets a field wins), starting with the `Font` and `Text` groups.

## OBJECTIVE

Appearance today comes three ways that overlap (`.bold()`/`.dim()`/`.size(n)` on text, `ButtonStyle`, and named looks registered with `.theme(theme([...], surface))`), and which wins depends on the order kivuli applies them. After this PRP there is one door, and no names to invent or register: a style is a value written on the node, reused through ordinary functions. *(you)*

**Scope (the user's call):** breaking changes, and only what was discussed. Ships: `.style`, `Styling`, `Sheet`, `Font`, `Text`, `Paint` (Theme, Color, Faded), their enums, `Font.theme()`/`Text.theme()`/`Sheet.theme()`, the merge rule, the checks and the CSS. Not the utilities (a later PRP). Deleted with no replacement in this PRP: `.bold()`, `.dim()`, `.size(n)` on text, `.button_style` and `ButtonStyle`, named looks (`Surface`, `Border`, `Theme`, `theme(…)`, `.theme(…)`, `.header_style`, look enums). Backgrounds, borders, corners and button looks come back through `.style` in the colours PRP. The old examples are deleted and new ones written.

This is the target. Every example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Styling, Font, Text, Size, Weight, Family, Paint, LineHeight, Decoration, Transform, button, column, row, spacer, text, hex };

/** Reused, so named: a plain function returning values. Nothing is registered. */
fun section(): Styling[] = [
    Font { size: Size.Scale(0.875), weight: Weight.Semibold, ..Font.theme() },
    Text { transform: Transform.Uppercase, letter_spacing: 1, color: Paint.Faded(0.55), ..Text.theme() },
];

fun view(m: Model): View<Msg> = column([
    text("Storage").style([Font { size: Size.Pt(22), weight: Weight.Bold, ..Font.theme() }]),
    text("DOWNLOADS").style(section()),
    row([
        text("Folder"),
        spacer(),
        text(m.folder).style([Font { family: Family.Monospace, tabular: true, ..Font.theme() }]),
    ]),
    text("Pictures are kept until you clear them.").wrap().style([
        Text { color: Paint.Faded(0.55), line_height: LineHeight.Times(1.6), ..Text.theme() },
    ]),
    text("Price").style([
        Font { weight: Weight.Medium, ..Font.theme() },
        Text { decoration: Decoration.Strikethrough, ..Text.theme() },
    ]),
    text(m.total).style([Text { color: Paint.Color(hex("#e0e0e0")), ..Text.theme() }]),
    button("Clear cache", Msg.Clear).style([Font { weight: Weight.Bold, ..Font.theme() }]),
]).spacing(10).padding(24)
```

The merge rule, field by field: the last value in the list that sets a field wins; values that leave it `Theme` don't count; a field nothing sets stays the GTK theme's. `.style(a).style(b)` is `.style(a + b)`. *(you)*

```
let bold = Font { weight: Weight.Bold, ..Font.theme() };
let big = Font { weight: Weight.Black, size: Size.Scale(2.0), ..Font.theme() };
.style([big, Font { weight: Weight.Thin, ..Font.theme() }, bold])   // size: big's; weight: Bold (the last that sets it)
.style([bold, big])                                                // weight: Black (big is last); size: big's
```

**The bedrock first (the user's call):** this PRP ships the types, the door, the merge rule, the checks and the CSS. The small utilities (`Font.bold()`, `Text.upper()`, …) come in a later PRP on top of it; their names must not clash with the fields (`E0310`: `Font.size(…)`, `Font.family(…)`, `Font.tabular()`, `Text.color(…)` can't exist).

## CONTEXT

- Starting state: kivuli 0.10.0. `Style` (`looks.la`) holds `size`, `bold`, `dim`, `button`, `looks` (names), `classes`, `clips`, `theme`, `header_looks`, `header_classes`. `theming.la`: `Color`, `Border`, `Surface` (`text` is a colour), `Theme`, `theme(looks, surface)`, `merged`, `css(t)` (one atomic class per property and value). `paint.la` fills classes from the root's theme before every diff. `live.show_look` loads the theme's CSS once, when the window opens. `styling.style_text` sets size and bold as Pango attributes and dim as `dim-label`.
- Ending state: new `src/typography.la` (pure): `Styling`, `Sheet`, `Font`, `Text`, their enums. `theming.la`: `Color`, `hex`, `rgb`, `Paint`. New `src/sheets.la` (pure): a `Sheet`'s checks and CSS rules. `Surface`, `Border`, `Theme`, `theme(…)`, `.theme(…)`, `.header_style`, `ButtonStyle` and `.button_style` are gone. `Style` holds one `Sheet` instead of `looks`/`classes`/`clips`/`theme`/`header_*`/`bold`/`dim`/`button`; `.size(n)` is for icons and pictures only. The stylesheet grows as new values appear while the app runs. The old examples deleted, new ones written; tests, README, DESIGN and the tutorial artifact migrated. kivuli 0.11.0.
- Lugha features relied on (checked in this session with lughac): an interface value is made where an interface type is expected, including the elements of an array argument, so `.style([Font { … }, Text { … }])` type-checks as `Styling[]` (scratch program, session 17).
- GTK 4.22 CSS properties used (checked present in libgtk-4): `font-family`, `font-size`, `font-weight`, `font-style`, `font-variant-numeric`, `line-height`, `letter-spacing`, `text-decoration-line`, `text-transform`, `color`, and the existing `background-color`, `border`, `border-radius`, `box-shadow`. The theme's sizes copied by the presets (from GTK 4.22's `Default-light.css`): `title-1` 800/200%, `title-2` 800/150%, `title-3` 700/150%, `title-4` 700/130%, `heading` 700/110%, `body` 400/110%, `caption` 400/90%. `dim-label` is `opacity: 0.55`.
- No new externs (`gtk_css_provider_load_from_string` and `to_string` exist since lugha-gtk 0.1.23).
- Open decisions: DECISION-023 (this PRP settles fonts, text and the door; button looks and colours are the next PRP).

## IMPLEMENTATION REQUIREMENTS

### The door *(you)*
- `.style(items: Styling[])` on any node: each item applies to a `Sheet` in order. `Styling` is a public interface (`fun apply(self, s: Sheet): Sheet`), implemented by `Font` and `Text` (the colours PRP adds more).
- The `Theme` variant of every enum means "not set here": applying skips it, so a value changes only what it sets.
- `.style` folds the list into the node's `Sheet` at once, so a `Style` stays plain, comparable data (the diff compares styles with `==`).

### Font *(you)*
```
public struct Font { family: Family, size: Size, weight: Weight, style: FontStyle, tabular: bool }
public enum Family { Theme, Sans, Serif, Monospace, Named(string, Family) }   // a named font names its fallback
public enum Size { Theme, Pt(i64), Scale(f64) }                              // points, or a multiple of the theme's size
public enum Weight { Theme, Thin, Light, Normal, Medium, Semibold, Bold, Black }   // 100 300 400 500 600 700 900
public enum FontStyle { Theme, Normal, Italic }
Font.theme()                                                                  // every field the theme's; tabular false
```
### Text *(you)*
```
public struct Text { color: Paint, line_height: LineHeight, letter_spacing: i64, decoration: Decoration, transform: Transform }
public enum LineHeight { Theme, Times(f64) }
public enum Decoration { Theme, None, Underline, Strikethrough }
public enum Transform { Theme, None, Uppercase, Lowercase, Capitalize }
Text.theme()                                                                  // every field the theme's; letter_spacing 0
```
`letter_spacing` 0 means "the font's own", so a later value can't set it back to 0; any non-zero value can. Same for `tabular: false`. *(proposed)*

### Paint (the minimum fonts and text need; the colour PRP extends it) *(proposed)*
```
public enum Paint { Theme, Color(Color), Faded(f64) }   // Faded: the theme's text colour at that opacity
```
`Paint.Faded(0.55)` is written as GTK CSS `color: alpha(currentColor, 0.55)`, so greyed text follows light and dark as the theme's `dim-label` does.

### What goes *(you)*
- `.bold()`, `.dim()`, and `.size(n)` on text. `.size(n)` stays for icons, pictures and a button's image (a box size: layout). `.size` on text stops the app with `kivuli: .size(n) on text is gone: use .style([Font { size: Size.Pt(n), ..Font.theme() }])` so a migration can't silently lose sizes. *(proposed message)*
- `.button_style` and `ButtonStyle`. *(you)*
- Named looks: `Surface`, `Border`, `Theme`, `theme(…)`, `.theme(…)`, `.header_style`. kivuli's checks move to the values: a bad colour still stops at `hex`; a bad size, scale, line height, opacity or font name stops the app with a message naming the value when the view is painted. *(you)*

### The stylesheet grows while the app runs *(proposed)*
- `paint` turns each node's `Sheet` into atomic classes (one per property and value, as now) and returns the CSS rules the view uses. `live` keeps every rule seen so far and reloads the one app provider (priority 600) whenever a render brings a new one. Rules are never removed (a class costs one line; a value used once may come back).

### Examples *(you: delete the old ones, write new ones)*
- Deleted: counter, counters, todo, stopwatch, notes, settings, gallery, frame, download, raw, bench (the sources; built binaries are protected files, removed by the user; `assets/sample.png` kept for controls.la).
- New, each using `.style` where it looks like anything *(proposed set)*:
  - `hello.la`: the counter, styled (the README's example).
  - `type.la`: a type specimen: each `Font` and `Text` field's values on a line naming it.
  - `tasks.la`: a to-do list (input with Enter, check boxes, keyed rows, `button_with` rows, a context menu), with its own tests.
  - `browser.la`: a header bar with tabs (context menus), home tiles in a wrap, an overlay info panel, `Cmd.task` page loads, with its own tests.
  - `controls.la`: drop-down, slider, text area, icons and a picture from bytes.
  - `split.la`: two copies of a component through `map`.
  - `bench.la`: the resolve-and-diff benchmark, rewritten.
- Rapidkoi is not touched from here; the CHANGELOG lists what it must change.

### Migration
- Every test, README, docs/DESIGN.md, CHANGELOG (0.11.0, under **Breaking**), CLAUDE.md and the tutorial artifact.

### Must NOT Do
- No colours beyond `Paint.Theme | Color | Faded`, no backgrounds, borders, corners or button looks: those are the next PRP.
- No named-look registry under another name. No cascade: nothing inherits from a parent's `.style` except what GTK's CSS itself inherits (font and colour properties do, as in CSS).

## ERROR HANDLING REQUIREMENTS

- Malformed values are programmer errors: `paint` returns `Err(message)` and the app stops with `kivuli: …`, as now. Messages: `Size.Pt(0): a font size must be positive`, `Size.Scale(0.0): a font scale must be positive`, `LineHeight.Times(-1.0): a line height must be positive`, `Family.Named(""): a font needs a name`, `Family.Named("Inter", Family.Named(…)): a font's fallback must be Sans, Serif or Monospace`, `Paint.Faded(1.5): an opacity is between 0 and 1`. *(proposed wording)*
- The pure check is a function returning `Result<(), string>`, tested directly.

## SECURITY CONSIDERATIONS

- A font name is app data written into CSS: it is quoted, and a name holding `"`, `\`, `;`, `{` or `}` or a newline is refused (it could close the string and inject rules). Tested.
- No new externs or libraries.

## TESTS TO WRITE

Pure (no display):
- [x] `test/typography.la` (new): `Font.theme()`, `Text.theme()`, `Sheet.theme()` set nothing.
- [x] `test/typography.la`: the merge rule: the last setter wins, `Theme` fields skipped, fonts and text merge separately in one list, `.style(a).style(b)` equals `.style(a + b)`.
- [x] `test/typography.la`: the OBJECTIVE's `view` compiles and each node's `Sheet` is what it wrote.
- [x] `test/sheets.la` (new): CSS for each property: `font-weight: 100` … `900`, `font-size: 22pt`, `font-size: 110%`, `font-family: "Inter", sans-serif`, `font-style: italic`, `font-variant-numeric: tabular-nums`, `line-height: 1.6`, `letter-spacing: 1px`, `text-decoration-line: line-through`, `text-transform: uppercase`, `color: alpha(currentColor, 0.55)`, `color: #e0e0e0ff`; one class per property and value; two nodes with the same value share a class.
- [x] `test/sheets.la`: each error message above, and the font-name injection cases.
- [x] `test/theming.la`: colours; paint gives classes, collects the view's rules once each, refuses bad values and `.size` on text.
- [x] `test/lib.la`: `.style` folds into the node's `Sheet`; `.size(n)` on text is refused by `paint`, kept on icons and pictures.
- [x] Existing tests that used `.bold`, `.dim`, `.size` on text, `.button_style`, `.theme` or look enums: rewritten to the new door where the intent survives, deleted where it tested a removed feature.
- [x] Each new example's own tests (`tasks.la`, `browser.la`).

Live GTK (skipped without a display), `test/live_type.la` (new):
- [x] a text styled bold and upper case carries the generated classes, and the app's provider holds their rules.
- [x] an update to a value never seen before (12pt → 30pt) adds its rule to the provider and moves the class; the widget is the same.
- [x] `Paint.Color(hex("#ff0000"))` and `Paint.Faded(0.55)` read back from GTK as the expected colour (`lgtk_widget_rgba`): red; the theme's text colour at about 55%.
- [ ] The new examples, by hand: `type.la` shows each value as its line names it.

## ROLLBACK PLAN
- Branch `prp-016-style_values` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [x] The OBJECTIVE's code compiles and its tests pass unchanged.
- [x] No `.bold`, `.dim`, text `.size`, `.button_style`, `ButtonStyle`, `Surface`, `.theme`, `theme(…)`, `.header_style` or look enum left in kivuli, its examples or tests.
- [x] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG (Breaking), CLAUDE.md, DECISIONS updated; the tutorial artifact republished; 0.11.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built.
- For the user: `lughac run examples/type.la` (each value as named), `examples/hello.la`, `examples/browser.la`.

## NOTES FROM THE BUILD

- **The utilities were deferred** (the user's call, session 17): `Font.size(…)`, `Font.family(…)`, `Font.tabular()` and `Text.color(…)` can't exist beside the fields of those names (E0310, static functions included). A later PRP names them.
- **Blocked on lughac, then unblocked**: a struct update literal (`Font { …, ..Font.theme() }`) wasn't converted to an interface value where one is expected (DECISION-024). The user fixed it in lugha between sessions 17 and 18.
- **`"{"` in a Lugha string starts interpolation**: the font-name check's brace test never matched until written `"{{"`; caught by `test/sheets.la`.
- **A style's classes also go on kivuli's own label** inside a button or check box: CSS doesn't inherit `text-decoration`, so a struck-through check box showed nothing without it (tested in `test/live_type.la`).
- **`Paint.Faded` is `alpha(currentColor, x)`**, read back from GTK as the theme's text colour at that alpha; mutation-checked (with alpha 1.0 the test fails).
- **No overlay panel in the new browser example**: without backgrounds (the colours PRP), a panel over the page would be see-through, so the Info panel sits beside the page and the overlay shows a "Loading…" note while a `Cmd.task` runs.
- **Test files moved**: `test/live_look.la`'s two header-bar tests are `test/live_header.la`; its looks tests and `test/diff_header.la` are deleted (the feature is gone).
- **Repo clean-up found eleven binaries committed in PRP-014** (`git add -A` while they sat at the root): removed from the tree, kept in history (the user's call). Examples now build with `-o examples/<name>`.
