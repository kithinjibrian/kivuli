# kivuli

*kivuli* (Swahili: "shadow"): the view is a shadow of the model.

A declarative UI framework for [Lugha](https://github.com/kithinjibrian/lugha), built on the Elm architecture and rendered to real GTK 4 widgets through [lugha-gtk](https://github.com/kithinjibrian/lugha-gtk). An app is three pure functions: a model, `update` making the next model from a message, and `view` describing the screen. kivuli shows the view, turns clicks and keystrokes into messages, and on each one diffs the new view against the last and changes only the widgets that differ.

```
import kivuli { View, ButtonStyle, button, column, input, row, sandbox, text };

struct Model {
    count: i64,
    name: string,
}

enum Msg {
    Add,
    Reset,
    Named(string),
}

fun update(m: Model, msg: Msg): Model = match msg {
    Add => Model { count: m.count + 1, ..m },
    Reset => Model { count: 0, ..m },
    Named(name) => Model { name: name, ..m },
};

fun view(m: Model): View<Msg> = column([
    input(m.name, Msg.Named).placeholder("Your name"),
    text("Hello, {m.name}").bold(),
    text("Clicked {m.count} times"),
    row([
        button("Add", Msg.Add).button_style(ButtonStyle.Primary),
        button("Reset", Msg.Reset).disabled(m.count == 0),
    ]).spacing(8),
]).padding(24).spacing(8).window("Counter", 320, 240);

fun main(): i32 = sandbox("org.kivuli.counter", Model { count: 0, name: "" }, update, view);
```

The app never touches a widget. A disabled button has no message to send; a text field keeps its cursor while you type; nothing is rebuilt that didn't change.

## Requirements

- `lughac` built from [lugha](https://github.com/kithinjibrian/lugha)'s `main`, milestone 100 or later (kivuli uses struct update, variants as functions, package entries, glob re-exports and recent inference fixes). In a lugha checkout: `cargo install --path . --locked`.
- GTK 4's development files and `pkg-config`. On Ubuntu:

```sh
sudo apt install libgtk-4-dev pkgconf
```

lugha-gtk (0.1.13 or later) is fetched by `lughac` itself, as a git package.

## Using it

List it in your program's `lugha.toml`:

```toml
[packages]
kivuli = { git = "https://github.com/kithinjibrian/kivuli", version = "0.1" }
```

and write `import kivuli { … };` or `import kivuli;` (then `kivuli.sandbox(…)`).

Inside this repository, run the examples directly:

```sh
lughac run examples/counter.la
lughac run examples/todo.la
lughac run examples/stopwatch.la
lughac run examples/counters.la
lughac run examples/notes.la
lughac run examples/settings.la
lughac run examples/gallery.la
```

## What's there

**Running an app**

| | |
| --- | --- |
| `sandbox(id, init, update, view)` | An app without effects: `update(model, msg)` returns the next model. |
| `application(App { id, init, update, view, subscriptions })` | An app with effects: `init` and `update` also return a `Cmd`, and `subscriptions(model)` says what to keep listening to. |

**Widgets**

| | |
| --- | --- |
| `text(s)` | A line of text. |
| `button(label, msg)` | Sends `msg` when clicked. |
| `input(value, on_change)` | A one-line text field; each edit sends `on_change(text)`. |
| `text_area(value, on_change)` | A multi-line text field that wraps and scrolls inside itself; each edit sends `on_change(text)`, Enter starts a new line. |
| `checkbox(label, checked, on_toggle)` | Each toggle sends `on_toggle(ticked)`. |
| `dropdown(choices, selected)` | A drop-down of `(label, msg)` choices; picking one sends its message. `selected` is the label shown picked; one matching no label shows a blank item. |
| `icon(name)` | A theme icon by name (`"user-trash-symbolic"`); symbolic icons follow the light or dark theme. An unknown name shows GTK's placeholder. |
| `image(path)` | A picture from a file (PNG, JPEG, SVG …), scaled to its room. A relative path is relative to where the app was started; a missing file shows nothing. |
| `slider(min, max, value, on_change)` | A whole-number slider showing its number; each move to a new step sends `on_change(value)`. `.step(n)` moves it in steps of `n` from `min`. |
| `column(children)`, `row(children)` | Children top to bottom, or left to right. |
| `keyed_column(children)` | Like `column`, with `(key, view)` children matched by key: reordering, inserting or deleting moves, adds or removes just those widgets. |
| `scroll(child)` | Scrolls its child when it's bigger than the room it's given. |
| `spacer()`, `separator()` | Room that grows along its column or row; a line across it. |
| `lazy(key, build)` | Calls `build` only when `key` changes at that place; otherwise reuses the subtree and skips diffing it. The key must cover everything the subtree shows. |

**Modifiers**, chained on any view (a modifier that doesn't fit a node is ignored):

| | |
| --- | --- |
| `.padding(n)`, `.spacing(n)` | Room around a node; between a column's or row's children (pixels). |
| `.expand()`, `.align(Align.Start \| Center \| End)` | Take spare room along the parent; place across it. |
| `.width(n)`, `.height(n)` | A minimum size in pixels. |
| `.size(points)`, `.bold()`, `.dim()` | Text. On an icon, `.size(n)` is its size in pixels. |
| `.wrap()`, `.wrap_at(chars)`, `.ellipsize()` | Long text on text and check box labels: onto more lines (at most `chars` wide), or cut with "…"; both: up to three lines, the last cut. |
| `.selectable()` | Text the user can select and copy. |
| `.tooltip(hint)` | Hover text on any node; tests can click a button by it. |
| `.shortcut(keys)` | Keys that press a button from anywhere in the window: `"Ctrl+N"`, `"Ctrl+Shift+Delete"`, `"Escape"`. A disabled button's shortcut does nothing; unreadable keys, or two enabled buttons with the same ones, stop the app with a message. |
| `.autofocus()` | Keyboard focus goes to the node when it's first shown. |
| `.button_style(ButtonStyle.Primary \| Danger \| Subtle \| Normal)` | Buttons. |
| `.disabled(bool)` | Buttons, text fields and text areas: greyed out by the theme, and they send nothing. A disabled field still shows its value and follows the model. `.disabled(false)` changes nothing. |
| `.step(n)` | Sliders: move in steps of `n`, counted from the least value. |
| `.icon(name)` | Buttons: a theme icon before the label; with the label `""`, an icon-only button (give it a `.tooltip`, which tests find it by). |
| `.fit(Fit.Contain \| Cover \| ScaleDown \| Fill)` | Pictures: the whole picture (default), filled and cropped, never enlarged, or stretched. |
| `.placeholder(hint)`, `.on_submit(msg)` | Text fields: a hint while empty; the message Enter sends. |
| `.name(id)` | A name tests find a text field, text area, drop-down or slider by (not shown). |
| `.window(title, width, height)` | On the root of `view`: the window's title (it may follow the model) and size. |

Everything visual comes from the GTK theme (button styles are the theme's `suggested-action`, `destructive-action` and `flat`): there are no colours to set, and apps follow the user's light or dark theme.

**Components.** A part of an app can have its own model, messages, `update` and `view`, and sit in a bigger app through `map`, which sends `f(msg)` wherever the part would send `msg`:

```
row([counter(m.left).map(Msg.Left), counter(m.right).map(Msg.Right)])
```

`Cmd` and `Sub` have `map` too, for a part's commands and subscriptions. `examples/counters.la` shows two copies of one component.

**Effects**, for `application`:

| | |
| --- | --- |
| `Cmd.none()`, `Cmd.batch(cmds)` | Nothing; several commands. |
| `Cmd.after(ms, msg)` | Send `msg` once, `ms` milliseconds from now. |
| `Cmd.quit()` | End the app. |
| `Cmd.focus(name)` | Move keyboard focus to the node named `name` (`.name(name)`). |
| `Sub.none()`, `Sub.batch(subs)` | Nothing; several subscriptions. |
| `Sub.every(ms, f)` | Send `f(now)` every `ms` milliseconds (`now` in milliseconds since 1970) while `subscriptions` keeps returning it. |

## Testing an app

`update` and `view` are pure, so app tests need no window. `kivuli.testing` works out what a user's action would send from a view alone:

```
import kivuli.testing { Lookup, click, is_disabled, is_field_disabled, type_into, submit, toggle, choose, slide, texts };

@test
fun adding_a_task() {
    assert_eq(is_disabled(view(start()), "Add"), Ok(true));
    assert_eq(type_into(view(start()), "What needs doing?", "milk"), Ok(Msg.Draft("milk")));
    assert_eq(click(view(start()), "Remove"), Err(Lookup.NotFound("button \"Remove\"")));
}
```

`click` and `is_disabled` (by a button's label or tooltip), `press` (by shortcut), `tooltip`, `type_into` and `submit` (finding a text field by its placeholder or `.name`, a text area by its `.name`), `toggle`, `choose(view, name, label)` and `slide(view, name, value)` (a drop-down or slider by its `.name`; `slide` clamps and rounds to the step as the slider would) and `texts` each return a `Lookup` error when what's asked for isn't there (`NotFound`), isn't unique (`Ambiguous`) or sends nothing (`Silent`: a disabled button or field, or Enter without `.on_submit`). `is_field_disabled(view, hint)` tells whether a text field or text area is disabled. `examples/todo.la` ends with its own tests: `lughac test examples/todo.la`.

## Tests

```sh
lughac test test/
```

The tests of the view tree, the diff, effects, `lazy` and the testing helpers need no display. The tests of the live GTK layer (`test/live*.la`) build hidden windows and read the widgets back from GTK; without a display they report themselves skipped.

## License

GPL-3.0-only, like Lugha.
