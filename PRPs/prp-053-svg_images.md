# PRP-053 — SVG images

Status: draft (session 116)
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
- **Shown only if it has a size**: its intrinsic width and height (`gdk_paintable_get_intrinsic_width`/`height`) must be above 0 and at most 4096 px each; otherwise (a broken SVG, one with no size, one claiming a huge one) it shows nothing, kivuli's rule for every picture, so an avatar's initials show through. *(proposed: an SVG with only a `viewBox` and no width or height, which GTK sizes as 0, shows nothing; `.size(n)` doesn't rescue it)* — verified by a probe first; if GTK gives a `viewBox`-only SVG a size, that's used.
- Pictures (`gtk_picture_set_paintable`), button images (`gtk_image_set_from_paintable`) and avatars take it; `.fit` and `.size` work as on a texture. A texture and an SVG are both paintables; kivuli keeps no reference after handing one over.
- **lugha-gtk 0.1.43**: `GtkSvg`, `gtk_svg_new`, `gtk_svg_set_features`/`get_features`, `gtk_svg_load_from_bytes`, `gtk_svg_unref`, `gdk_paintable_get_intrinsic_width`/`height` for it, and `gtk_picture_set_paintable`/`gtk_image_set_from_paintable` taking a `GtkSvg` (the same C functions, typed for it), checked against `gtk/gtksvg.h`, `gdk/gdkpaintable.h`, `gtk/gtkpicture.h`, `gtk/gtkimage.h`. Pushed under the standing go. No new `[link]` library: GtkSvg is GTK's.

### Backgrounds
- `Background.image(…)` takes no SVG (*you: A*): it goes to GTK's CSS as a `data:` URL, where the features can't be set. `Image.Bytes` holding an SVG is refused by the sheet's check (`an SVG can't be a background`); an `Image.File` that turns out to be an SVG shows nothing, as an unreadable file does (its bytes are read only when shown).

### Must NOT Do
- No animated SVGs, no SVG states or symbolic recolouring, no SVG backgrounds, no `.svg` icons by theme name (theme icons are unchanged).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/pictures.la`: `kind_of` for an SVG with and without the XML prolog, with a byte-order mark and leading space, `<?xml` without `<svg`, text that isn't an image, over 16 MB; `size_of` and `data_url` still refuse an SVG.
- [ ] `test/sheets.la` (or `checks`): a background of SVG bytes is refused with its reason.
- [ ] `test/live_pictures.la`: an SVG file and SVG bytes shown in a picture (a paintable of their size); in a button image; its features are system resources only; an SVG pointing at an external image doesn't load it; a broken SVG, one with no size and one claiming 10000 px show nothing.
- [ ] lugha-gtk: an SVG loaded with features set first; its intrinsic size; features read back.
- [ ] A C probe first: GtkSvg's intrinsic size for width/height, `viewBox` only and broken input, and that external `<image>` isn't fetched with the feature off.
- [ ] `examples/controls.la` (pictures) gains an SVG (`examples/assets/sample.svg`, written for it); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.48.0, CLAUDE.md; merged, tagged; pushed only on your go.
