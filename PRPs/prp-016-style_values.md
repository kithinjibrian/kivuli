# PRP-016 — Styles as values: one door, fonts and text first

Status: draft, for the user's approval (session 17)
Discovery: session 17, 2026-10-19, in conversation (DECISION-023). *(you)*: chosen in the conversation; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: every look goes through `.style([...])`, a list of typed values merged left to right (the last item that sets a field wins), starting with the `Font` and `Text` groups and their small Tailwind-like utilities.

## OBJECTIVE

Appearance today comes three ways that overlap (`.bold()`/`.dim()`/`.size(n)` on text, `ButtonStyle`, and named looks registered with `.theme(theme([...], surface))`), and which wins depends on the order kivuli applies them. After this PRP there is one door, and no names to invent or register: a style is a value written on the node, reused through ordinary functions. *(you)*

This is the target. Every example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Font, Text, Surface, Size, Weight, Family, Paint, Border, button, column, row, spacer, text, hex };

// Reused, so named: a plain function returning values. Nothing is registered.
fun section(): Styling[] = [Font.sm(), Font.semibold(), Text.upper(), Text.wide(), Text.dim()];
fun card(): Surface = Surface { background: Some(hex("#242424")), radius: Some(12), ..Surface.theme() };

fun view(m: Model): View<Msg> = column([
    text("Storage").style([Font.h1()]),
    text("DOWNLOADS").style(section()),
    row([
        text("Folder"),
        spacer(),
        text(m.folder).style([Font.mono(), Font.tabular()]),
    ]),
    text("Pictures are kept until you clear them.").wrap().style([Text.dim(), Text.loose()]),
    text("Last cleared").style([Font.h4(), Font.thin()]),                 // h4's size, thin's weight
    text("Price").style([Font { weight: Weight.Medium, tabular: true, ..Font.theme() }]),   // a full value
    text(m.total).style([Font.size(22), Text.color(hex("#e0e0e0"))]),     // values no utility has
    column(rows).style([card()]).padding(16),                              // a surface, inline
]).spacing(10).padding(24)
```

The merge rule, field by field: the last item in the list that sets a field wins; items that leave it `Theme` don't count; a field nothing sets stays the GTK theme's. `.style(a).style(b)` is `.style(a + b)`. *(you)*

```
.style([Font.h1(), Font.thin(), Font.bold()])   // size: h1's; weight: Bold (the last that sets it)
.style([Font.bold(), Font.h1()])                // weight: h1's Black (h1 is last); size: h1's
```

## CONTEXT

- Starting state: kivuli 0.10.0. `Style` (`looks.la`) holds `size`, `bold`, `dim`, `button`, `looks` (names), `classes`, `clips`, `theme`, `header_looks`, `header_classes`. `theming.la`: `Color`, `Border`, `Surface` (`text` is a colour), `Theme`, `theme(looks, surface)`, `merged`, `css(t)` (one atomic class per property and value). `paint.la` fills classes from the root's theme before every diff. `live.show_look` loads the theme's CSS once, when the window opens. `styling.style_text` sets size and bold as Pango attributes and dim as `dim-label`.
- Ending state: new `src/typography.la` (pure): `Font`, `Text`, their enums, their utilities. `theming.la`: `Paint`, the `Styling` interface, `Sheet` (the merged result), `Surface` as a `Styling` value (no `text` field), CSS for every group. `Theme`, `theme(…)`, `.theme(…)` and named looks are gone. `Style` holds one `Sheet` (and one for the header bar) instead of `looks`/`theme`/`bold`/`dim`; `.size(n)` is for icons and pictures only. The stylesheet grows as new values appear while the app runs. Every example, test, README, DESIGN and the tutorial artifact migrated. kivuli 0.11.0.
- Lugha features relied on (checked in this session with lughac): an interface value is made where an interface type is expected, including the elements of an array argument, so `.style([Font.thin(), Text.upper()])` and `.style([Font { … }])` type-check as `Styling[]` (scratch program, session 17).
- GTK 4.22 CSS properties used (checked present in libgtk-4): `font-family`, `font-size`, `font-weight`, `font-style`, `font-variant-numeric`, `line-height`, `letter-spacing`, `text-decoration-line`, `text-transform`, `color`, and the existing `background-color`, `border`, `border-radius`, `box-shadow`. The theme's sizes copied by the presets (from GTK 4.22's `Default-light.css`): `title-1` 800/200%, `title-2` 800/150%, `title-3` 700/150%, `title-4` 700/130%, `heading` 700/110%, `body` 400/110%, `caption` 400/90%. `dim-label` is `opacity: 0.55`.
- No new externs (`gtk_css_provider_load_from_string` and `to_string` exist since lugha-gtk 0.1.23).
- Open decisions: DECISION-023 (this PRP settles fonts, text and the door; button looks and colours are the next PRP).

## IMPLEMENTATION REQUIREMENTS

### The door *(you)*
- `.style(items: Styling[])` on any node: each item applies to a `Sheet` in order. `Styling` is a public interface (`fun apply(self, s: Sheet): Sheet`), implemented by `Font`, `Text` and `Surface`.
- The `Theme` variant of every enum (and `None` of a `Surface` field) means "not set here": applying skips it. So utilities and full values merge the same way.
- `.style` folds the list into the node's `Sheet` at once, so a `Style` stays plain, comparable data (the diff compares styles with `==`).
- `.header_style(items: Styling[])`: the same door for the header bar. *(proposed)*

### Font *(you)*
```
public struct Font { family: Family, size: Size, weight: Weight, style: FontStyle, tabular: bool }
public enum Family { Theme, Sans, Serif, Monospace, Named(string, Family) }   // a named font names its fallback
public enum Size { Theme, Pt(i64), Scale(f64) }                              // points, or a multiple of the theme's size
public enum Weight { Theme, Thin, Light, Normal, Medium, Semibold, Bold, Black }   // 100 300 400 500 600 700 900
public enum FontStyle { Theme, Normal, Italic }
Font.theme()                                                                  // every field the theme's; tabular false
```
Utilities, each a `Font` setting only what its name says:
- presets copying the theme's text styles (size and weight): `h1` (title-1), `h2` (title-2), `h3` (title-3), `h4` (title-4), `heading`, `body`, `caption`;
- size alone: `xs` 0.75, `sm` 0.875, `base` 1.0, `lg` 1.125, `xl` 1.25, `xxl` 1.5 (multiples of the theme's size) *(proposed values)*; `size(pt)`;
- weight: `thin`, `light`, `normal`, `medium`, `semibold`, `bold`, `black`;
- `italic`; `sans`, `serif`, `mono`, `family(name, fallback)`; `tabular`.

### Text *(you)*
```
public struct Text { color: Paint, line_height: LineHeight, letter_spacing: i64, decoration: Decoration, transform: Transform }
public enum LineHeight { Theme, Times(f64) }
public enum Decoration { Theme, None, Underline, Strikethrough }
public enum Transform { Theme, None, Uppercase, Lowercase, Capitalize }
Text.theme()                                                                  // every field the theme's; letter_spacing 0
```
Utilities: `upper`, `lower`, `capitalize`, `underline`, `strike`, `tight` (line height 1.2), `loose` (1.6), `wide` (letter spacing 1), `narrow` (-1), `dim`, `color(c)`. *(proposed values)*

`letter_spacing` 0 means "the font's own", so it can't be set back to 0 by a later item; `Text.narrow()` / `wide()` and any non-zero value can. Same for `tabular: false`. *(proposed)*

### Paint (the minimum fonts and text need; the colour PRP extends it) *(proposed)*
```
public enum Paint { Theme, Color(Color), Faded(f64) }   // Faded: the theme's text colour at that opacity
```
`Text.dim()` is `Text { color: Paint.Faded(0.55), … }`, written as GTK CSS `color: alpha(currentColor, 0.55)`, so it follows light and dark as the theme's `dim-label` does.

### Surface becomes a value *(proposed: it follows from dropping named looks)*
- `Surface` implements `Styling` and goes in `.style([...])` directly. Its fields stay (`background`, `border`, `radius`, the four corners, `flat`) except `text`, which is now `Text.color`.
- Gone: `Theme`, `theme(looks, surface)`, `.theme(…)`, apps' look enums. kivuli's checks move to the values: a bad colour still stops at `hex`; a bad size, scale, line height or font name stops the app with a message naming the value when the view is painted.

### What goes *(you)*
- `.bold()`, `.dim()`, and `.size(n)` on text. `.size(n)` stays for icons, pictures and a button's image (a box size: layout). `.size` on text stops the app with `kivuli: .size(n) on text is gone: use .style([Font.size(n)])` so a migration can't silently lose sizes. *(proposed message)*
- Not in this PRP: `.button_style` / `ButtonStyle` (button looks come with colours, next PRP); the `Role` idea is replaced by the utilities. *(proposed)*

### The stylesheet grows while the app runs *(proposed)*
- `paint` turns each node's `Sheet` into atomic classes (one per property and value, as now) and returns the CSS rules the view uses. `live` keeps every rule seen so far and reloads the one app provider (priority 600) whenever a render brings a new one. Rules are never removed (a class costs one line; a value used once may come back).

### Migration
- Every example, test, README, docs/DESIGN.md, CHANGELOG (0.11.0, under **Breaking**) and the tutorial artifact. `examples/frame.la`'s looks become functions returning `Surface`/`Styling[]`.
- Rapidkoi is not touched from here; the CHANGELOG lists what it must change.

### Must NOT Do
- No colours beyond `Paint.Theme | Color | Faded`, no button looks, no borders or spacing groups: those are the next PRPs.
- No named-look registry under another name. No cascade: nothing inherits from a parent's `.style` except what GTK's CSS itself inherits (font and colour properties do, as in CSS).

## ERROR HANDLING REQUIREMENTS

- Malformed values are programmer errors: `paint` returns `Err(message)` and the app stops with `kivuli: …`, as now. Messages: `Size.Pt(0): a font size must be positive`, `Size.Scale(0.0): a font scale must be positive`, `LineHeight.Times(-1.0): a line height must be positive`, `Family.Named(""): a font needs a name`, `Family.Named("Inter", Family.Named(…)): a font's fallback must be Sans, Serif or Monospace`, `Paint.Faded(1.5): an opacity is between 0 and 1`. *(proposed wording)*
- The pure check is a function returning `Result<(), string>`, tested directly.

## SECURITY CONSIDERATIONS

- A font name is app data written into CSS: it is quoted, and a name holding `"`, `\`, `;`, `{` or `}` or a newline is refused (it could close the string and inject rules). Tested.
- No new externs or libraries.

## TESTS TO WRITE

Pure (no display):
- [ ] `test/typography.la` (new): every utility sets exactly its field(s) and leaves the rest `Theme`; `Font.theme()`/`Text.theme()`.
- [ ] `test/typography.la`: the merge rule: `[h1, thin, bold]`, `[bold, h1]`, a full value after a utility, `.style(a).style(b)` equals `.style(a + b)`, `Theme` fields skipped, a Surface between fonts.
- [ ] `test/typography.la`: the OBJECTIVE's `view` compiles and each node's `Sheet` is what its comment says.
- [ ] `test/theming.la`: CSS for each property: `font-weight: 100` … `900`, `font-size: 22pt`, `font-size: 110%`, `font-family: "Inter", sans-serif`, `font-style: italic`, `font-variant-numeric: tabular-nums`, `line-height: 1.6`, `letter-spacing: 1px`, `text-decoration-line: line-through`, `text-transform: uppercase`, `color: alpha(currentColor, 0.55)`, `color: #e0e0e0ff`; one class per property and value; two nodes with the same value share a class.
- [ ] `test/theming.la`: each error message above, and the font-name injection cases.
- [ ] `test/lib.la`: `.style` folds into the node's `Sheet`; `.size(n)` on text is refused by `paint`, kept on icons and pictures.
- [ ] Existing tests that used `.bold`, `.dim`, `.size` on text, `.theme` or look enums: rewritten to the new door, same intent.

Live GTK (skipped without a display), `test/live_type.la` (new):
- [ ] a text styled `[Font.bold(), Text.upper()]` carries the generated classes, and the app's provider holds their rules.
- [ ] an update to a value never seen before (`Font.size(12)` → `Font.size(30)`) adds its rule to the provider and moves the class; the widget is the same.
- [ ] `Text.color(hex("#ff0000"))` and `Text.dim()` read back from GTK as the expected colour (`lgtk_widget_rgba`): red; the theme's text colour at about 55%.
- [ ] `examples/frame.la` looks the same as before by hand (its looks migrated, not redesigned).

## ROLLBACK PLAN
- Branch `prp-016-style_values` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [ ] The OBJECTIVE's code compiles and its tests pass unchanged.
- [ ] No `.bold`, `.dim`, text `.size`, `.theme`, `theme(…)` or look enum left in kivuli, its examples or tests.
- [ ] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines.
- [ ] README, docs/DESIGN.md, CHANGELOG (Breaking), CLAUDE.md, DECISIONS updated; the tutorial artifact republished; 0.11.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built.
- For the user: `lughac run examples/frame.la` and `examples/counter.la`: the same looks as before; `examples/settings.la`'s headings and dim text.
