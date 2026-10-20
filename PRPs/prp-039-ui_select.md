# PRP-039 — kivuli.ui: Select

Status: built (session 79); merged into main in session 80 without running its tests (the user's call)
Discovery: session 77, 2026-10-20 (the tenth component; named `dropdown`/`Dropdown` throughout, not Select: `select` is a Lugha keyword, and the native widget is `dropdown` (the user: A); the reference is ui.shadcn.com/docs/components/base/select, its code collapsed; GTK's drop-down probed: `dropdown > button.toggle` (the shown item, an `arrow`), `popover.menu > contents > … listview > row` (each item: a label and a tick image), the pop-up a CSS descendant of the drop-down). *(you)*: chosen; *(proposed)*: for you to confirm or change.

## FEATURE: `ui.dropdown(choices, selected, placeholder)` with `Dropdown.invalid()`; the parts `Part.Trigger`, `Part.Arrow`, `Part.List`, `Part.Item`; states inside a part; and `.placeholder(…)` on any drop-down.

## OBJECTIVE

```
import kivuli.ui { Dropdown };

ui.dropdown([("Apple", Msg.Fruit("apple")), ("Banana", Msg.Fruit("banana"))], m.fruit, "Select a fruit")
ui.dropdown(choices, m.fruit, "Select a fruit").style([Dropdown.invalid()])
field([field_label("Fruit"), ui.dropdown(choices, m.fruit, "Select a fruit"), field_error("Please select a fruit.")])
```

*(you: A)*

## REQUIREMENTS

### Parts, and states inside a part
- `Part.Trigger` (the drop-down's button: `> button`, a child, so the list's own buttons don't match), `Part.Arrow` (`arrow`), `Part.List` (the pop-up's box: `popover > contents`), `Part.Item` (an item in the pop-up: `popover row`, not the shown item in the trigger). A part's selector is a path where a bare node would match too much *(proposed)*; its class carries the part's name (`kv-item-…`).
- **A `When` inside an `Inside`** now puts the state on the part: `Inside(Part.Item, [When(State.Hover, […])])` is `.kv-item-hover-… popover row:hover`. (Before, it stopped the app.) An `Inside` inside an `Inside` still stops the app.

### The placeholder (native)
- `.placeholder(text)` on a drop-down: the blank item kivuli shows when `selected` matches no choice reads `text` instead of nothing (picking it still sends nothing). A changed placeholder updates the same widget *(proposed)*.

### The drop-down
- `kivuli.ui`'s `dropdown<Msg>(choices: (string, Msg)[], selected: string, placeholder: string): View<Msg>`: the native drop-down with the placeholder and:
  - the trigger: `Font.sm()`, a 1 px `Input` border, `Border.rounded_md()`, `Sizing.min_height(36)`, padding 12 across, a transparent background, the faint shadow; while it has keyboard focus, the `Ring` border and 3 px halo; when nothing is picked, its text `MutedForeground` (the placeholder showing);
  - the arrow: `MutedForeground`;
  - the list: `Popover` on `PopoverForeground`, a 1 px `Border` border, `Border.rounded_md()`, a medium shadow, 4 px padding;
  - items: `Font.sm()`, padding 6 down and 8 across (32 at the end, for the tick), `Radius.Small`; hovered, `Accent` on `AccentForeground`. The tick GTK shows on the picked item takes the item's colour.
- **`Dropdown.invalid()`**: the trigger's border `Destructive`, its focus halo `Destructive` at 20%.

### Must NOT Do
- No groups, labels or separators in the list; no disabled drop-downs (their messages are per choice: a later PRP); no item placed over the trigger (GTK places its pop-up below).

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] `test/parts.la`: the four new parts' selectors (a child, a path), under a widget state, and a state inside a part (`popover row:hover`); an `Inside` inside an `Inside` still refused.
- [x] `test/selects_ui.la` (new, pure): the trigger's, arrow's, list's and items' values, the focus ring, the muted placeholder when nothing is picked (and not when something is), `Dropdown.invalid()`, still native.
- [x] `test/live_choices.la`: a drop-down's placeholder reads as its blank item, and a changed placeholder updates the same widget.
- [x] GTK parses the generated rules (the C probe); `examples/components.la` gains a Select section (plain, a placeholder, invalid inside a field); only it built. How it looks: by hand.

## ACCEPTANCE
- [ ] The OBJECTIVE compiles; fmt clean; no file over 500 lines; README, DESIGN, CHANGELOG 0.34.0, CLAUDE.md; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **The test is `test/dropdowns_ui.la`** (the component is `dropdown`).
- **The placeholder** lives in the drop-down's live `Choices`; `styling` sets it on a restyle and syncs GTK's list (not while GTK is inside the drop-down's own signal: the next sync shows it).
- **A part's state rule drops its rank repeats** (`c.c:active` → one class on the part).
- **Three stale tests fixed** (found by type-checking every test, which earlier PRPs skipped): `test/styles.la`'s `Text` and `Sheet` literals lacked fields added by PRP-027/030/036; `test/lib.la` called a check box's and a slider's handler directly after PRP-037/038 made them `Option`s.
- **GTK parsed the drop-down's CSS** (86 lines, 0 errors). Tests type-checked only; only components built.
