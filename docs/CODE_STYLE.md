# Code Style — kivuli

Documentation rules for all `.la` code in this project.
Read this before writing any function, type, or module.

Formatting is not a style choice: `lughac fmt` decides it. Run it, and don't argue with it.

---

## The Two Layers of Documentation

### Layer 1 — Doc comments (what this is)

Every `public` function, method, struct, enum, interface and variant gets a `/** … */` doc comment. `lughac lsp` shows them on hover, completion and signature help. Package-level and private items get one only if their purpose isn't obvious.

**Format:**

    /**
     * [One sentence from the caller's point of view, starting with a verb.]
     *
     * [Optional: when to use it, what to watch out for.]
     * @param name [what it is: units (ms, px), allowed values]
     * @returns [what comes back; for a Result, the Ok value and every Err variant]
     *
     * Example:
     *     button("Add", Msg.Add).style(Primary)
     */

Rules:
- The first line is one sentence starting with the verb: "Builds", "Returns", "Starts". Never "This function…".
- `@param` for every parameter whose meaning isn't plain from its name and type. `@returns` on every public function that returns something.
- One example on every public API function and modifier (`ui` and `ui.testing`). App authors learn the API from hover.
- Short items take a one-line doc comment: `/** Ends the application once the current event is handled. */`
- Mention units every time: intervals are in **milliseconds**, sizes and spacing in **pixels**.

### Layer 2 — Inline comments (why this decision was made)

Inline `//` comments explain decisions, not code.

**Good:** `// Connected once per widget: the message comes from the Ref, so a re-render never reconnects (R4)`
**Bad:** `// Set the label text`

Rules:
- Put the comment above the line it explains.
- Comment when: a GTK call is used non-obviously, a C constant appears (`0` meaning `G_SOURCE_REMOVE`), a guard prevents a non-obvious bug (echoes from `set_text`), or a workaround exists for a lughac limitation.
- Every workaround for a lughac limitation names it: `// lughac: no struct update yet (DECISION-001), every field written out`.
- A TODO gets a DECISIONS.md entry or a date: `// TODO(DECISION-002): …`

---

## Module-Level Documentation

Every file starts with a doc comment:

    /**
     * [What this module is responsible for, and what it is NOT responsible for.]
     *
     * Depends on: [modules it imports]
     * Used by: [modules that import it; omit for a leaf]
     */

---

## What Good Documentation Looks Like

    /**
     * Tree diff: compares the last view with the next and lists the patches that
     * turn one into the other. Pure: never calls GTK, so it is tested headlessly.
     *
     * Depends on: ui
     * Used by: live
     */

    /**
     * Builds a button that sends `msg` when clicked.
     *
     * @param label the button's text
     * @param msg the message `update` receives on each click
     * @returns a button node; `.disabled(true)` removes its message and greys it out
     *
     * Example:
     *     button("Reset", Msg.Reset).disabled(m.count == 0)
     */
    public fun button<Msg>(label: string, msg: Msg): View<Msg> = …

---

## What Bad Documentation Looks Like — Do Not Write This

    // BAD: public with no doc comment, no units
    public fun after<Msg>(t: i64, msg: Msg): Cmd<Msg> = …

    // BAD: describes what, not why
    // loop over the children
    // compare each child

---

## Documentation Anti-Patterns

1. **Describing the code.** The code is the description. Comments explain what the code cannot.
2. **Stale comments.** Change the comment in the same edit as the code.
3. **TODOs without a reference.** Every TODO points to a DECISIONS.md entry or carries a date.
4. **Over-documenting private helpers.** Obvious helpers need no block.
5. **Under-documenting the error contract.** Every `@returns` on a `Result` lists the `Err` variants. Every function that panics on misuse says when.
