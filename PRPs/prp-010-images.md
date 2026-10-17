# PRP-010 — Images from any source: theme icons, files and bytes

Status: built (session 11) on branch prp-010-images; merged into main
Discovery: session 11, 2026-10-18. The brief asked for images from bytes; talked through first, the user wanted one image model that grows to web images. *(you)*: chosen in the talk and the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: One image type, `Image.Icon(name) | Image.File(path) | Image.Bytes(key, bytes)`, shown by `image(…)` and put on buttons by `.image(…)`; PNG and JPEG decoded safely from bytes, every source size-checked before GTK decodes.

## OBJECTIVE

Rapidkoi holds app icons and banners as PNG bytes from a bundle and must never write them to disk (its PRP-003, DECISION-020 item 1). Web images come later the same way: **an image shows bytes it already has; the app fetches over the network with a non-blocking command** (a later PRP, DECISION-007), never kivuli behind a view's back. *(you: model B)*

```
image(Image.Bytes("counter:banner", banner)).width(320).height(120).fit(Fit.Cover)
image(Image.File("assets/logo.png")).size(64)
icon("list-add-symbolic")                                     // shorthand: image(Image.Icon(…))
button("", Msg.Tab(id)).image(Image.Bytes("tab:{id}", icon)).tooltip(name)
button("Counter", Msg.Open).image(Image.Bytes("tile:counter", icon)).size(48)
```

## DECISIONS ALREADY MADE (session 11)

- **Model B for the web** (above). *(you)*
- **DECISION-018 resolved: move `View.map`'s body out** to a new `src/mapping.la`; `views` and `mapping` import each other, the one written exception to CLAUDE.md's "imports run one way". `views.la` drops from 499 to about 440 lines, done first as its own commit. *(you)*
- **`image(Image.…)` is the one standalone widget**; `image(path)` becomes `image(Image.File(path))` (breaking); `icon(name)` stays as a shorthand. *(you)*
- **PNG and JPEG, header-checked before GTK decodes, for every source, files included**; other formats show nothing (SVG files, which `image(path)` showed before, no longer do). *(you)*
- **On a button, an image is the theme's icon size; `.size(n)` sets it in pixels.** *(you)*

## CONTEXT

- Starting state: kivuli 0.4.1. `View.Icon(string)`, `View.Image(string)` (a path, GtkPicture), `.icon(name)` a `Style` field shown in kivuli's button content (a GtkImage, PRP-005); `src/views.la` 499 lines; lugha-gtk 0.1.24.
- Ending state: `View.Image(Image)` replaces `View.Icon` and `View.Image(path)`; `View.Button(label, Option<Msg>, Option<Image>)` holds the button's image (the `Style.icon` field goes); new `src/mapping.la` and pure `src/pictures.la` (reading headers, the identity of a source); tests in `test/pictures.la` (new), `test/lib.la`, `test/diff.la` or a new diff file (it's at the limit), `test/testing.la`, `test/map.la`, `test/live_images.la`, `test/live_health.la`; lugha-gtk 0.1.25; kivuli 0.5.0.
- GTK and GLib functions needed, each to be checked against `/usr/include/gtk-4.0` and the GLib headers before it's declared, new in lugha-gtk 0.1.25: `GBytes`, `GdkTexture`, `GError` handle types; `g_bytes_new` (copies the data, so C never holds Lugha memory), `g_bytes_unref`; `gdk_texture_new_from_bytes` (GTK 4.6, a `GError **` as an `out` parameter), `gdk_texture_get_width` / `get_height`, `g_error_free`, the texture's `g_object_unref`; `gtk_picture_set_paintable` / `get_paintable`, `gtk_image_set_from_paintable` / `get_paintable` (a texture is a `GdkPaintable`: the same pointer, as a string list is passed where GTK takes a `GListModel`); a `g_object_weak_ref` for textures (tests count their finalizations).
- Open decisions that must be resolved first: none (DECISION-018 resolved above).

## IMPLEMENTATION REQUIREMENTS

### The image type
- **`public enum Image { Icon(string), File(string), Bytes(string, u8[]) }`**: a theme icon's name, a local file's path, or a key and the bytes. *(you)*
- **Identity, not contents**: two images are the same when they're the same kind with the same name, path or key; the bytes are never compared. A new key (or path, or name) replaces the image; the same key never re-decodes. A file that changes on disk under the same path isn't noticed (give it a new path or use bytes). *(you; proposed: the file rule)*

### Decoding, safely (pure checks first)
- Before GTK decodes anything, kivuli reads the image's size from its header in pure Lugha: PNG (the 8-byte signature, then width and height from IHDR) or JPEG (the SOI marker, then the first start-of-frame marker's height and width, scanning a bounded number of segments). Anything else, a truncated header, or a size over **4096 × 4096 px** shows nothing. So a small file claiming a huge size never reaches the decoder. *(you; proposed: the limits)*
- Input over **16 MB** shows nothing. A file is read with `read_bytes` (std.fs has no file size, so a large local file is read before it's refused: recorded for lughac, see the notes); a missing file shows nothing. *(proposed)*
- Then `gdk_texture_new_from_bytes`; a decode error (`GError`) is freed and shows nothing. Never a panic; empty bytes are just "nothing". *(you)*
- "Nothing": the widget stays (its `.width`/`.height` kept), with no picture; on a button, the image is hidden and the label shows alone. An unknown theme icon name still shows GTK's placeholder, as today. *(proposed)*

### Showing
- **`image(Image.Icon(name))`** is a `GtkImage` as `icon(name)` is today (theme size; `.size(n)` its pixel size).
- **`image(Image.File(…))` and `image(Image.Bytes(…))`** are a `GtkPicture` showing the texture, with `.fit(…)` as today; `.size(n)` gives it an n × n box (as `.width(n).height(n)`). *(proposed: `.size` on a picture)*
- **`.image(src)` on a button** puts the image where `.icon(name)` goes (before the label; alone when the label is `""`), in kivuli's button content (PRP-005's GtkImage, now also from a texture); `.size(n)` on the button sets the image's pixel size; `.icon(name)` is `.image(Image.Icon(name))`. Ignored on other nodes. *(you)*
- Memory: kivuli drops its own references to the GBytes and the texture as soon as the widget holds the texture, so they're freed with the widget, or when a new key replaces the texture.

### The view and the diff
- `View.Image(Image)` replaces `View.Icon` and `View.Image(path)`; `View.Button(string, Option<Msg>, Option<Image>)`; `Style.icon` goes. *(proposed)*
- Diff: an image whose identity changes gives `SetImage(path, image)`; the same identity gives nothing; an icon that becomes a picture (or back) is a `Replace` (a different widget). A button whose image identity changes gives `SetButtonImage(path, Option<Image>)`. *(proposed: patch shapes)*
- `View.map` keeps images unchanged (in `mapping.la`).
- `kivuli.testing`: a button with an image is found by its tooltip (or label), as for icons. *(you)*

### Renames, docs, example
- `image(path)` → `image(Image.File(path))` in `examples/gallery.la` and the README; CHANGELOG with migration.
- Docs: README (`Image`, `image`, `.image`, `.size` on buttons and pictures), docs/DESIGN.md (images: sources, the checks, model B). *(you)*
- Example: `examples/gallery.la` shows the sample picture from a file and from bytes (read at start, as an app holding bundle bytes would), and a button with a bytes image. *(proposed)*
- Release: 0.5.0 (breaking: `image(path)`); tag once every check passes; push on your go. *(you)*

### Must NOT Do
- No images from URLs or the network; no fetching of any kind (that's the non-blocking commands PRP). *(you)*
- No formats beyond PNG and JPEG; no caching textures across different keys; no image editing. *(you)*

## ERROR HANDLING REQUIREMENTS

- Every failure to show an image (unknown format, too large, truncated, corrupt, missing file, decode error) shows nothing, never panics, and frees what GTK allocated. Pure `pictures.size_of(bytes): Result<(i64, i64), string>` says why, for tests.
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- Hostile bytes are bounded before decoding: 16 MB input, 4096 × 4096 px, PNG/JPEG headers parsed in pure Lugha with every index checked (Lugha panics on out-of-range; the parser checks lengths first so it never does).
- C never keeps Lugha memory: `g_bytes_new` copies.
- Each new extern matches its header; `GError` is freed; texture references dropped.
- No network access, no new library (GTK decodes PNG and JPEG itself).

## TESTS TO WRITE

Fixtures inline as bytes: a tiny valid PNG and JPEG; a PNG header claiming 5000 × 5000; a PNG with a valid header and corrupt data; truncated and empty bytes.

Pure:
- [x] `test/pictures.la`: PNG and JPEG sizes read; not an image, truncated, over 4096, over 16 MB (a long array), empty: each refused with its reason; identity: same key equal whatever the bytes, different keys not, kinds differ.
- [x] `test/lib.la`: `image(Image.…)`, `icon(name)` as `Image.Icon`, `.image` and `.icon` land in the button node; `.size` on a button.
- [x] diff: same key → nothing; new key → `SetImage` / `SetButtonImage`; icon ↔ picture → `Replace`; a button losing its image → `SetButtonImage(None)`.
- [x] `test/testing.la`: a button with a bytes image found by tooltip.
- [x] `test/map.la`: images pass through `map` (now in `mapping.la`).

Live GTK (skipped without a display):
- [x] a valid PNG shows (the picture's texture reads back its width and height); a valid JPEG too.
- [x] corrupt, too large and empty bytes show nothing, without a crash.
- [x] a key change replaces the texture (a new one, the old finalized); the same key keeps the same texture.
- [x] a button with a bytes image shows it (its GtkImage has the texture), `.size(48)` reads back 48.
- [x] `test/live_health.la`: removed image widgets and replaced textures are finalized.
- [x] lugha-gtk `test/textures.la`: each new extern.

## ROLLBACK PLAN
- Branch `prp-010-images` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.25 is additions only.

## ACCEPTANCE CRITERIA
- [x] Images from theme icons, files and bytes show through one type; hostile bytes never crash; nothing leaks across re-renders.
- [x] `views.la` has room again (DECISION-018).
- [x] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] fmt clean; public items documented; no file over 500 lines.
- [x] README, docs/DESIGN.md, CHANGELOG (with migration), CLAUDE.md (the import exception, new modules), DECISIONS.md updated; 0.5.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/gallery.la`: the sample picture from a file and from bytes look the same; the bytes-image button shows it.

## NOTES FROM THE BUILD

- **DECISION-018 first:** `View.map`'s body moved to `src/mapping.la` (views.la 499 → 452; 473 after this PRP's `.image`); the import exception is written into CLAUDE.md rule 2.
- **Two new modules:** `src/pictures.la` (pure: `Image`, `identity`, the PNG/JPEG header checks, `MAX_SIDE`, `MAX_BYTES`) and `src/textures.la` (GTK: a checked image → a texture; build.la was at 462 lines). CLAUDE.md rule 4 lists `textures.la` among the modules that call GTK.
- **The JPEG check** walks segments by their lengths (at most 64), skipping fill bytes and the standalone markers (TEM, RST0–7), and reads the first SOF0–15 (less DHT, JPG, DAC); the scan starting first, a length under 2 or a cut header refuse it. Proven on a real JPEG (3 × 2, width and height not swappable).
- **A genuine 4097 × 1 PNG fixture** (85 bytes) proves the check runs before decoding: GTK decodes it happily; without kivuli's check it showed. The 5000 × 5000 fixture only claims its size, so GTK's decoder refuses it anyway; it tests the pure check alone. Mutation-checked: without the check, the wide PNG shows; without dropping kivuli's texture reference, removed pictures leak every texture.
- **`.icon("")` still takes the image away**: it meant "no icon" as a style field, and a view in the tests relied on it (it showed GTK's placeholder for a moment in the build, caught by `test/live_images.la`).
- **The diff tests** went to `test/diff_images.la`; `test/diff.la` (at the limit) lost one duplicated test to make room for the rewritten PRP-005 ones.
- **The gallery** reads its sample's bytes once at start (as an app holds a bundle's) and shows them in a picture and on an image-only button; its own tests caught an ambiguous "Add" label on the way.
- **Fixtures as hex:** as byte arrays, the formatter put each byte on its own line (842 lines); `test/fixtures.la` keeps them as hex strings decoded by `from_hex` (70 lines).
- **Recorded:** DECISION-019 (model B, resolved) and DECISION-020 (lughac: `std.fs` has no file size, so a file is read before its size is refused; deferred, reported).
