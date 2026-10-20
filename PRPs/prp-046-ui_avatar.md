# PRP-046 — kivuli.ui: Avatar

Status: approved (session 102)
Discovery: session 101, 2026-10-20 (the reference's avatar: a round 32 px picture, a `Muted` fallback circle with initials when the picture can't load, a stacked group ringed in the page colour). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.avatar(image, initials)`, `Avatar.size(px)` and `avatar_group(avatars)`, from what kivuli has: an overlay, a checked picture, a rounded node that clips.

## OBJECTIVE

```
import kivuli.ui { Avatar, avatar_group };

ui.avatar(Some(Image.File(m.photo)), "CN")
ui.avatar(None, "LR")
ui.avatar(Some(Image.Bytes(key, bytes)), "ER").style([Avatar.size(40)])
avatar_group([ui.avatar(Some(a), "CN"), ui.avatar(Some(b), "LR"), ui.avatar(None, "ER")])
```

*(you: A)*

## REQUIREMENTS

- **`ui.avatar<Msg>(image: Option<Image>, initials: string): View<Msg>`**: an overlay; its base the fallback (`initials` centred, `Font.sm()`, `Font.medium()`, `MutedForeground`, on `Muted`); its layer the picture (`image(…)`, `Fit.Cover`, filling the avatar). A picture that can't be shown shows nothing (kivuli's rule for every picture), so the initials show through: the fallback needs no failure handling of the app's. `None`: the initials alone.
- The avatar 32 × 32 (an exact size: kivuli's clamp), fully rounded, so the picture is clipped round.
- **`Avatar.size(px)`**: a `Look` setting the exact size both ways *(proposed: the picture shrinks to it, `Fit.Cover`)*.
- **`avatar_group<Msg>(avatars: View<Msg>[]): View<Msg>`**: a row, each avatar after the first 8 px over the last (a −8 px start margin), each ringed by a 2 px halo of `Background` (a shadow: it takes no room) so the overlaps read *(proposed)*.

### Must NOT Do
- No status dots or badges on an avatar (an app overlays a `badge`), no loading of pictures from URLs, no square avatars beyond `Border.rounded(…)` an app adds.

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [ ] `test/avatars_ui.la` (new, pure): the overlay's base and layer, the fallback's look, the picture's fit, `None` showing initials alone, `Avatar.size`, `avatar_group`'s margins and rings.
- [ ] GTK parses the generated rules; `examples/components.la` gains an Avatar section (the sample picture, initials, a broken picture showing its initials, a group, a large one); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.41.0, CLAUDE.md; merged, tagged; pushed only on your go.
