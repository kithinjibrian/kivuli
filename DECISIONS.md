# DECISIONS.md — kivuli

Tracks architectural and design questions that are open, deferred, or resolved.

Rules:
- Every open decision blocks implementation of the code it affects.
- The AI must not implement anything that depends on an open decision.
- When a decision is resolved, move it to the RESOLVED section and record the outcome.
- Once resolved, copy the outcome to MEMORY.md as an architectural decision.

---

## OPEN — Requires human input before implementation

### DECISION-001 — Wait for the Lugha prerequisites, or start kivuli v1 now?

**Status:** open
**Raised:** 2026-10-17 — Session 1
**Resolved by:** human
**Blocks:** starting PRP-001's build (not its discovery)

**Question:** The kickoff brief puts struct update syntax and variants-as-functions first, as lugha milestones 91–92. lugha has since used 91 for green threads and 92 for actors (being built in lugha session 214). Neither prerequisite exists. Do we build them in lugha first, or start kivuli now?

**Options:**
- A) Build both in lugha first, after actors (milestones 93–94), as the brief orders. Examples and README use the nice syntax from day one. Cost: kivuli waits.
- B) Start kivuli v1 now, writing every struct field and `fun(s) = Msg.Named(s)`, and switch the examples over when the features land. The brief says v1 "can be started without them". Cost: examples are rewritten later.
- C) Run discovery for PRP-001 now (no code), and decide A or B once actors are done.

**Notes:** Recommended: C, then A. Discovery costs nothing and doesn't depend on syntax. The brief's stated order is A.

---

### DECISION-002 — The shape of `lazy` (risk R1)

**Status:** open
**Raised:** 2026-10-16 — kickoff brief §6
**Resolved by:** human, after a technical check of `lughac spec`
**Blocks:** step 6 of kivuli v1 (`lazy`)

**Question:** `lazy(input: T, f)` must keep `input` in a `View<Msg>` node to compare next render, but `View` is generic only over `Msg`. How do we store and compare the input?

**Options:**
- A) Store a closure `fun(): bool` that compares against a captured previous value. Awkward, because it needs the previous value captured.
- B) A cheap key supplied by the app: `lazy(key: string, f: fun(): View<Msg>)` / `lazy_i64(key: i64, …)` (Elm's keyed discipline).
- C) An `Eq`-bounded interface value, if Lugha allows `Eq` as an interface type (check `lughac spec`).

**Notes:** Brief: prefer the simplest that works. B needs no language support.

---

### DECISION-003 — Module path of the testing helpers

**Status:** open
**Raised:** 2026-10-16 — kickoff brief §7
**Resolved by:** human
**Blocks:** step 7 of kivuli v1 (`ui.testing`)

**Question:** Decision 10 says `ui.testing`, but `src/testing.la` gives `import kivuli.testing;`.

**Options:**
- A) Re-export from `ui.la` (`public import testing;`), so apps write `ui.testing.click(…)`. Check that the spec allows this re-export form.
- B) Name it `kivuli.testing` and amend decision 10.

**Notes:** A matches the agreed API. B is simpler if re-export of a whole module doesn't behave as needed.

---

### DECISION-004 — Where the extra GTK externs live long-term

**Status:** open
**Raised:** 2026-10-16 — kickoff brief §3
**Resolved by:** human
**Blocks:** nothing in v1 (they start in `src/gtk_extra.la`). It blocks moving them.

**Question:** Keep kivuli's extra externs private, or contribute them to lugha-gtk 0.2?

**Options:**
- A) Private in `src/gtk_extra.la` for v1, then contribute after v1 ships.
- B) Contribute to lugha-gtk 0.2 first, and kivuli depends on 0.2.

**Notes:** A keeps kivuli unblocked. Contributing is an outward-facing action in another repo.

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

(none yet. Decisions agreed before this repository existed are in MEMORY.md.)
