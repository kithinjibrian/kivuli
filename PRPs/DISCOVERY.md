# Discovery

Use when a feature request arrives without a PRP.

## Research first

Answer everything you can yourself, and cite what you checked: the code it touches, `lughac spec`, the GTK headers in `/usr/include/gtk-4.0` and GTK's own source when behaviour matters, lugha-gtk's bindings, DECISIONS.md, and the request itself (Rapidkoi's `docs/specs/kivuli-requests.md`). Cover:

1. The API the app author writes, and what appears on screen
2. Which messages it sends, when, and with what payload
3. Misuse: panics, `Result` errors, ignored modifiers
4. Edge cases: empty values, rapid re-renders, focus and cursor, a node changing kind
5. The files it touches, new GTK externs, and files near the 500-line limit
6. What it must never change
7. Open entries in DECISIONS.md it depends on
8. Security: new externs, pinned closures, new dependencies
9. How it's tested headlessly, and what the user checks by eye

## Then ask only the real decisions

Write the PRP to `PRPs/prp-{NNN}-{feature_name}.md` from the template, and present it with only the questions that truly need the user: choices with real trade-offs, permissions (editing or pushing another repository), scope. Each comes with a recommendation. Everything with a clear answer is settled in the PRP, not asked.

Build only after explicit approval.
