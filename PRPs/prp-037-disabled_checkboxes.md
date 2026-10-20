# PRP-037 — Disabled check boxes

Status: built (session 73) on branch prp-037-disabled-checkboxes
Discovery: found in PRP-036's build (session 71): `.disabled` skips check boxes. The user asked for it next (session 72). Architecture rule 6 decides the design: disabled means no message. *(proposed)*: for you to confirm or change.

## FEATURE: `.disabled(true)` on a check box (native or `ui.checkbox`): it can't be toggled, GTK greys it, and `kivuli.testing.toggle` says it's silent.

## OBJECTIVE

```
checkbox("Notifications", m.on, Msg.On).disabled(!m.signed_in)
ui.checkbox("Accept terms", m.ok, Msg.Ok).disabled(m.sending)
assert_eq(kivuli.testing.toggle(view(m), "Accept terms"), Err(Lookup.Silent("check box \"Accept terms\"")));
```

## REQUIREMENTS *(proposed: rule 6, as buttons, fields and text areas do it)*
- **The view**: `View.Checkbox(string, bool, Option<fun(bool): Msg>)`, the toggle function `None` when disabled. `checkbox(…)` gives `Some`; `.disabled(true)` (`mapping.disabled_node`) gives `None`; `map` maps it.
- **Built**: `gtk_widget_set_sensitive(check, false)` when `None`; the `toggled` handler reads the `Ref` and sends only through `Some` (connected once, rule 5).
- **Diffed and patched**: `SetOnToggle(path, Option<…>)` sets the `Ref` and the sensitivity, as a button's message does; the check box keeps its widget.
- **`kivuli.testing.toggle`**: `Err(Lookup.Silent("check box \"…\""))` when disabled; `Lookup.Silent`'s doc lists check boxes.
- **`ui.checkbox`**: its disabled look (half opacity) now shows; `examples/components.la`'s Checkbox section gains a disabled one.

### Must NOT Do
- No separate `disabled` flag (rule 6); no change to how a checked state is shown.

## TESTS (narrow; type-check now, run once the user's faster tests are in)
- [x] `test/lib.la` or `test/map.la`: `.disabled(true)` on a check box (and a styled one) takes its toggle away; `map` keeps `None`.
- [x] `test/diff.la` (or a new `test/diff_checkbox.la` if diff.la is full): enabled → disabled → enabled gives `SetOnToggle` with `None`, then `Some`.
- [x] `test/testing.la`: `toggle` on a disabled check box is `Silent`.
- [x] `test/live_disabled.la`: a disabled check box isn't sensitive and sends nothing when toggled; enabled again, it sends.
- [x] `examples/components.la` built, and every example that uses `checkbox` type-checks.

## ACCEPTANCE
- [ ] fmt clean; no file over 500 lines; README, CHANGELOG; merged, tagged; pushed only on your go.

## NOTES FROM THE BUILD

- **Where the tests went**: map.la (the modifier, a styled one, `map` keeping `None`), a new diff_checkbox.la (diff.la is near the limit; its two check box expectations now carry `Some`), testing.la, live_disabled.la. All type-check; none run (the user's call).
