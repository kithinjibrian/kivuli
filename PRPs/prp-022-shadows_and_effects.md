# PRP-022 — Shadows and effects through `.style`

Status: draft, for the user's approval (session 25)
Discovery: session 25, 2026-10-20. *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Shadow` (box and text shadows, each a list of `Drop`s) and `Effects` (opacity, filters, transforms) in `.style([...])`.

## OBJECTIVE

Cards that lift off the page, text that reads over a picture, a faded or greyed-out node, a tile that grows on hover: through the one door, as CSS's `box-shadow`, `text-shadow`, `opacity`, `filter` and `transform` (all parsed by GTK 4.22, session 23's probe). *(you)*

The target (a test, see TESTS):

```
import kivuli { View, Shadow, Drop, Effects, Filter, Move, When, State, Paint, column, hex, text, button };

fun lifted(): Shadow = Shadow { boxes: [
    Drop { y: 8, blur: 24, color: Paint.Color(hex("#00000026")), ..Drop.theme() },
    Drop { y: 1, blur: 2, color: Paint.Color(hex("#00000033")), ..Drop.theme() },
], ..Shadow.theme() };

fun view(m: Model): View<Msg> = column([
    column(rows).style([lifted()]),
    text("Over a picture").style([Shadow { text: [Drop { y: 1, blur: 2, color: Paint.Color(hex("#000000aa")), ..Drop.theme() }], ..Shadow.theme() }]),
    column(rows).style([Effects { opacity: Some(0.5), ..Effects.theme() }]),
    text("avatar").style([Effects { filters: [Filter.Grayscale(1.0), Filter.Brightness(1.1)], ..Effects.theme() }]),
    button("Tile", Msg.Open).style([
        lifted(),
        When(State.Hover, [Effects { transform: [Move.Scale(1.05)], ..Effects.theme() }]),
    ]),
    button("Flat", Msg.Open).style([Shadow.off()]),
])
```

## REQUIREMENTS

### First: room for more style groups *(proposed; needs your approval: an architecture rule)*
- `src/styles.la` is 431 lines and every group must sit with `Sheet` (each group's `apply` takes a `Sheet`, and `Sheet` holds every group). New groups go in a new `src/style_groups.la`, which imports `styles` for `Sheet` while `styles` imports it for `Sheet`'s fields: **a second allowed import cycle**, like `views` ↔ `mapping` (DECISION-018), recorded as DECISION-025 and in CLAUDE.md's rule 2. Plain values (`Drop`, `Filter`, `Move`) go in `style_values.la` as before.

### The values *(you)*
```
public struct Drop { x: i64, y: i64, blur: i64, spread: i64, color: Paint, inset: bool }   // pixels; spread and inset: boxes only
public struct Shadow { boxes: Drop[], text: Drop[] }      // [] is the theme's
public struct Effects { opacity: Option<f64>, filters: Filter[], transform: Move[] }
public enum Filter { Blur(i64), Brightness(f64), Contrast(f64), Grayscale(f64), HueRotate(i64), Invert(f64), Saturate(f64), Sepia(f64) }
public enum Move { Translate(i64, i64), Rotate(i64), Scale(f64), ScaleXY(f64, f64) }
Drop.theme(), Shadow.theme(), Effects.theme(), Shadow.off()
```
- A list is one field: a later non-empty list replaces the earlier; `boxes` and `text` merge apart; `opacity` replaces when set.
- `Drop.color` `Theme` is `currentColor`.
- **`Shadow.off()`** removes the theme's own box shadow (a button's): its `boxes` is one transparent `Drop`, written `box-shadow: 0 0 0 0 transparent`. *(proposed: `[]` must mean "the theme's" for merging to work)*
- **A transform changes only how the node is drawn**, never the room it takes: neighbours don't move.
- All of it works in `When(…)` (each is a plain CSS rule).

### CSS *(proposed)*
- One class per value list: `kv-bsh-{list in hex}` → `box-shadow: x y blur spread color[ inset], …`; `kv-tsh-…` → `text-shadow: x y blur color, …`; `kv-op-{x·1000}` → `opacity`; `kv-flt-…` → `filter: blur(6px) grayscale(1.0) …`; `kv-tf-…` → `transform: translate(4px, 2px) rotate(-8deg) scale(1.05) scale(1.1, 0.9) …`. These go in `box_rules.la` (196 lines).

### Checks (stop the app with a message) *(proposed wording)*
- `Effects.opacity: 1.5: an opacity is between 0 and 1`; `Filter.Blur(-2): a blur can't be negative`; `Filter.Grayscale(2.0): Grayscale, Invert and Sepia are between 0 and 1`; `Filter.Brightness(-1.0): Brightness, Contrast and Saturate can't be negative`; `Drop: a shadow's blur can't be negative`; `Shadow.text: a text shadow has no spread or inset`.

### Must NOT Do
- No transitions or animations (next), no `backdrop-filter` (GTK has none), no 3D transforms, no transform origin other than the centre.

## TESTS (narrow, per the user: only what this changes)
- [ ] Before the module move: the style tests pass unchanged after it (styles, states, spacing, sizing).
- [ ] `test/shadows.la` (new, pure): the theme values; merging (a later list replaces; boxes and text apart); the CSS for each value (several drops, inset, faded colours, every filter, every move, `Shadow.off()`); paint's classes; each check message; a shadow and an effect inside `When`; the OBJECTIVE reads as written.
- [ ] `test/live_type.la`: a shadow's and an effect's classes land and their rules join the stylesheet; GTK parses each generated kind (C probe).
- [ ] Every example builds; no full suite (the user's call). How it looks: by hand in `examples/borders.la` ("Shadows and effects").

## ACCEPTANCE
- [ ] The OBJECTIVE compiles and its test passes; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.17.0, CLAUDE.md, DECISIONS; merged, tagged; pushed only on your go.
