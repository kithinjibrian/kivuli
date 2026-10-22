# PRP-056 — Group states: a child styled by its group's hover or focus

Status: built (session 136); merged into main in session 136 (its own tests run and pass)
Discovery: session 135, 2026-10-22 (rapidkoi's DECISION-036: a small × on each app in its sidebar, shown whenever the pointer is anywhere over that app's button; Tailwind's `group` / `group-hover`; GTK's CSS has descendant selectors, `.parent:hover .child`). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `Group.mark()`, a style value making a node a group; `State.GroupHover` and `State.GroupFocusWithin`, states a node takes while an enclosing group is hovered or holds the focus.

## OBJECTIVE

```
button_with(row([face, spacer(), close]), Msg.Select(id)).style([Group.mark()])

close.style([
    Effects.opacity(0.0),
    When(State.GroupHover, [Effects.opacity(1.0)]),
    When(State.GroupFocusWithin, [Effects.opacity(1.0)]),
])
```

*(you: A, a style value, as `views.la` is at its limit; A, hover and focus-within)*

## REQUIREMENTS

- **`Group.mark()`** (a new style group, `styles/groups.la`; *proposed: the file and name*): the node's widget gets the class `kv-group`. A `Look` can hold it, so kivuli.ui components can be groups.
- **`State.GroupHover`**: the node's values while an enclosing group is hovered: the rule `.kv-group:hover .kv-ghover-… { … }` (a descendant selector). **`State.GroupFocusWithin`**: while something in the group has the focus: `.kv-group:focus-within .kv-gfocus-… { … }`.
- **Order**: a group state's rule outranks the node's own plain values (it comes later and is more specific), and the node's own `Hover`/`Press`/`Focus` outrank it *(proposed: the node's own state is the closer one)*.
- **Inside a part**: `Inside(Part.X, [When(State.GroupHover, …)])` styles the part while the group is hovered, as other states inside parts do *(proposed; tested)*.
- **Nesting**: any enclosing group counts (CSS's descendant selector): a node inside two groups shows on either's hover. Tailwind's named groups (`group/name`) are not added.
- **Overlays**: a layer isn't inside the base on GTK's side, so the group goes on the overlay itself: `overlay(base, [close]).style([Group.mark()])`. Documented, and tested.
- `Group.mark()` on a bar's style or inside a state or part is refused by the checks (`Group.mark() belongs on a node's own style`) *(proposed)*.

### Must NOT Do
- No named groups, no `GroupPress`, no making a hidden node ignore clicks (an opacity-0 × still takes them: rapidkoi's fine with that).

## TESTS (narrow; type-check every test and example; run this PRP's own)
- [x] `test/groups.la` (new, pure): `Group.mark()` gives the `kv-group` class; `GroupHover` and `GroupFocusWithin` rules and their selectors; a group state inside a part; the order against the node's own states; the refusals.
- [x] `test/states.la`: the two states merge as the others do.
- [x] `test/live_style.la` (or a new `live_groups_states.la`): a marked node's widget has `kv-group`; an overlay marked as a group, its layer's group rule under it (the classes read back). Hover itself can't be driven without a pointer: how it looks, by hand.
- [x] GTK parses the generated rules; `examples/components.la` gains a "Group hover" row (tiles whose × shows on hover); only it built.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.52.0, CLAUDE.md; merged, tagged; pushed only on your go. Then rapidkoi's DECISION-036 can take it, in its own session.

## NOTES FROM THE BUILD

- **The order differs from the proposal**: the node's own `Hover` doesn't outrank a group state. `.kv-group:hover .kv-ghover-…` is three classes' worth of specificity, and the node's own Hover rule two (its class once and `:hover`); lowering the group's would mean raising every state's rank, changing every existing rule. So: a group state outranks the node's own Hover, ties Focus (the later rule wins), and `FocusWithin`, `Press`, `Checked`, `Disabled` outrank it.
- **The key stays node-first** (`kv-ghover-…`), as `paint` reads a node's class from the key's front; `rule_text` (motion_rules.la) puts the group's state before it when writing the rule. A part's (`kv-check-ghover-… check`) works the same way.
- **`Group.mark()` is a rule with no declarations** (`kv-group`), so `paint` gives the node its class like any value's.
- **The gallery's Group hover row** uses overlays (the × a layer), as rapidkoi's sidebar may: a `button_with` can't hold a button.
- **Tests run and pass**: groups (5), states, sheets, parts, styles (38 with them, pure); live_group_states (a group's class, a child's and an overlay layer's group classes), live_type. Every test and example type-checks; components built; GTK parsed the rules (0 errors).
