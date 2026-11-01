## FEATURE: A radial gradient with its own shape and centre: an ellipse, sized and placed as fractions of the node

Status: proposed (2026-11-01), for the user's approval. Branch `prp-072-elliptical_gradients`.

## OBJECTIVE

An app can start a radial gradient anywhere on a node, a corner included, and stretch it to any oval, so a colour can wash in from the top-left and fade out before the far edges. Today `Gradient.Radial(stops)` is only a circle from the node's centre (`radial-gradient(circle, …)`).

```
import kivuli { View, Background, Ellipse, Gradient, Stop, Paint, column, hex };

/** An app's colour washing in from the top-left corner of a card's picture area. */
fun wash(): Background = Background.ellipse(
    Ellipse { x: 0.0, y: 0.0, across: 1.1, down: 1.5 },
    [
        Stop { color: Paint.Color(hex("#14b8a680")), at: 0.0 },
        Stop { color: Paint.Color(hex("#14b8a61f")), at: 0.6 },
        Stop { color: Paint.Color(hex("#14b8a600")), at: 0.9 },
    ],
);

fun view(m: Model): View<Msg> = column([icon]).height(84).style([Background.color(hex("#1b1b1b")), wash()])
```

The gradient sits where every gradient sits: over the background's colour, under its image. `Gradient.Radial(stops)` stays exactly as it is.

## CONTEXT

- **The request:** Rapidkoi's new tab (`../rapidkoi/docs/designs/newtab-target.md`, "What it needs, by repository", kivuli 1). Its small cards' picture areas and the page's corner glow are `radial-gradient(ellipse 110% 150% at 0% 0%, …)` and `radial-gradient(ellipse 60% 35% at 0% 0%, …)`. Rapidkoi's replica draws them as bundle SVGs until kivuli, then Rapidkoi's contract, can. The owner asked for this work in kivuli directly (2026-11-01), so there's no `kivuli-requests.md` section to remove.
- **Files read:** `src/styles/values.la` (`Gradient`, `Stop`), `src/styles/sheets.la:322` (`gradient_css`), `src/styles/checks.la:87` (the gradient's checks), `src/styles/boxes.la:59` (`Background.radial`), `src/styles/theming.la:235` (`permille`), `test/sheets.la:212`, `test/utilities.la:128`, `test/theming.la:163`, `examples/backgrounds.la:100`, `docs/DESIGN.md:30`, README.md:180 and :268, PRP-017 (backgrounds).
- **Files changed:** those, plus `src/lib.la` (export `Ellipse`), CHANGELOG.md and `lugha.toml` (0.67.0). File sizes are all well under 500 lines: `boxes.la` at 434 is the largest, and this adds about 8 lines to it.
- **GTK 4.22.4's CSS, checked with a probe** (`gtk_css_provider_load_from_string` with a `parsing-error` handler, 2026-11-01). There were no parsing errors, and the provider wrote each one back:
  - `radial-gradient(ellipse 95% 130% at 0% 0%, …)` comes back as `… at left top`;
  - `ellipse 60.5% 35% at 100% 50%` (one decimal place) comes back as `… at right`;
  - `ellipse 50% 35% at -20% 140%` (a centre outside the node) is kept as written;
  - `ellipse 0% 35%` is accepted, but it draws nothing useful, so kivuli refuses it (below).
- **Nothing new:** no externs, no dependencies, no lugha-gtk change.
- **Decisions:** this continues DECISION-023 (styles as typed values) and PRP-017. The only open entry is DECISION-041 (a portable renderer); it doesn't block this. If kivuli later stops drawing through CSS, an ellipse's centre, radii and stops are plain numbers any renderer can draw.

## REQUIREMENTS

### Must Do

- **`public struct Ellipse { x, y, across, down }`**, all `f32`, all fractions of the node:
  - `x` and `y` are its centre: 0.0 is the left or top edge, 1.0 the right or bottom edge;
  - `across` and `down` are its radii, against the node's width and height. 1.0 reaches the full width or height.
- **Why `f32`, when `Stop.at` is `f64`** (the user, 2026-11-01, after building): Lugha keeps an enum's payload inline, and every styled node carries its `Gradient`. With four `f64`s it grew about 24 bytes per node, and `test/gathering.la`'s message-cost test failed about 1 run in 3: collections landed in the timed messages while a 3000-node tree was alive. `main` passed 15 runs out of 15. With `f32` it grows 8 bytes, and the test passed 8 runs out of 8 (an `f64` control in the same copy failed 3 of 8). Seven digits are plenty for a shape, and literals like `1.1` work unchanged.
- **`Gradient.Ellipse(Ellipse, Stop[])`**: from the ellipse's centre (a stop at 0.0) out to its edge (1.0).
- **`Background.ellipse(e, stops)`**, beside `radial`: the full value it stands for.
- **The CSS** is `radial-gradient(ellipse {across}% {down}% at {x}% {y}%, {stops})`. Each number is a percentage with one decimal place, as stops are written, and is signed correctly: a centre at -0.25 is `-25.0%`, not `-2.-5%`. (`permille`'s truncation and `p / 10`/`p % 10` both go wrong for negatives, so the ellipse gets its own signed formatting.)
- **Checks**, as the other style errors are reported (`checks.la`, a sentence naming the value):
  - `across` and `down` must be positive: "Ellipse across {n}: a radius must be positive";
  - all four numbers must be finite;
  - the stops are checked as every gradient's are (at least two, between 0 and 1, in order).
  - A centre outside 0 to 1 is allowed, as CSS allows it: a glow from just beyond a corner.
- **The class** follows the existing rule: `kv-bgi-g{css in hex}`, so equal ellipses share one class and different ones don't.
- **Layers and merging are unchanged.** `Ellipse` is a `gradient`, so it sits under the image and over the colour. The last value that sets `gradient` wins.

### Must NOT Do

- **Change `Gradient.Radial`** or its CSS: apps and Rapidkoi's code keep working, and this is a minor version.
- **Add CSS's size keywords** (`closest-side`, `farthest-corner`…) or lengths in pixels. Fractions say everything Rapidkoi's design needs, and px would tie a gradient to one size.
- **Use start and end instead of left and right.** Gradients are physical, as `Linear`'s angles already are. A wash from the top-left stays top-left in a right-to-left language. This is documented, not hidden.
- **Add repeating gradients, conic gradients or colour interpolation.** None is asked for.

## DESIGN

- `values.la`: `Ellipse` (public fields, `/** … */` on each) and the new variant. It derives what `Stop` derives, for `assert_eq` in tests.
- `sheets.la`: `gradient_css` gains the `Ellipse` arm, plus a private `percent(x: f64): string` that rounds half away from zero and writes the sign once.
- `checks.la`: the `Ellipse` arm checks the four numbers, then `check_stops`.
- `boxes.la`: `Background.ellipse`.
- `lib.la`: `Ellipse` exported next to `Gradient` and `Stop`.
- `examples/backgrounds.la`: one more swatch, "Ellipse from the top-left", next to "Radial".
- No new `Live` variant, patch, signal or extern.

## TESTS

- [ ] `test/sheets.la`: an ellipse's rule is exactly `background-image: radial-gradient(ellipse 110.0% 150.0% at 0.0% 0.0%, … 0.0%, … 60.0%, … 90.0%);`.
- [ ] Equal ellipses share a class; a different centre or radius gives another.
- [ ] A negative centre is written `-25.0%`, and 0.0005 rounds to `0.1%`.
- [ ] An ellipse with an image sits in the image's one `background-image` rule (`kv-bgp-…`), the gradient under the picture.
- [ ] `test/utilities.la`: `Background.ellipse(e, stops)` equals the full value.
- [ ] Checks (each with its sentence): a zero radius, a negative radius, `NaN` or infinity in any field, too few stops, stops out of order. A centre at -0.2 or 1.4 passes.
- [ ] `test/theming.la`: an ellipse with too few stops is reported like `Radial([])` is today.
- [ ] Merging: an ellipse over a linear gradient wins, and a linear gradient over an ellipse wins.
- [ ] GTK, skipped without a display: a node styled with an ellipse builds, and its rule loads with no CSS parsing error.

## ACCEPTANCE

- [ ] The example in OBJECTIVE compiles and its test passes.
- [ ] `lughac test test/`, the examples' tests and `lughac fmt --check src examples test` pass; every example builds.
- [ ] README (the `Background` rows), `docs/DESIGN.md:30` and CHANGELOG.md say it; the version is 0.67.0.
- [ ] For the user, by eye: `lughac run examples/backgrounds.la`. The "Ellipse from the top-left" swatch shows colour washing in from its top-left corner and gone before the far edges, and Radial looks as it did.
