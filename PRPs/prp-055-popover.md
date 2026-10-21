# PRP-055 — Popover

Status: draft (session 132)
Discovery: session 132, 2026-10-21 (the reference's popover: a panel of any content in the popover colours, 288 px wide, `rounded-md`, a border, 16 px padding, a medium shadow, 4 px below its trigger, centred on it; a click outside or Escape closes it; it fades and zooms in). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `popover(trigger, open, content, on_close)` and `popover_at(side, …)`, native; `ui.popover(…)` and `ui.popover_at(…)` in the palette's look. Open is the model's.

## OBJECTIVE

```
ui.popover(
    ui.button("Dimensions", Msg.Open(true)).style([Button.outline()]),
    m.editing,
    column([
        text("Dimensions").style([Font.medium()]),
        field_horizontal([field_label("Width"), ui.input(m.width, Msg.Width)]),
        ui.button("Save", Msg.Save),       // Save sets editing to false: the popover closes
    ]),
    Msg.Open(false),                       // a click outside or Escape
)
popover_at(Edge.Top, trigger, m.open, content, Msg.Close)
```

*(you: A, the model decides; A, a builder, read from "continue": `ui.popover_at(Edge.Top, …)` for the side, in place of the `Popover.side` style first shown, which would need a new kind of style value)*

## REQUIREMENTS

### The view
- **`popover<Msg>(trigger: View<Msg>, open: bool, content: View<Msg>, on_close: Msg): View<Msg>`**: `trigger` shown in the page; `content` in a popover beside it while `open`, below it. **`popover_at(side: Edge, …)`**: on that side (`Edge.Top`, `Bottom`, `Left`, `Right`). GTK flips it when there's no room and centres it on the trigger.
- `View.Popover(View<Msg>, bool, View<Msg>, Msg, Edge, bool)`: the trigger, open, the content, the close message, the side, and whether GTK's arrow shows (`popover` yes, `ui.popover` no). `map`, `walk` (both trigger and content, so `kivuli.testing` finds buttons and fields inside an open popover *(proposed: inside a closed one too; it's in the view either way)*), `paint` (both), `lazies`, and `test/live.la`'s `shown()` follow.
- **Rule 7**: GTK closing it (a click outside, Escape) sends `on_close`; kivuli never closes it on its own. An `open: true` view after `on_close` (an app that ignores it) opens it again.

### On GTK
- `Live.Popover`: a box holding the trigger's widget, with a `GtkPopover` parented to that box (`gtk_widget_set_parent`), its child the content's widget, built whether open or not, so its fields keep their text and the content is diffed like any node (`Patch` paths go through it). `gtk_popover_popup`/`popdown` when `open` changes; position and arrow set at build and on change.
- Its `closed` signal (connected once, rule 5) sends the close message from a `Ref` unless kivuli itself closed it (guarded, as the echo rules: rule 8's spirit).
- **lugha-gtk 0.1.44**: `gtk_popover_new`, `gtk_popover_set_child`/`get_child`, `gtk_popover_set_position`/`get_position`, `gtk_popover_get_has_arrow` (checked against `gtk/gtkpopover.h`); `set_has_arrow`, `popup`, `popdown` exist. Pushed under the standing go.
- Unparented when the node goes (as context menus are, `popovers.la`), so nothing leaks.

### The look (`kivuli.ui`)
- **`Part.Popover`** (`> popover > contents`, under the wrapper box). `ui.popover`: `Popover` on `PopoverForeground`, a 1 px `Border` border, `Border.rounded_md()`, padding 16, 288 px wide (min and max *(proposed: a minimum, so wider content isn't cut; the reference's `w-72`)*), the medium shadow, `Motion.fade_in(150)`; no arrow.

### File sizes
- `views.la` (497) gains the variant's two lines (499). `diff.la` (478): the popover's diff arm calls a function in a new `core/diff_popover.la` *(proposed, approved with this PRP)*, so diff.la stays under 500.

### Must NOT Do
- No hover-opened popovers (that's a hover card), no modal popovers, no popover without a trigger, no arrow in `ui.popover`, no state of kivuli's own (anti-pattern 9).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/popovers.la` (new, pure): the node of `popover` and `popover_at`; `map`; `walk` reaches trigger and content; the diff: open → closed is one patch on the same node, content changes are patches inside it, a changed side or close message patches in place.
- [ ] `test/testing.la`: `click` finds a button inside a popover.
- [ ] `test/live_popover.la` (new, GTK): built closed, opened on update (visible), closed on update; GTK closing it (`popdown` from outside kivuli) sends the close message once, kivuli's own close sends nothing; a field inside keeps its text across open and close; the side and arrow read back; removing it unparents the popover.
- [ ] `test/popovers_ui.la` (new, pure), `test/parts.la`: the look's values; `Part.Popover`'s selector.
- [ ] lugha-gtk: a popover's child, position and arrow set and read back.
- [ ] GTK parses the generated rules; `examples/components.la` gains a Popover section (the reference's Dimensions form); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.51.0, CLAUDE.md; merged, tagged; pushed only on your go.
