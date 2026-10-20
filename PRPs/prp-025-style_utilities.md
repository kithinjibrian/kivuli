# PRP-025 — Style utilities

Status: approved (session 31)
Discovery: sessions 17 and 27 (parked), resumed in 30 once lughac allowed a field and a function of one name. *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: short static functions on each style group (`Font.bold()`, `Spacing.padding(16)`, `Shadow.md()`), each returning an ordinary style value.

## OBJECTIVE

Full values (`Font { weight: Weight.Bold, ..Font.theme() }`) stay; the common cases get a few characters. Each utility is a static on the group it sets *(you)*, named with words that match the fields *(you)*, numbers in plain pixels. They mix freely with full values; the last value that sets a field wins, as always.

```
text("Storage").style([Font.h2(), Font.bold()])
text(m.folder).style([Font.mono(), Font.tabular(), Text.dim()])
text(m.total).style([Font.size(22), Text.color(hex("#e0e0e0"))])
card.style([
    Spacing.padding(16),
    Border.rounded(12),
    Border.all(1, Paint.Faded(0.15)),
    Shadow.md(),
    Background.color(hex("#ffffff")),
])
button("Save", Msg.Save).style([
    Spacing.padding_x(16),
    Spacing.padding_y(8),
    When(State.Hover, [Background.color(hex("#3a62a8"))]),
])
column(rows).style([Sizing.max_width(560), Spacing.gap(8)])
```

## THE UTILITIES *(proposed list and values)*

**Font**: `h1()` (black, 2.0×), `h2()` (black, 1.5×), `h3()` (bold, 1.5×), `h4()` (bold, 1.3×), `heading()` (bold, 1.1×), `body()` (normal, 1.1×), `caption()` (normal, 0.9×) — the theme's own text styles; `xs()` 0.75×, `sm()` 0.875×, `base()` 1.0×, `lg()` 1.125×, `xl()` 1.25×, `xxl()` 1.5×; `size(pt)`, `scale(x)`; `thin()` `light()` `normal()` `medium()` `semibold()` `bold()` `black()`; `italic()`; `sans()` `serif()` `mono()` `family(name, fallback)`; `tabular()`.

**Text**: `upper()` `lower()` `capitalize()`; `underline()` `strike()`; `tight()` (1.2) `loose()` (1.6) `line_height(x)`; `wide()` (+1 px) `narrow()` (−1 px) `letter_spacing(px)`; `dim()` (faded 0.55) `faded(x)`; `color(c)`.

**Background**: `color(c)`, `tint(x)` (the theme's text colour at `x`, following light and dark), `linear(angle, stops)`, `radial(stops)`, `image(img)` (cover).

**Border**: `all(px, paint)`, `top(px, paint)` `right(…)` `bottom(…)` `left(…)` (solid), `rounded(px)`, `corners(tl, tr, br, bl)`, `none()` (every side `Line.None`).

**Outline**: `ring(px, paint)` (solid), `offset(px)`.

**Spacing**: `padding(px)`, `padding_x(px)`, `padding_y(px)`, `padding_top` / `_right` / `_bottom` / `_left(px)`; the same six for `margin` (negative allowed); `gap(px)`.

**Sizing**: `width(px)`, `min_width(px)`, `max_width(px)`, `height(px)`, `min_height(px)`, `max_height(px)`.

**Shadow**: `sm()`, `md()`, `lg()`, `xl()` (Tailwind's own layered black shadows); `off()` (exists).

**Effects**: `opacity(x)`, `blur(px)`, `grayscale()`, `rotate(deg)`, `scale(x)`, `translate(x, y)`.

Each is one line of code and a one-line doc comment, and returns exactly what a full value written out would.

## FILES *(you: folders)*

Utilities must sit in the file of their type, and `src/styles.la` (about 435 lines) can't hold them. The groups move into a `src/styles/` folder (Lugha maps `src/styles/fonts.la` to `kivuli.styles.fonts`; a file and a folder may share a name, checked in session 30). Each group file imports `styles` for `Sheet` while `styles` imports them for `Sheet`'s fields, so DECISION-025's cycle widens to **`styles` ↔ the files in `styles/`**:
- `src/styles/fonts.la` (new): `Font`, `Text`, their utilities.
- `src/styles/boxes.la` (new): `Background`, `Border`, `Outline`, `Spacing`, `Sizing`, their utilities.
- `src/styles/effects.la` (today's `src/style_groups.la`, moved): `Shadow`, `Effects`, their utilities.
- `src/styles.la` keeps `Styling`, `Sheet`, `applied`, `State` and `When` (about 120 lines).

`style_values.la`, `sheets.la` and `box_rules.la` stay where they are (not this PRP). The move is its own commit, no behaviour change, before the utilities. The entry re-exports as now; apps see no difference.

## TESTS (narrow)
- [ ] After the move: the style tests pass unchanged (styles, states, spacing, sizing, shadows, sheets, sheets_border, theming).
- [ ] `test/utilities.la` (new): every utility equals the full value it stands for (one assertion each); utilities and full values merge as values do; the OBJECTIVE reads as written.
- [ ] Every example builds; `examples/hello.la` and `examples/borders.la` use a few utilities where they shorten the code.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README (a utilities table), DESIGN, CHANGELOG 0.20.0, CLAUDE.md (module tree with the `styles/` folder, rule 2's cycle), DECISIONS (025 widened); merged, tagged; pushed only on your go.
