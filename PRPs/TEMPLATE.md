## FEATURE: [one sentence]

## OBJECTIVE
[2–3 sentences describing what "done" looks like from an app author's perspective, with a short `.la` example of the API in use]

## CONTEXT

- Starting state: [which files currently exist and are relevant]
- Ending state: [which files will be created or modified]
- Related existing code: [specific file paths to read before starting, including in ../lugha-gtk]
- Related source files: [docs/source/… notes that shaped this feature]
- Lugha spec sections relied on: [`lughac spec` sections; confirm each feature exists]
- GTK/GLib functions needed: [each checked against /usr/include/gtk-4.0; mark which are new externs]
- Open decisions that must be resolved first: [DECISIONS.md entries that block this feature]

## IMPLEMENTATION REQUIREMENTS

### Must Do
- [specific requirement]

### Must NOT Do
- [explicit exclusion, and why]

## ERROR HANDLING REQUIREMENTS

- [Which failures return `Result` and with which error enum variants]
- [Which conditions are programmer errors and panic, and the panic message]
- [What the caller receives on each failure path]

## SECURITY CONSIDERATIONS

- [New externs: each matches its C prototype exactly]
- [Closures handed to C: pinned with `data`/`release`, connected once per widget]
- [New package or `[link]` dependencies, and why]
- [External input handled by examples, and how it is validated]

## TESTS TO WRITE

List the specific test cases before any implementation begins. All must run with no display:
- [ ] Happy path: [describe]
- [ ] Error path: [each error variant]
- [ ] Edge case: [describe]
- [ ] Smoke (windowless, `test/smoke.la`): [what is built, read back, and quit]

## ROLLBACK PLAN

If this feature needs to be abandoned mid-implementation:
- Branch to return to: [branch name]
- State the codebase should be in: [describe]

## ACCEPTANCE CRITERIA
- [ ] [testable criterion]
- [ ] `lughac test test/ src/` passes
- [ ] Every example passes `lughac check` and `lughac build`
- [ ] `lughac fmt --check src examples test` is clean
- [ ] Every `public` item has a `/** … */` doc comment
- [ ] No file over 500 lines
- [ ] CHANGELOG.md updated

## VALIDATION
Run these commands to verify completion:
- `lughac fmt --check src examples test`
- `lughac test test/ src/`
- `for f in examples/*.la; do lughac build "$f" || exit 1; done`
- [feature-specific check]
- For the user: [what to open and click to confirm it visually]
