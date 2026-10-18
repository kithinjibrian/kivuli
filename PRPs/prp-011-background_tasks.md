# PRP-011 — Background tasks: `Cmd.task` and `Sub.receive`

Status: draft, for the user's approval
Discovery: session 12, 2026-10-18, talked through. *(you)*: chosen in the talk; *(proposed)*: for you to confirm or change with this PRP.

## FEATURE: Lugha's green threads run while GTK's main loop runs; `Cmd.task(work, done)` runs work in the background and sends its result to `update` as one message; `Sub.receive(chan, f)` turns every value sent on a channel into a message.

## OBJECTIVE

An app's `update` must never block the window, but apps need slow work: downloads (`std.http`), waiting, computing. Rapidkoi needs this to fetch bundles (its DECISION-021), and kivuli's image model needs it for web images (DECISION-019: apps fetch, images show bytes). *(you)*

```
// one-shot: run work in the background, get one message back
Open(url) => (Model { page: Page.Loading(url), ..m },
              Cmd.task(fun(): Result<u8[], string> = download(url), Msg.Got)),
Got(Ok(bytes)) => (Model { page: Page.Loaded(bytes), ..m }, Cmd.none()),

// open-ended: a worker built with Lugha's own spawn, channels or actors reports as it goes
fun subscriptions(m: Model): Sub<Msg> = Sub.receive(m.progress, Msg.Progress);
```

## WHAT LUGHA ALLOWS (read in the spec and, for reference, `../lugha/runtime`)

- **One OS thread.** Every green thread shares the program's one OS thread; Lugha has no OS threads, and "a callback must be called on the thread that registered it". So an OS thread per task (option b in the brief) is impossible.
- **GTK's main loop holds that thread.** `g_application_run` is one `extern` call for the app's whole life, and `extern` calls run to completion, so today a `spawn`ed green thread inside a kivuli app never runs at all.
- **`task.yield_now()` is exactly the hook needed, and it never blocks** (`runtime/spawn.c`): it wakes due sleepers, polls ready sockets with `epoll_wait(…, 0)`, and if any green thread is ready, runs them for one round (a busy one is cut off after 4,096 steps by Lugha's preemption) before returning.
- So **option (a): a GLib source drives Lugha's scheduler** by calling `yield_now()` while kivuli has work in flight. No change to lughac.

**What then runs in the background** (documented in the README and DESIGN.md):

| Work | |
| --- | --- |
| Sockets: `std.http`, `std.net`, `std.tls`; `task.sleep` | in the background: the window stays responsive |
| CPU work written in Lugha | time-sliced with GTK: the window stays responsive, the work runs slower than alone |
| One `extern` call or blocking intrinsic: `read_bytes` and other file calls, DNS lookups (at a connection's start), `std.crypto` (OpenSSL), `std.process` | **still blocks the window**: it holds the one OS thread, task or not. Only a Lugha runtime change can move these (DECISION-022, below) |

## DECISIONS MADE IN THE TALK (session 12)

- **Build now on `yield_now`** (option 1), and record two requests for the lugha session instead of working around them. *(you, by going ahead; proposed: the wording)*
- **`Cmd.task` is `spawn` plus a channel, not a new mechanism**: what kivuli adds is giving green threads their turns, and handing a result to `update` at a safe moment (only `update` changes the model, one message at a time, on GTK's thread). `update` stays pure: it describes the work, kivuli runs it, tests stay offline. *(you)*
- **`Sub.receive(chan, f)` in this PRP**, so apps can use Lugha's own `spawn`, channels and actors for long-running workers and progress. *(you)*
- One form of `Cmd.task` for now (no `task_with` shorthand). *(proposed)*

## CONTEXT

- Starting state: kivuli 0.5.0. `src/effects.la` (`Cmd`: none, after, quit, focus, batch; `Sub`: none, every, batch; `commands`, `timers`), `src/live.la` (`perform`, `resubscribe`, `dispatch`, GLib timers with `g_timeout_add_full`); lugha-gtk 0.1.25.
- Ending state: `src/effects.la` gains `Cmd.task` and `Sub.receive`; `src/live.la` gains the background source (or a new small `src/background.la` if live.la would pass 500 lines); tests in `test/effects.la`, `test/live_tasks.la` (new); a new `examples/download.la`; kivuli 0.6.0.
- GTK/GLib functions needed: none new (`g_timeout_add_full`, `g_get_monotonic_time` are bound). Lugha: `spawn`, `Chan`, `std.task.yield_now`.
- Open decisions that must be resolved first: none. Resolves Rapidkoi's DECISION-021 (option A) in that repo.

## IMPLEMENTATION REQUIREMENTS

### The API
- **`Cmd.task(work: fun(): T, done: fun(T): Msg): Cmd<Msg>`**: runs `work` on a new green thread; when it returns, `update` gets `done(result)`, exactly once. `done` runs in the task, so `Cmd` keeps only `Msg` as its type. `Cmd.map` and `Cmd.batch` work with it. *(you)*
- **`Sub.receive(chan: Chan<T>, f: fun(T): Msg): Sub<Msg>`**: while `subscriptions` returns it, every value sent on `chan` arrives as `f(value)`, in order. Values sent while nobody subscribes wait in the channel (a buffered channel holds them; an unbuffered `send` waits for the next turn kivuli takes). Closing the channel just ends its values. `Sub.map` and `Sub.batch` work with it. *(you)*
- The work runs on a copy of what it captured (Lugha's rule for closures and `spawn`); it must not call kivuli or GTK (it isn't on GTK's turn). *(proposed: documented)*
- **No cancellation**: a task runs to its end; an app ignores a result it no longer wants (a closed tab). **A panic in a task ends the program**, as any green thread's does. *(you)*

### Running it (proposed)
- While at least one task is unfinished or a `Sub.receive` is active, a GLib timeout (`g_timeout_add_full`, a few milliseconds) runs a **turn**: it calls `yield_now()`, and as long as that keeps finding work (a call that takes measurably long), calls it again, up to a time budget well inside the tick, so CPU-bound Lugha work gets a fair share and the window keeps its frames. Then it collects finished tasks' messages and drains each `Sub.receive` channel (`try_recv`, a bounded number per turn), and dispatches them in order, each a normal `dispatch` on GTK's thread. The tick interval and budget are tuned in the build against a test, and recorded. *(proposed)*
- With no task and no receive, the source removes itself: an idle app costs nothing. A new task or a receive starts it again.
- A task's result travels on a one-value buffered channel kivuli makes; the turn takes it with `try_recv`, so kivuli itself never blocks.
- Tests: `kivuli.testing` needs nothing new; an app test calls `update`, sees a task in `commands(cmd)`, and feeds `update` the result message itself.

### Docs, example, release
- README (commands and subscriptions, and the table of what runs in the background), docs/DESIGN.md (the runtime facts above, model B completed), CHANGELOG. *(you)*
- **`examples/download.la`**: a small local HTTP server (in a task, on `localhost`, so no internet is needed) answering slowly; the window downloads from it with `Cmd.task` and shows a spinner (animated text, `Sub.every`) while the window stays usable; and a second panel with a prime-counting worker reporting progress through `Sub.receive`. Its own tests. *(proposed)*
- Release 0.6.0, tag once every check passes; push on your go. *(you)*
- DECISIONS.md, for the lugha session (reported, not worked around):
  - **DECISION-021**: runtime hooks so GLib waits exactly instead of polling: the `epoll` descriptor, the next sleeper's deadline, and whether any green thread is ready.
  - **DECISION-022**: blocking calls (file I/O, DNS, `std.crypto`, `std.process`) off the main thread, e.g. a worker pool in the runtime that parks only the calling green thread.

### Must NOT Do
- No OS threads, no change to lughac (requests only). *(you)*
- No cancellation, no task priorities, no progress API beyond `Sub.receive`. *(you)*
- No kivuli or GTK calls from inside a task.

## ERROR HANDLING REQUIREMENTS

- A task's failure is the app's own value (`Result` from `work`), delivered as a message. A panic ends the program (Lugha's rule).
- A `done` or `f` that panics ends the program likewise.

## SECURITY CONSIDERATIONS

- No new externs or libraries. Tasks run app code with the app's permissions; kivuli opens no connection of its own (DECISION-019).
- The example's server listens on `localhost` only, on a free port.

## TESTS TO WRITE

Pure (`test/effects.la`):
- [ ] `Cmd.task` appears in `commands(cmd)`, through `batch` and `map` (its message mapped); running its work in the test gives the expected message.
- [ ] `Sub.receive` appears in the subscriptions' receivers, through `batch` and `map`; values sent on a buffered channel come out as messages, in order; an empty channel gives none.

Live GTK (skipped without a display), `test/live_tasks.la`:
- [ ] **responsive**: a task that sleeps 300 ms; meanwhile a GTK timeout at 50 ms clicks a button: `update` gets the click before the task's message, and the task's message arrives once.
- [ ] **sockets**: a task serving one HTTP request on localhost and another doing `std.http.get` to it, both in the background, while GTK runs: the body arrives as a message.
- [ ] **CPU work** in a task finishes while GTK keeps handling a repeating timeout (both make progress).
- [ ] **Sub.receive**: a worker sends three values on a channel: three messages, in order; when the subscription goes, later values aren't delivered.
- [ ] **idle**: with no task or receive, the source isn't running (checked through kivuli's own state).
- [ ] the example's own tests.

## ROLLBACK PLAN
- Branch `prp-011-background_tasks` from `main`; abandon by leaving it unmerged. No lugha-gtk change.

## ACCEPTANCE CRITERIA
- [ ] Downloads and sleeps run in the background while the window responds; CPU work in Lugha is time-sliced; results arrive as exactly one message; channels deliver as messages.
- [ ] What still blocks (file calls, DNS, `std.crypto`, `std.process`) is documented, and DECISIONS 021–022 are recorded and reported.
- [ ] Tests above written first; narrow runs while building; the full suite and the examples' tests pass before the merge.
- [ ] fmt clean; public items documented; no file over 500 lines.
- [ ] README, docs/DESIGN.md, CHANGELOG, MEMORY.md updated; 0.6.0 tagged; merged. Pushed only on your go.

## VALIDATION
- While building: only the tests covering each step.
- Before the merge: `lughac fmt --check src examples test`, `lughac test test/`, the examples' own tests, every example built.
- For the user: `lughac run examples/download.la`: start a download; the spinner turns and the window scrolls and clicks while it loads; the prime counter's progress climbs.
