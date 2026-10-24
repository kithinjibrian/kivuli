## FEATURE: [one sentence]

## OBJECTIVE
[What "done" looks like for an app author: what they write, and what appears on screen. A short `.la` example of the API in use.]

## CONTEXT
- Files to read first, and the files this creates or changes.
- Specs and sibling APIs relied on (`lughac spec`, `../lugha-gtk/`, GTK headers in `/usr/include/gtk-4.0`): confirm each exists.
- Where the request comes from (a Rapidkoi DECISION number, if any).
- Decisions it depends on (DECISIONS.md): none may be open.

## REQUIREMENTS

### Must Do
- [specific requirement]

### Must NOT Do
- [exclusion, and why]

## DESIGN
[Only what isn't obvious: new nodes, modifiers and patches, error enums and panics, new externs (each checked against its header), pinned closures, new dependencies.]

## TESTS
- [ ] [pure, no display: happy path, each error, edge cases]
- [ ] [GTK, skipped without a display: what is built, patched and read back]

## ACCEPTANCE
- [ ] [testable criterion]
- [ ] `lughac test test/`, the examples' tests and `lughac fmt --check src examples test` pass; every example builds
- [ ] CHANGELOG.md updated
- [ ] For the user, by eye: [what to open and try]
