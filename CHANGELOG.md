# CHANGELOG — kivuli

Follows [Keep a Changelog](https://keepachangelog.com) format.
Updated at the end of every session when something is completed and merged.
Never deleted. Older entries are never modified.

---

## [Unreleased]

### Added
- `import kivuli;` reaches the whole public API (the package entry, `src/lib.la`).
- `View` with `text`, `button` and `column`: a screen described as plain data.
- `sandbox`: runs an app from `init`, `update` and `view` in a GTK window (rebuilt on every message for now).
- Messages now patch only the widgets whose view changed, instead of rebuilding the window.
- `examples/counter.la` and `examples/raw.la` (the same counter on raw GTK, for comparison).
- Project context system: CLAUDE.md, MEMORY.md, CONTEXT.md, DECISIONS.md, PRP templates, code style and design docs.
