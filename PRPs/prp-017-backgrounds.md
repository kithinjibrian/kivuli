# PRP-017 — Backgrounds through `.style`: colours, gradients, images

Status: built (session 19) on branch prp-017-backgrounds; merged into main
Discovery: session 19, 2026-10-19 (DECISION-023's next group). *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: a `Background` value in `.style([...])`: a colour, a gradient and an image, stacked as CSS stacks them, merged like every other style value.

## OBJECTIVE

0.11.0 removed named looks, and with them every way to give a node a background. This brings backgrounds back through the one door, as typed values: a colour (`Paint`, so a tint can follow the theme), a linear or radial gradient, and a picture from a theme icon, a file or bytes, filling the node one of four ways. *(you)*

This is the target. Each example below is a test (see TESTS TO WRITE) and must read exactly like this when the PRP is done:

```
import kivuli { View, Background, Gradient, Stop, Fill, Image, Paint, Text, column, hex, text };

/** A card that works in light and dark: a tint of the theme's own text colour. */
fun card(): Background = Background { color: Paint.Faded(0.08), ..Background.theme() };

fun view(m: Model): View<Msg> = column([
    column(rows).style([card()]).padding(16),
    text("NEW").style([
        Background { color: Paint.Color(hex("#c0392b")), ..Background.theme() },
        Text { color: Paint.Color(hex("#ffffff")), ..Text.theme() },
    ]),
    column([]).height(120).style([Background {
        gradient: Gradient.Linear(180, [
            Stop { color: Paint.Color(hex("#1e3c72")), at: 0.0 },
            Stop { color: Paint.Color(hex("#2a5298")), at: 1.0 },
        ]),
        ..Background.theme()
    }]),
    column([]).height(160).style([Background {
        color: Paint.Color(hex("#101010")),
        gradient: Gradient.Linear(180, [
            Stop { color: Paint.Color(hex("#00000000")), at: 0.0 },
            Stop { color: Paint.Color(hex("#000000aa")), at: 1.0 },
        ]),
        image: Some(Image.File("examples/assets/sample.png")),
        fill: Fill.Cover,
    }]),
    column([]).height(80).style([Background { image: Some(Image.Icon("starred-symbolic")), fill: Fill.Tile, ..Background.theme() }]),
])
```

Merging is the rule every value follows: for each field, the last value that sets it wins; `Theme` (and `None` for `image`) sets nothing. *(you)*

```
.style([card(), Background { gradient: g, ..Background.theme() }])   // card's colour, under g
```

## CONTEXT

- Starting state: kivuli 0.11.0. `src/typography.la` (`Styling`, `Sheet` with `font` and `text`, `Font`, `Text`); `src/sheets.la` (checks, atomic CSS rules); `src/paint.la` (classes, `stylesheet(view)`); `live.add_rules` (one growing app stylesheet); `src/pictures.la` (`Image`, `identity`, PNG/JPEG header checks: `size_of`); `src/textures.la` (reads files: `read_bytes`).
- Ending state: `Background`, `Gradient`, `Stop`, `Fill` beside `Font` and `Text`; `Sheet.background`; their checks and CSS; pictures embedded as checked `data:` URLs; `examples/backgrounds.la` (new); tests; kivuli 0.12.0.
- GTK 4.22 CSS used (checked in libgtk-4 in session 19): `background-color`, `background-image` with `linear-gradient`, `radial-gradient`, `url("data:…")` and `-gtk-icontheme("name")`, `background-size`, `background-position`, `background-repeat`.
- No new externs. No new libraries: base64 is written in Lugha (std has none).
- Open decisions: none (this continues DECISION-023).

## IMPLEMENTATION REQUIREMENTS

### The value *(you)*
```
public struct Background {
    color: Paint,          // Theme | Color(c) | Faded(x)
    gradient: Gradient,
    image: Option<Image>,  // None: not set here
    fill: Fill,
}
public enum Gradient {
    Theme,
    Linear(i64, Stop[]),   // degrees: 0 points up, 90 right, 180 down
    Radial(Stop[]),        // a circle from the node's centre out
}
public struct Stop { color: Paint, at: f64 }   // at: 0.0 to 1.0
public enum Fill { Theme, Cover, Contain, Tile, Stretch }
Background.theme()        // every field the theme's
```
- `Background` implements `Styling`; `Sheet` gains `background`.
- **Layers** stack as CSS stacks them: the image over the gradient over the colour. *(you)*
- **`Paint` on a background**: `Theme` is none of its own; `Color(c)` is that colour, and themed widgets' own background image is cleared so it shows; `Faded(x)` is the theme's text colour at that opacity (`alpha(currentColor, x)`), a tint that follows light and dark. *(you)*
- **Fill** *(you)*: `Cover` fills the node, cropping (the default with an image when `fill` is `Theme`); `Contain` shows the whole picture; `Tile` repeats it at its own size; `Stretch` fits it exactly. Images are always centred.

### Pictures *(you)*
- From a file or bytes, checked with `pictures.size_of` (PNG/JPEG, 4096 × 4096 px, 16 MB) before GTK sees them, then embedded in the stylesheet as a `data:image/png;base64,…` (or `image/jpeg`) URL, so GTK never decodes unchecked bytes or opens a path from CSS.
- One that can't be shown (missing, corrupt, not PNG/JPEG, too big) shows no image; the colour and gradient still show; the app keeps running.
- A theme icon is `-gtk-icontheme("name")`.
- An image's class is named by its identity (`pictures.identity`, in hex): the same key or path is encoded once; a file changed on disk under the same path isn't noticed, as for `image(…)`.
- Reading a file is I/O, so `paint` stays pure: `sheets` gives an image's class and the image; `live` (which already reads files through `textures`) turns it into a rule when the class is first seen. *(proposed)*

### Checks (stop the app with a message, as for fonts) *(proposed wording)*
- `Gradient: at least two stops`; `Stop.at(1.5): a stop is between 0 and 1`; `Gradient: stops must be in order`; `Paint.Faded(…)` in a stop or the colour, as now; `Image.Icon("…"): an icon name can't hold quotes, backslashes, ; { } or line breaks`.

### Module *(proposed)*
- `src/typography.la` holds `Styling`, `Sheet` and every style value, so with `Background` it isn't typography any more: renamed `src/styles.la` (own commit, no behaviour change), and `test/typography.la` → `test/styles.la`. Base64 and `data:` URLs go in `src/pictures.la` (pure).

### Showcase *(proposed)*
- `examples/backgrounds.la` (new): a specimen of every value: colours, a faded tint, linear and radial gradients, each fill with the sample picture, a theme icon tiled, and the three layers together.

### Must NOT Do
- No conic or repeating gradients, no image positions other than centred, no several images on one node, no borders or corners (later). *(you)*
- No path in the stylesheet: files are read and embedded by kivuli.

## ERROR HANDLING REQUIREMENTS

- Bad values are programmer errors: `paint` returns `Err(message)` and the app stops (`kivuli: …`).
- A picture that can't be shown is data, not a programmer error: no image, no message, as for `image(…)`.

## SECURITY CONSIDERATIONS

- Every file or bytes picture is header-checked before GTK decodes it, and reaches GTK only as bytes in a `data:` URL: no paths in CSS.
- An icon name goes inside a CSS string: the same character guard as font names. Tested.
- Base64 is pure Lugha; tested against RFC 4648's vectors.

## TESTS TO WRITE

Pure:
- [x] `test/styles.la` (renamed): `Background.theme()` sets nothing; the merge rule across backgrounds (colour kept under a later gradient; a later image replaces an earlier one; `None` skipped); a background between fonts and text; the OBJECTIVE's `view` reads as written.
- [x] `test/sheets.la`: CSS for a colour (`background-color: #…; background-image: none`), a faded tint, a linear and a radial gradient (stops, angle, faded stops), an icon, each fill (`background-size`/`-repeat`/`-position`), the three layers in one `background-image` with the colour beneath; each check message; icon-name injection cases.
- [x] `test/pictures.la`: base64 of RFC 4648's vectors ("", "f", "fo", "foo", "foob", "fooba", "foobar"); a `data:` URL for PNG and JPEG bytes; none for bytes that fail the header check.
- [x] `test/theming.la`: `paint` gives a background's classes and refuses its bad values.

Live GTK (skipped without a display), `test/live_type.la`:
- [x] a node with a background colour, a gradient and an image (from bytes and from a file) carries its classes, and the app's stylesheet holds their rules, the image's as a `data:` URL; a missing file gives no image rule and no crash; a picture never seen before joins while the app runs.
- [ ] How backgrounds look: checked by hand in `examples/backgrounds.la`.

## ROLLBACK PLAN
- Branch `prp-017-backgrounds` from `main`; abandon by leaving it unmerged.

## ACCEPTANCE CRITERIA
- [x] The OBJECTIVE's code compiles and its tests pass unchanged.
- [x] Pictures reach GTK only checked, as bytes; bad ones show nothing.
- [x] Tests written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG, CLAUDE.md, DECISIONS updated; 0.12.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built with `-o examples/<name>`.
- For the user: `lughac run examples/backgrounds.la`: each value as its label names it, in light and dark.

## NOTES FROM THE BUILD

- **`background-image` is one property**, so the gradient and the image share one class (`kv-bgi-g…` for a gradient alone, `kv-bgp-…` for a picture with the gradient under it), and `kv-bgi-none` only for a colour alone: two classes never set `background-image` differently.
- **A picture's rule is built by the runtime**: `paint` (pure) names its class from the image's identity, gradient and fill; `live` reads a file, checks the bytes and builds the rule (`sheets.url_of`, `picture_css`) when the class is first seen. The pure pieces are tested with a fake file reader.
- **Every generated rule kind was run through GTK 4.22's CSS parser** (a C probe with `parsing-error`): no errors, gradients with `#rrggbbaa` and `alpha(currentColor, …)` stops, `-gtk-icontheme`, `data:` URLs, `background-size: 100% 100%`.
- **A style compares with `==` in the diff**: a background picture from bytes compares its bytes on every update. Fine at the sizes allowed; worth a key-only identity if it shows up in a profile.
- **`pictures` couldn't name both a module and a function** (E0314): `paint.background_pictures`.
