# DECISIONS.md — kivuli

Tracks architectural and design questions that are open, deferred, or resolved.

Rules:
- Every open decision blocks implementation of the code it affects.
- The AI must not implement anything that depends on an open decision.
- When a decision is resolved, move it to the RESOLVED section and record the outcome.
- Once resolved, copy the outcome to MEMORY.md as an architectural decision.

---

## OPEN — Requires human input before implementation

---

## DEFERRED — Acknowledged, not yet needed

### DECISION-005 — Multiple windows and dialogs

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 2
**Revisit when:** an example or a user app needs a second window or a dialog.

**Question:** How does `view` describe more than one window?

**Notes:** v1 is one window by decision.

---

### DECISION-006 — Framework-owned local widget state

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 7
**Revisit when:** a real widget can't be built without it (a v2 dropdown, for example).

**Question:** Should some widgets keep state outside the app's `Model`?

**Notes:** Elm manages without it. Adding it weakens "the view is a shadow of the model".

---

### DECISION-007 — Non-blocking I/O commands

**Status:** deferred
**Raised:** 2026-10-16 — kickoff brief, decision 6
**Revisit when:** a blocking `update` is a real problem in an app, or lugha's green threads/actors (v17) are stable enough to drive from the GLib loop.

**Question:** Should file/process/network I/O become `Cmd`s?

**Notes:** v1 lets `update` block.

---

## RESOLVED

### DECISION-003 — Module path of the testing helpers

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §7
**Resolved:** 2026-10-17 — Session 3

**Question:** How do app tests reach the testing helpers?

**Outcome:** `src/testing.la`, re-exported whole from the entry (`public import testing;` in `lib.la`): `import kivuli;` gives `kivuli.testing.click(…)`, and `import kivuli.testing { click };` works too. Text fields are found by placeholder, so `.placeholder(s)` and `.on_submit(msg)` (decision 3) are built first.

**Rationale:** matches decision 10's `ui.testing`, renamed with the entry. The helpers are pure, and lughac builds only what's used.

**Copied to MEMORY.md:** yes

### DECISION-002 — The shape of `lazy` (risk R1)

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §6
**Resolved:** 2026-10-17 — Session 3

**Question:** `lazy` must recognise an unchanged input next render, but `View` is generic only over `Msg`, so it can't hold the input. How is the input compared?

**Outcome:** a string key supplied by the app: `lazy(key: string, build: fun(): View<Msg>)`. Before each diff, kivuli resolves the new view: a lazy whose key it built last time reuses that subtree without calling `build`, and the diff skips two lazies with equal keys entirely. Built subtrees are cached by key, and the cache keeps only the keys the current view used. Modifiers go inside `build`; modifiers on `lazy(…)` itself are ignored.

**Rationale:** works with no language support, and it's the brief's preferred "simplest that works". The cost: a key that leaves out something the subtree depends on shows a stale subtree. Rejected: comparing the input with `==` (needs a type-erased comparable value Lugha may not have), and leaving `lazy` out of v1.

**Copied to MEMORY.md:** yes

### DECISION-010 — Modifiers wait for lughac: the expected type doesn't reach a method's receiver

**Status:** resolved
**Raised:** 2026-10-17 — Session 3 (step 5, part 1)
**Resolved:** 2026-10-17 — Session 3

**Question:** `fun v(): View<Msg> = text("x").padding(8);` was E0412: a generic call whose parameter appears only in its result couldn't be a method's receiver.

**Outcome:** fixed in lughac (lugha PR #48, 4e0b028): a method returning its receiver's own type passes the expected type down to the receiver, also inside lists, tuples, generic calls and `==`. Every kivuli modifier returns `View<Msg>`, so chains like `text("Total").padding(8).align(Align.Center)` check inside `column([...])`.

**Rationale:** the API stays as designed (decision 5). Limit to remember: only methods returning exactly the receiver's type (same parameters, same order) pass it down. A modifier returning anything else (say `Option<View<Msg>>`) wouldn't.

**Copied to MEMORY.md:** no (a lughac fix, not a kivuli decision). The limit is noted in CLAUDE.md.

### DECISION-009 — How a node carries its modifiers

**Status:** resolved
**Raised:** 2026-10-17 — Session 3 (step 5)
**Resolved:** 2026-10-17 — Session 3

**Question:** Does every variant get its own node struct with a layout field (the brief's sketch), or do modifiers live in one wrapper?

**Outcome:** one wrapper, `View.Styled(View<Msg>, Style)`, holding every modifier in one `Style` record. Chained modifiers merge into one `Style` (struct update from the current one, or from `plain()`), never nest. The diff splits each side into its inside and its style: same kind of inside gives `SetStyle(path, style)` when the styles differ, plus the inside's own patches, at the same path. A different kind gives `Replace` with the styled node. `live` applies the whole style on create and on `SetStyle`, using only the fields that fit the widget. `disabled` still goes through `Button`'s `Option<Msg>` (decision 6).

**Rationale:** no existing variant, diff arm or test changes, and new widgets get every modifier for free. Rejected: a record per node (type-precise, but every variant, arm and test changes, and every widget repeats the layout field).

**Copied to MEMORY.md:** yes

### DECISION-008 — How tests compare views once they hold functions

**Status:** resolved
**Raised:** 2026-10-17 — Session 2 (adding `input`)
**Resolved:** 2026-10-17 — Session 2

**Question:** `input`'s node holds `on_change: fun(string): Msg`. Functions have no `==`, so `View` (and `Patch`, which holds views) can't be compared with `assert_eq` (E0404). How do tests compare them?

**Outcome:** compare printed forms with a helper, `assert_same(actual, expected)` = `assert_eq("{actual}", "{expected}")`. Functions print as `<fun>`, so structure is checked fully. Which function a node holds is checked separately, by calling it (`f("x")` gives `Msg.Named("x")`).

**Rationale:** smallest change. The existing tests keep their shape, and the runtime stays as designed. Rejected: keeping `View` comparable through handler ids in a side table (a runtime redesign), and a hand-written `describe()` format (more code to keep in step with `View`).

**Copied to MEMORY.md:** yes

### DECISION-001 — Wait for the Lugha prerequisites, or start kivuli v1 now?

**Status:** resolved
**Raised:** 2026-10-17 — Session 1
**Resolved:** 2026-10-17 — Session 2

**Question:** Build struct update syntax and variants-as-functions in lugha first, or start kivuli without them?

**Outcome:** moot. Both landed in lugha before kivuli needed them: milestone 93 struct update (PR #32, tag m93) and milestone 94 variants as functions (PR #33, tag m94). Verified on a `lughac` reinstalled from lugha `main` at e7abc33.

**Rationale:** the brief's order (prerequisites first) held.

**Copied to MEMORY.md:** yes

### DECISION-004 — Where the extra GTK externs live long-term

**Status:** resolved
**Raised:** 2026-10-16 — kickoff brief §3
**Resolved:** 2026-10-17 — Session 2

**Question:** Keep kivuli's extra externs private, or contribute them to lugha-gtk?

**Outcome:** contribute them to lugha-gtk's `gtk.raw`. The user directed it. `gtk_box_remove` and `gtk_box_insert_child_after` were pushed to lugha-gtk `main` (1d20cae). Tagged `v0.1.1` (6303b64); kivuli uses them from `gtk.raw` and `src/gtk_extra.la` is gone.

**Rationale:** one set of GTK bindings, not two.

**Copied to MEMORY.md:** yes
