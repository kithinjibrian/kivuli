# PRP-023 — The header bar as a node you style

Status: approved by the user (session 28); building on branch prp-023-header_bar
Discovery: session 28, 2026-10-20. *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `header_bar(start, title, end)` builds the bar; `.style([...])` styles it; `.header(bar)` puts it in the window's title bar.

## OBJECTIVE

The bar itself can't be styled since 0.11 (`.header_style` went with named looks). Styling stays one door: the bar is a thing you build and style like a node. *(you)*

```
fun view(m: Model): View<Msg> = page(m).header(
    header_bar([back], address(m), [info]).style([
        Background { color: Paint.Color(hex("#242424")), ..Background.theme() },
        Border { bottom: Side { width: Width.Px(1), color: Paint.Faded(0.1), style: Line.Solid }, ..Border.theme() },
        Shadow.off(),
    ]),
)
```

## REQUIREMENTS

- **`header_bar(start: View<Msg>[], title: View<Msg>, end: View<Msg>[]): Bar<Msg>`** and **`Bar.style(items: Styling[]): Bar<Msg>`**, merged as any node's style. `Bar` is its own small type, not a `View`, so it can only go in `.header` *(proposed: no new `View` node, so the diff, walk and map are untouched)*.
- **`.header(bar: Bar<Msg>)`** replaces `.header(start, title, end)` (breaking). The bar's style goes on GTK's header bar only; its window controls stay GTK's. The page's own `.style` still styles the page.
- **What works on a bar**: fonts and text (its title inherits them), background, borders, outline, shadows, effects, padding and margin, minimum heights, states. **Refused with a message** *(proposed)*: an exact or maximum size and a gap (`header_bar(…): a header bar's size and gap are the window's`), and a background image (it would need the clamp's runtime path; later if wanted).
- A bar style change is a restyle of the root, as the old header looks were.

## TESTS (narrow)
- [ ] `test/lib.la`: `.header(header_bar(…))` makes the framed root; the bar's style lands in the root's style; `.header(header_bar(…).style(…))` and the page's `.style` stay apart.
- [ ] `test/theming.la`: paint gives the bar's classes; the refusals.
- [ ] `test/live_header.la`: the bar carries its classes (and the window controls are still shown); a style change re-classes the same bar; the existing header tests, rewritten to `header_bar`.
- [ ] Every example builds; `examples/browser.la` styles its bar. No full suite (the user's call).

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.18.0 (Breaking: `.header(start, title, end)` → `.header(header_bar(start, title, end))`); merged, tagged; pushed only on your go.
