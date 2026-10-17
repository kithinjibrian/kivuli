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
        button("Add", Msg.Add).style(ButtonStyle.Primary),
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

kivuli isn't published yet. Once it is, list it in your program's `lugha.toml`:

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
| `.size(points)`, `.bold()`, `.dim()` | Text. |
| `.wrap()`, `.wrap_at(chars)`, `.ellipsize()` | Long text on text and check box labels: onto more lines (at most `chars` wide), or cut with "…"; both: up to three lines, the last cut. |
| `.selectable()` | Text the user can select and copy. |
| `.tooltip(hint)` | Hover text on any node; tests can click a button by it. |
| `.style(ButtonStyle.Primary \| Danger \| Subtle \| Normal)`, `.disabled(bool)` | Buttons. |
| `.placeholder(hint)`, `.on_submit(msg)` | Text fields: a hint while empty; the message Enter sends. |
| `.name(id)` | A name tests find a text field or text area by (not shown). |
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
| `Sub.none()`, `Sub.batch(subs)` | Nothing; several subscriptions. |
| `Sub.every(ms, f)` | Send `f(now)` every `ms` milliseconds (`now` in milliseconds since 1970) while `subscriptions` keeps returning it. |

## Testing an app

`update` and `view` are pure, so app tests need no window. `kivuli.testing` works out what a user's action would send from a view alone:

```
import kivuli.testing { Lookup, click, is_disabled, type_into, submit, toggle, texts };

@test
fun adding_a_task() {
    assert_eq(is_disabled(view(start()), "Add"), Ok(true));
    assert_eq(type_into(view(start()), "What needs doing?", "milk"), Ok(Msg.Draft("milk")));
    assert_eq(click(view(start()), "Remove"), Err(Lookup.NotFound("button \"Remove\"")));
}
```

`click` and `is_disabled` (by a button's label or tooltip), `tooltip`, `type_into` and `submit` (finding a text field by its placeholder or `.name`, a text area by its `.name`), `toggle` and `texts` each return a `Lookup` error when what's asked for isn't there (`NotFound`), isn't unique (`Ambiguous`) or sends nothing (`Silent`). `examples/todo.la` ends with its own tests: `lughac test examples/todo.la`.

## Tests

```sh
lughac test test/
```

The tests of the view tree, the diff, effects, `lazy` and the testing helpers need no display. The tests of the live GTK layer (`test/live*.la`) build hidden windows and read the widgets back from GTK; without a display they report themselves skipped.

## License

GPL-3.0-only, like Lugha.
