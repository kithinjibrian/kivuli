# Discovery Interview Protocol

Use when a feature request arrives without a PRP.

## When to Run a Discovery Interview

Any time a feature is described in one or two sentences without specifying:
- What the app author writes, and what appears on screen
- What messages it produces and when
- What the error and edge-case behavior should be
- Which existing files it touches
- What it must not touch
- Which open decisions in DECISIONS.md are relevant

## Question Sequence

Ask one question at a time. Do not batch questions. **Attach a recommended answer to every question**, with a short `.la` example where it helps. Wait for the answer before continuing.

Cover in order:
1. What does it do: the API the app author writes, and what appears on screen
2. Which messages it sends, when, and with what payload
3. What happens when it is misused: panics, `Result` errors, ignored modifiers
4. Edge cases: empty lists, rapid re-renders, focus and cursor, a node changing kind
5. Which existing files it reads from or writes to, and which new GTK externs it needs
6. What it must never modify
7. Which open entries in DECISIONS.md it depends on
8. Security: new externs, pinned closures, new dependencies
9. What rollback looks like if it is abandoned
10. How success is verified: headless tests, the windowless smoke run, and what the user should check on screen

## After the Interview

Write the completed PRP to `/PRPs/prp-{NNN}-{feature_name}.md` using the template.
Present it to the user.
Wait for explicit approval before writing any code.
