# PRP-031 — kivuli.ui: Card

Status: draft, for the user's approval (session 53)
Discovery: session 53, 2026-10-20 (the third component; the reference is the current card at ui.shadcn.com/docs/components/base/card). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `card([...])` and its parts `card_header(title, description)`, `card_header_with(title, description, action)`, `card_content([...])`, `card_footer([...])`, one size (16 px spacing).

## OBJECTIVE

A container in the palette's card colours with a header, content and a footer that runs to its edges. Parts are plain functions returning views, so any can be left out or repeated, and an app's own views go anywhere. *(you: A, A)*

```
import kivuli.ui { card, card_header, card_header_with, card_content, card_footer, badge, Button };

card([
    card_header("Login to your account", "Enter your email below to log in"),
    card_content([email_field, password_field]),
    card_footer([
        ui.button("Login", Msg.Login),
        ui.button("Login with Google", Msg.Google).style([Button.outline()]),
    ]),
]).style([Sizing.max_width(380)])

card([
    card_header_with("Design systems meetup", "A practical talk on component APIs", badge("Featured")),
    card_content([text("Thursday, 6 pm")]),
])
```

## REQUIREMENTS

### The card *(you: A: parts)*
- `card<Msg>(parts: View<Msg>[]): View<Msg>`: a column, `Background.color(Token.Card)`, `Text.color(Token.CardForeground)`, `Font.sm()`, a 1 px border of `Foreground` at 10% (the reference's ring), padding 16 top and bottom and none across (the parts pad themselves across), 16 px between parts, corners rounded by the palette's radius plus 4 (shadcn's `rounded-xl`), which clips the footer's corners.
- **`Radius.XLarge` and `Border.rounded_xl()`** *(proposed: the radius + 4 px, `calc(var(--kv-radius) + 4px)`; the card needs one size above `Large`)*.
- One size *(you: A)*: the small size waits until a container's spacing can reach its parts (recorded as an open question in DECISIONS).

### The parts
- `card_header<Msg>(title: string, description: string): View<Msg>`: the title (`Font.medium()`, the theme's size, one size up from the card's text) over the description (`Font.sm()`, `Token.MutedForeground`), 4 px apart, 16 px across. An empty description is left out.
- `card_header_with<Msg>(title: string, description: string, action: View<Msg>): View<Msg>`: the same, with `action` (a button, a badge) at the top of the end side.
- `card_content<Msg>(children: View<Msg>[]): View<Msg>`: a column, 16 px across, 8 px between children *(proposed: the reference's content is a plain block, and apps nearly always stack fields; 8 is the form gap)*.
- `card_footer<Msg>(children: View<Msg>[]): View<Msg>`: a row, 8 px apart *(proposed)*, padding 16, a 1 px `Border` top, `Muted` at 50% behind it, and a margin of −16 at the bottom so it meets the card's edge (the card's corners clip it).

### Must NOT Do
- No small size, no image slot (an app puts a `picture` first; its top corners are clipped by the card), no clickable card.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [ ] `test/cards.la` (new, pure): the card's values; each part's values; the empty description left out; the header with an action; the footer's margin; parts left out and repeated; the OBJECTIVE reads as written.
- [ ] `test/palettes.la`: `Radius.XLarge`'s CSS.
- [ ] GTK parses the generated rules (the C probe); `examples/components.la` gains a Card section (login, a header with a badge, a terms card with two buttons), the only example built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.26.0, CLAUDE.md, DECISIONS (container spacing reaching its parts: open); merged, tagged; pushed only on your go.
