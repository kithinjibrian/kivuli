# PRP-053 — SVG images

Status: built (session 118); amended by DECISION-030; merged into main in session 118 (its own tests run and pass)
Discovery: session 116, 2026-10-20 (GTK 4.22's own SVG renderer, `GtkSvg` in `gtk/gtksvg.h`: a `GdkPaintable`, so pictures and button images show it sharp at any size; its `GtkSvgFeatures` turn off external resources, animations and GTK's extensions). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Image.File` and `Image.Bytes` take SVG beside PNG and JPEG, everywhere an image is shown but backgrounds.

## OBJECTIVE

```
image(Image.File("assets/logo.svg")).size(64)
image(Image.Bytes("logo", logo_svg_bytes))
button("", Msg.Save).image(Image.File("assets/save.svg"))
ui.avatar(Some(Image.File("me.svg")), "BK")
```

*(you: A)*

## REQUIREMENTS

### Telling an SVG apart (pure, `view/pictures.la`)
- An SVG is bytes that start, after any UTF-8 byte-order mark and whitespace, with `<svg` or with `<?xml` and hold `<svg` within their first 1 KB *(proposed)*. The 16 MB cap holds; there's no header size to read before drawing, so the side limit is checked on what GTK reads (below).
- `size_of` keeps meaning PNG and JPEG (backgrounds use it); a new `kind_of(bytes)` says PNG, JPEG, SVG or why not.

### Drawing it (`render/textures.la`, GTK)
- An SVG becomes a `GtkSvg`: `gtk_svg_new()`, then `gtk_svg_set_features(GTK_SVG_SYSTEM_RESOURCES)` (external resources, animations and extensions off: *you: A*), then `gtk_svg_load_from_bytes`: features set before the bytes are read, so nothing outside the SVG is ever opened.
- **No `<image>` (DECISION-030, you: A)**: GTK 4.22.4 segfaults on a non-`data:` `<image>` with external resources off, so an SVG holding `<image` anywhere (a namespace prefix too, even in a comment) is refused before GTK sees it: it shows nothing. Pure, in `kind_of`.
- **Shown only if it loads and has a size**: an SVG GTK reports a syntax error for shows nothing (GTK would otherwise draw its 200 × 200 default); other reported errors (an unknown element) don't stop it. Its intrinsic width and height (`gdk_paintable_get_intrinsic_width`/`height`) must be at most 4096 px each and above 0. A `viewBox`-only SVG takes the viewBox's size (probed). Otherwise it shows nothing, kivuli's rule for every picture, so an avatar's initials show through.
- Pictures (`gtk_picture_set_paintable`), button images (`gtk_image_set_from_paintable`) and avatars take it; `.fit` and `.size` work as on a texture. A texture and an SVG are both paintables; kivuli keeps no reference after handing one over.
- **lugha-gtk 0.1.43**: `GtkSvg`; a C helper `lgtk_svg_load(bytes)` (`src/c/svg.c`: a new GtkSvg with its features set to system resources only before the bytes are read, its `error` signal counting syntax errors: `NULL` on one; the signal takes a `GError`, which Lugha can't name in a handler), `gtk_svg_get_features`, `gtk_svg_unref`, `gdk_paintable_get_intrinsic_width`/`height` for it, and `gtk_picture_set_paintable`/`gtk_image_set_from_paintable` taking a `GtkSvg` (the same C functions, typed for it), checked against `gtk/gtksvg.h`, `gdk/gdkpaintable.h`, `gtk/gtkpicture.h`, `gtk/gtkimage.h`. Pushed under the standing go. No new `[link]` library: GtkSvg is GTK's.

### Backgrounds
- `Background.image(…)` takes no SVG (*you: A*): it goes to GTK's CSS as a `data:` URL, where the features can't be set. `Image.Bytes` holding an SVG is refused by the sheet's check (`an SVG can't be a background`); an `Image.File` that turns out to be an SVG shows nothing, as an unreadable file does (its bytes are read only when shown).

### Must NOT Do
- No animated SVGs, no SVG states or symbolic recolouring, no SVG backgrounds, no `.svg` icons by theme name (theme icons are unchanged).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] `test/pictures.la`: `kind_of` for an SVG with and without the XML prolog, with a byte-order mark and leading space, `<?xml` without `<svg`, text that isn't an image, over 16 MB, an SVG holding `<image` or `<s:image` refused; `size_of` and `data_url` still refuse an SVG.
- [x] `test/sheets.la` (or `checks`): a background of SVG bytes is refused with its reason.
- [x] `test/live_pictures.la`: an SVG file and SVG bytes shown in a picture (a paintable of their size); in a button image; its features are system resources only; an SVG pointing at an external image doesn't load it; a broken SVG, one with no size and one claiming 10000 px show nothing.
- [x] lugha-gtk: an SVG loaded with features set first; its intrinsic size; features read back.
- [x] A C probe first (session 117–118; found the crash, DECISION-030): GtkSvg's intrinsic size for width/height, `viewBox` only and broken input, and that external `<image>` isn't fetched with the feature off.
- [x] `examples/controls.la` (pictures) gains an SVG (`examples/assets/sample.svg`, written for it); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.48.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **The probe found GTK 4.22.4's crash** (DECISION-030): `<image>` with external resources off segfaults; the user chose A, so kivuli refuses any SVG holding `<image` first. GTK doesn't expand `<!ENTITY>`s (a syntax error) and skips a prefixed `<s:image>`, so neither gets one past.
- **A broken SVG gets GTK's 200 × 200 default size**, so a size check alone wouldn't catch it: `lgtk_svg_load` (lugha-gtk 0.1.43, `src/c/svg.c`) counts syntax errors through the `error` signal and returns `None` on one. A `viewBox`-only SVG takes the viewBox's size.
- **The tests are their own files**: `test/svgs.la` (pure: 4) and `test/live_svgs.la` (GTK: 1), not pictures.la and live_pictures.la; live_pictures' `shown` reads a picture's paintable as a texture, which an SVG isn't.
- **`texture_of` is gone**: `drawn_of` gives a texture or an SVG.
- **Tests run and pass**: svgs, pictures, live_svgs, live_pictures, live_images (17); the controls example's own (3). Every test and example type-checks; controls built (an SVG at 32 and 96 px).
- **Confirmed by hand** (session 120): the sample SVG is sharp at 32 and 96 px in `examples/controls.la`.
