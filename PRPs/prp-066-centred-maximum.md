# PRP-066 — A maximum size, placed by `.align`

Status: approved (2026-10-24: start, centre and end; lugha-gtk 0.1.54, pushed and tagged)
Request: Rapidkoi's DECISION-061, section 14 of `../rapidkoi/docs/specs/kivuli-requests.md`

## FEATURE: a node with a maximum size and an alignment is as big as there's room for, at most its maximum, and sits where its alignment says (CSS's `max-width` with `margin: 0 auto`).

## OBJECTIVE

```
column([
    note_body(m).style([Sizing.max_width(680)]).align(Align.Center),
])
```

On a 1200 px window the note is a 680 px column in the middle; on a 500 px one it's 500 px wide. Today it shrinks to its natural width, a narrow strip of wrapping text.

## CONTEXT
- `src/render/styling.la` (the `Clamp` arm of `apply_style`) and lugha-gtk's `src/c/clamp.c`. Today the clamp copies the node's `halign`/`valign`, so a centred clamp gets its natural width. When it has more room than its maximum, it puts its child at (0, 0).
- GTK (`gtkwidget.h`): `gtk_widget_allocate(child, w, h, baseline, GskTransform*)`, and `gtk_widget_get_direction` for right-to-left. `GtkAlign` values (`gtkenums.h`): 0 fill, 1 start, 2 end, 3 center. libadwaita's `AdwClamp` does the same: it gives the child its maximum and centres it.
- `test/live_sizing.la` expects "align on the clamp 3"; this changes it.
- No open decision is involved.

## REQUIREMENTS

### Must Do
- In an axis where the node has a maximum (or an exact size) and an alignment other than `Fill`:
  - the clamp fills that axis (keeping the node's expand);
  - the node gets the room, up to its maximum, placed by its alignment: start, centre or end. Start and end flip right-to-left.
- In an axis with no maximum, and with `Fill`, nothing changes.
- lugha-gtk 0.1.54: `lgtk_clamp_set_align(clamp, halign, valign)`; the clamp places its child by them when it has more room than the child's size. The default, fill, keeps today's behaviour.
- CHANGELOG 0.62.0, README if it describes sizes.

### Must NOT Do
- No new modifier: `max_width` and `.align`/`.place` are enough, as Rapidkoi already writes them.
- No change to nodes without a maximum: a centred node keeps GTK's natural width.

## TESTS
- [ ] lugha-gtk `test/clamp.la`: a clamp with max 100 in 300 px, centred, puts its child at x = 100, width 100; with end, at x = 200; with fill, at x = 0, as before.
- [ ] `test/live_sizing.la`: a centred node with a maximum has a clamp that fills (halign 0), with the clamp placing it at centre. A restyle to `Fill` puts it back.

## ACCEPTANCE
- [ ] lugha-gtk tests pass; kivuli's full suite (in batches), the examples' tests, `fmt --check`; every example type-checks
- [ ] CHANGELOG.md updated
- [ ] For the user, by eye: `examples/borders` gains a centred `max_width 560` text. Widen the window: it stays 560 and centred. Narrow it: it shrinks with the window.
- [ ] Once tagged, the user is told, so Rapidkoi can resolve DECISION-061
