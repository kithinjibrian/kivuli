# PRP-005 — Icons and images

Status: built (session 3) on branch prp-005-icons_and_images; merged into main
Discovery: session 3, 2026-10-17, talked through in chat first. *(you)*: chosen in the interview; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: `icon(name)`, a theme icon; `.icon(name)` on buttons; `image(path)`, a picture from a file, with `.fit(Fit.…)`.

## OBJECTIVE

An app can show the theme's icons, on their own or on buttons (with or without a label), and pictures from files scaled to the room they're given, with the same rules as every kivuli widget: the view says what's shown, changes are patches, and everything visual comes from the GTK theme. Icons are the theme's only: an app that wants another set (Font Awesome, Lucide …) builds that on top, for example with `image(path)`. *(you)*

```
fun view(m: Model): View<Msg> = column([
    row([
        icon("emblem-ok-symbolic").size(48),
        image("examples/assets/sample.png").width(96).height(96).fit(Fit.Cover),
    ]),
    button("Add", Msg.Add).icon("list-add-symbolic"),
    button("", Msg.Delete(1)).icon("user-trash-symbolic").tooltip("Delete milk"),
]);

@test
fun deleting_by_tooltip() {
    assert_eq(kivuli.testing.click(view(m), "Delete milk"), Ok(Msg.Delete(1)));
}
```

## CONTEXT

- Starting state: `src/views.la` (470 lines: too close to 500 for this), `src/widgets.la`, `src/diff.la` (441), `src/live_tree.la`, `src/build.la`, `src/styling.la`, `src/patch.la`, `docs/DESIGN.md` (nothing on icons or images yet), lugha-gtk 0.1.21.
- Ending state: those changed, `src/views.la` split (see Must Do); tests in `test/lib.la`, `test/diff.la`, `test/testing.la`, `test/map.la`, a new `test/live_images.la`, `test/live.la`'s printer; lugha-gtk 0.1.22; a new `examples/gallery.la` and `examples/assets/sample.png`.
- Related existing code: check boxes (kivuli owns the label child, PRP-002: the same idea for button contents here); `.size(n)` and `style_text` (a style field read differently per node); tooltips and `kivuli.testing.click` by tooltip (PRP-002).
- Related source files: none.
- Lugha spec sections relied on: nothing new.
- GTK functions needed (checked in GTK 4.22's headers), new externs in lugha-gtk 0.1.22:
  - icons: `gtk_image_new_from_icon_name`, `gtk_image_set_from_icon_name`, `gtk_image_get_icon_name` (tests; `const char *`, `NULL` as `None`), `gtk_image_set_pixel_size` / `get_pixel_size` (`-1`: the theme's size);
  - pictures: `gtk_picture_new_for_filename`, `gtk_picture_set_filename`, `gtk_picture_set_content_fit` / `get_content_fit` (`GtkContentFit`: `FILL` 0, `CONTAIN` 1, `COVER` 2, `SCALE_DOWN` 3, values to check in `gtkenums.h`), `gtk_picture_get_file` + `g_file_get_path` (tests: the path read back; GLib allocates it, so it's freed);
  - buttons: `gtk_button_set_child` / `get_child`; `gtk_widget_set_visible` / `get_visible`.
- Open decisions that must be resolved first: none.

## IMPLEMENTATION REQUIREMENTS

### Must Do
- **`icon(name)`**: a theme icon by name (`"user-trash-symbolic"`). Symbolic icons take the theme's text colour, so they follow light and dark themes. *(you)*
- **Icon size**: the theme's size by default; `.size(n)` on an icon sets it to `n` pixels (on text it stays points). Dropping `.size` goes back to the theme's size. Icons on buttons always use the theme's size. *(you)*
- **`.icon(name)` on a button** (a style field, `""` none; ignored elsewhere): a label and an icon show both, the icon first; a label `""` with an icon is an icon-only button. Adding, changing or dropping it is a `SetStyle`. *(you)*
- **Every button's content is kivuli's own** (a box holding an icon and a label, each hidden when empty), as check boxes have their own label since PRP-002, so a label and an icon can come and go through one path. `SetLabel` sets that label. The theme's button classes are kept as GTK itself would set them: `text-button` (label only), `image-button` (icon only), `image-text-button` (both). *(proposed: for every button, not only those with icons; the class names to be confirmed against GTK's theme during the build)*
- **`image(path)`**: a picture from a file (PNG, JPEG, SVG: whatever GTK's loaders read), sized by `.width` / `.height`. A relative path is relative to the directory the app was started from (documented; packaging is a later topic). *(you)*
- **`.fit(Fit.Contain | Cover | ScaleDown | Fill)`** on an image (a style field; ignored elsewhere): `Contain` (the default) shows the whole picture, scaled to fit, keeping its shape; `Cover` fills the room and crops; `ScaleDown` is `Contain` but never enlarges; `Fill` stretches. *(you; proposed: including `Fill`, since GTK has it)*
- **Missing things**: a file that doesn't exist or can't be read, and an icon name the theme doesn't have, show GTK's "missing image" icon; the app keeps running. *(you)*
- **Diff**: an icon whose name changes gives `SetIcon(path, name)`; an image whose path changes gives `SetFile(path, file)`; otherwise nothing (no messages to re-point). An icon that becomes an image (or anything else) is a `Replace`. *(proposed: the patch shapes)*
- **`View.map`** carries icons and images unchanged (they send nothing). *(proposed)*
- **Testing**: no new helpers. Icon-only buttons are found by their tooltip, which they should have anyway (it's what screen readers and hovering users get). `texts` doesn't list icons or images. *(you)*
- **Split `src/views.la` first** (470 lines; this adds about 60): the style types (`Style`, `Window`, `ButtonStyle`, `Align`, the new `Fit`, and `plain()`) move to a new `src/looks.la` (not `style.la` or `styles.la`: `style` is a common local name, E0311), re-exported by `lib.la` as today, as its own commit with no behaviour change. The `impl View` modifiers stay in `views.la` (they must sit with `View`). *(proposed: file size rule)*
- **`docs/DESIGN.md`** gains a section: icons are the theme's (symbolic names preferred), pictures follow `.fit`, kivuli never recolours or draws either. *(proposed)*
- **Showcase**: a new `examples/gallery.la`: theme icons at the theme's size and at 32 and 64 pixels, a label button with an icon, an icon-only button with a tooltip, and one bundled picture shown four times, once per `.fit`, in equal boxes; with its own tests. `examples/assets/sample.png` is a small picture we make ourselves (generated, so it's ours, GPL-3.0 like the repo), under 20 KB. *(you; proposed: how the picture is made)*

### Must NOT Do
- No icon folders or icon sets beyond the theme (Font Awesome etc.): an app's own job. *(you)*
- No image data from the model (bytes, textures), no loading in the background, no animated images.
- No `kivuli.testing` helper by icon name. *(you)*
- No icon sizes on buttons, no alternative text for pictures (a later PRP if accessibility work asks for it).
- No change to lughac; lugha-gtk additions only (0.1.22).

## ERROR HANDLING REQUIREMENTS

- The builders and modifiers can't fail. Missing files and icons show GTK's placeholder (Must Do); nothing is checked at view time, so pure tests see only the path or name.
- `.size(n)` on an icon with `n <= 0` means the theme's size. *(proposed)*
- Patches pointing at the wrong kind of node keep panicking with a message naming the patch.

## SECURITY CONSIDERATIONS

- Each new extern matches its header; `gtk_image_get_icon_name` returns `const char *` that GTK owns (copied, `NULL` as `None`); `g_file_get_path` returns memory the caller frees (freed, as `lgtk_text_get`'s is).
- A picture's path comes from the app; kivuli opens nothing itself (GTK reads the file). Image decoding is GTK's loaders (gdk-pixbuf / GTK's own), already linked: no new libraries.
- No new closures handed to C.

## TESTS TO WRITE

Pure:
- [x] `test/lib.la`: `icon` and `image` make their nodes; `.icon`, `.fit` and `.size` land in the style; `.fit` defaults to `Contain`.
- [x] `test/diff.la`: a changed icon name is `SetIcon`, a changed path `SetFile`, unchanged ones nothing; adding `.icon` to a button or changing `.fit` is a `SetStyle`; an icon becoming an image is `Replace`.
- [x] `test/testing.la`: an icon-only button is clicked by its tooltip; `texts` skips icons and images.
- [x] `test/map.la`: icons and images pass through `map`.

Live GTK (skipped without a display), `test/live_images.la`:
- [x] an icon's name and pixel size read back; `.size(48)` → 48, dropped → the theme's (-1); a renamed icon shows the new name.
- [x] an image's path and fit read back; a new path and a new fit change them in place (same widget); a missing file still builds (no panic).
- [x] buttons: label only, icon only, both: which parts are visible and which theme class is set; adding, changing and dropping the icon; `SetLabel` still relabels; a click still sends its message.
- [x] `test/live.la`'s printer reads a button's label from kivuli's label child.
- [x] lugha-gtk `test/images.la`: each new extern set and read back.
- [x] `examples/gallery.la`'s own tests.

## ROLLBACK PLAN
- Branch `prp-005-icons_and_images` from `main`; abandon by leaving it unmerged. lugha-gtk 0.1.22 is additions only.

## ACCEPTANCE CRITERIA
- [x] Theme icons on their own and on buttons; pictures from files with `.fit`; missing ones show a placeholder.
- [x] Existing buttons look and behave as before (label-only buttons keep the `text-button` class).
- [x] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [x] Every example builds; fmt clean; public items documented; no file over 500 lines.
- [x] CHANGELOG.md, README.md and docs/DESIGN.md updated. Merged into `main`.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built; lugha-gtk `lughac test test/`.
- For the user: `lughac run examples/gallery.la` from the repo root: icons follow your light or dark theme; the four pictures show each fit; the buttons look like GTK's own.

## NOTES FROM THE BUILD

- **GTK's theme has no `image-text-button`.** The PRP guessed three classes; GTK's Default theme (read from the stylesheet in libgtk-4) styles a button with an icon and a label as `button.text-button.image-button`. kivuli sets `text-button` for a label, `image-button` for an icon, both for both, as GTK does itself; the theme's label padding gives the gap, so kivuli adds no spacing. `docs/DESIGN.md` records it.
- **Every button's content is kivuli's**: a `GtkBox` with a `GtkImage` (hidden until `.icon` names one) and a `GtkLabel`; `styling.show_content` sets visibility and classes, after `SetStyle` and after `SetLabel` (mutation-checked: without the call after `SetLabel`, an icon-only button keeps an empty label and the `text-button` class). Buttons are made with `gtk_button_new_with_label("")`, whose label `set_child` replaces: lugha-gtk binds no plain `gtk_button_new`.
- **The style types moved to `src/looks.la`** (views 470 → 372 lines). Tests imported `Style` and `plain` through `kivuli.views` and kept compiling after the move: lughac lets a module's private imports be imported through it (DECISION-011, deferred, reported). The tests now import from `kivuli.looks`.
- **A component made only of icons and pictures** has no message for lughac to infer `Msg` from before `.map`: annotate it (`let part: View<Part> = …`). Expected, not a bug.
- **GTK's default fit is already `contain`**, matching `Fit.Contain`; kivuli sets it anyway, so a dropped `.fit` is undone.
- **The sample picture** is 128×64 (697 bytes), generated with a short Python script (not kept in the repo); smaller than the gallery's 160-pixel boxes, so `ScaleDown` stays small while `Contain` grows, `Cover` crops and `Fill` squashes the sun into an oval.
- **Not covered by a test:** that a real file's picture loads (tests use missing paths, which GTK accepts silently); the gallery's look, for the user to judge.
