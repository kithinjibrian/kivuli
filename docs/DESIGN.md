# Design System — kivuli

kivuli draws nothing. Every visual comes from the user's GTK theme (Adwaita by default). This file maps kivuli's style vocabulary onto GTK, and it is the only place those mappings are decided.

---

## Tokens: kivuli vocabulary → GTK

### Button styles (`.button_style(ButtonStyle.…)`)

| kivuli | GTK CSS class | Meaning |
|--------|---------------|---------|
| `Normal` | none | default action |
| `Primary` | `suggested-action` | the one main action in a view |
| `Danger` | `destructive-action` | deletes or discards |
| `Subtle` | `flat` | low-emphasis, toolbar-like |

### Text (`.size`, `.bold`, `.dim`)

| kivuli | GTK | Notes |
|--------|-----|-------|
| `.dim()` | CSS class `dim-label` | theme-defined opacity |
| `.bold()` | Pango weight bold (label attributes) | no CSS colour |
| `.size(n)` | Pango size `n` pt (label attributes) | the only numeric typography input. It comes from the app, never from kivuli |

`bold` and `size` are Pango attributes on the label (a fresh `PangoAttrList` each time, or none); `dim` is the `dim-label` CSS class (session 3, the user's agreement).

### Layout (`.padding`, `.spacing`, `.expand`, `.align`, `.width`, `.height`)

| kivuli | GTK |
|--------|-----|
| `.padding(n)` | `gtk_widget_set_margin_{top,bottom,start,end}(n)` |
| `.spacing(n)` | `gtk_box_set_spacing(n)` (column/row only) |
| `.expand()` | `gtk_widget_set_hexpand` / `vexpand(true)` (along the parent's axis) |
| `.align(Start\|Center\|End)` | `gtk_widget_set_halign` / `valign` (GTK_ALIGN_START/CENTER/END) |
| `.width(n)` / `.height(n)` | `gtk_widget_set_size_request` |

Units are pixels. All numbers come from the app. kivuli has **no default spacing or padding of its own**: a fresh `column` looks exactly like a fresh `GtkBox`.

### Icons and pictures (`Image`, `icon`, `image`, `.image`, `.fit`; PRPs 005, 010)

| kivuli | GTK |
| --- | --- |
| `icon(name)`, `image(Image.Icon(name))` | a `GtkImage` showing the theme's icon of that name; `.size(n)` sets its pixel size, else the theme's (`-1`) |
| `image(Image.File(path))`, `image(Image.Bytes(key, bytes))` | a `GtkPicture` showing a `GdkTexture` decoded from the bytes (a file is read first); `.fit` → `content-fit`; `.size(n)` an n × n size request |
| `.image(…)` / `.icon(name)` on a button | kivuli's own content: an image before a label, each hidden when empty; a texture or theme icon in its `GtkImage`, `.size(n)` its pixel size; classes `text-button` (label), `image-button` (image), both for both, as GTK sets them itself (its theme has no `image-text-button`) |

**Where images come from (DECISION-019's model B, session 11).** An image shows bytes it already has: a theme icon, a local file, or bytes in memory with a key. kivuli never fetches from the network; an app downloads with a non-blocking command (a later PRP) and shows the bytes. A view can't make kivuli open a connection.

**Hostile bytes.** Every file or bytes image is checked from its header in pure Lugha (`src/pictures.la`) before GTK decodes anything: PNG (signature, IHDR) or JPEG (SOI, the first start-of-frame), at most 4096 × 4096 px and 16 MB. A small file claiming a huge image never reaches the decoder. A decode error is freed and shows nothing; nothing panics. The diff compares an image's kind and name, path or key, never its bytes.

Icons are the theme's only. Prefer symbolic names (`…-symbolic`): they take the theme's text colour, so they follow light, dark and high-contrast themes. kivuli never recolours an icon or a picture, and ships no icon set: an app that wants another one (Font Awesome, Lucide …) brings it itself. A missing icon or file shows GTK's placeholder.

### Disabled

Disabled means no message (MEMORY decision 6), mapped to `gtk_widget_set_sensitive(false)`; the theme greys it out.

| kivuli | GTK |
| --- | --- |
| a button with `on_click == None` | the button insensitive |
| an input with no messages (`.disabled(true)`) | the entry insensitive: it still shows its value, the user can't type, and Enter sends nothing |
| a text area with no `on_change` | its editor (the text view) insensitive, not its scrolled window, so long text still scrolls |

---

## Component Rules

- A modifier that doesn't apply to a node is ignored silently (decision 5).
- A view's root carries `.window(title, w, h)`. Window size is the default size, not a fixed size.
- Correct:

      button("Delete", Msg.Delete).button_style(ButtonStyle.Danger)

- Wrong:

      // never reach for custom CSS colours, even for "danger"
      button("Delete", Msg.Delete).css("color: #c01c28")

---

### An app's own colours (`.style`, `.theme`; PRP-007, DECISION-012)

kivuli ships no colours. An app that paints its own frame gives its colours as typed looks: an enum of its own, a `Surface` per look (`background`, `text`, `border`, `radius`; unset fields stay the theme's), `.style([…])` on nodes and `.theme(theme(…))` on the root. kivuli generates the stylesheet: one class per property and value (`.kv-bg-242424ff`), merged per node in kivuli's code (a later look wins), loaded once for the display at application priority (600), when the window opens. Apps write no CSS and no selectors, so they can't depend on how kivuli or GTK build widgets. A background also clears the theme's background image, so it shows on themed widgets. kivuli removes only its own `kv-` classes from a widget.

### Overlay (`overlay`, `.place`; PRP-013)

| kivuli | GTK |
| --- | --- |
| `overlay(base, layers)` | a `GtkOverlay`: the base as its child, each layer added on top with `add_overlay`, in order |
| `.place(across, down)` | the layer's `halign` / `valign` (a layer is styled with its own axis, so `.align` and `.expand` don't apply) |

The overlay takes the base's size (`measure-overlay` off). A `GtkOverlay` only adds on top, so replacing a middle layer takes it and those above off (holding the ones above alive) and adds them back in order. Click-outside-to-close is a full-size layer the app adds, not a feature.

### Wrapping grid (`wrap`, `keyed_wrap`; PRP-012)

| kivuli | GTK |
| --- | --- |
| `wrap`, `keyed_wrap` | a `GtkFlowBox`, selection `NONE`; each child inside a `GtkFlowBoxChild` GTK makes, set not focusable so Tab goes straight to what's inside |
| `.spacing(n)` | row and column spacing |
| `.equal()` | `homogeneous` |
| `.max_per_line(n)` | `max-children-per-line`; without it, 65535 (GTK's own default caps a line at 7) |

A keyed move takes a child out and puts it back at its new place (a flow box can't reorder), holding the widget alive meanwhile. "Selected" tiles are the app's model plus a look.

### Header bar (`.header`)

| kivuli | GTK |
| --- | --- |
| `.header(start, title, end)` on the root | a `GtkHeaderBar` set as the window's titlebar, `show-title-buttons` on: GTK draws the window's controls |
| each `start` / `end` view | packed into the bar itself (`pack_start` / `pack_end`), so the theme's spacing between them applies |
| `title` | the bar's title widget; `.expand()` lets it take the bar's spare width |

**The header bar's look** (`.header_style([…])`, PRP-008): kivuli's classes go on the `headerbar` node only, so its window controls stay GTK's. GTK's Default theme and Yaru-dark both draw the line under a header bar as a 1 px bottom border (`border-width: 0 0 1px`), not a shadow, over a gradient background. A look with a `background` clears the gradient; `flat: Some(true)` sets `box-shadow: none; border-width: 0` (only `box-shadow: none` when the look has its own `border`), so a header bar and a rail with one flat look read as one frame, focused or not (kivuli's rules outrank the theme's in every state).

**Corner radius** (PRP-009): `radius` rounds all four corners, `radius_top_left` and the others one each. Four equal corners give `border-radius`, otherwise one `border-…-radius` per set corner; for every radius a theme uses, kivuli generates all five rules, so any merge of looks finds its classes. GTK 4.22 rounds a box's own background but does **not** clip its children to the curve (probed: the child is drawn outside any rounded clip unless the box's overflow is hidden). So kivuli sets `gtk_widget_set_overflow(HIDDEN)` on a node whose looks round a corner, marked with a `kv-clip` class so it can undo it; a widget GTK itself clips is left alone.

Dark mode: apps follow the desktop's preference. Forcing dark from an app is deferred (DECISION-017).

---

## Background work (`Cmd.task`, `Sub.receive`; PRP-011)

Lugha runs every green thread on the program's one OS thread, and GTK's main loop holds that thread inside one `extern` call (`g_application_run`). So kivuli hands Lugha's scheduler turns: while any task or `Sub.receive` is active, a GLib timeout (every 4 ms) calls `task.yield_now()`, which never blocks (it wakes due sleepers, polls sockets with `epoll_wait(…, 0)`, and runs ready threads one round, Lugha's preemption cutting a busy one off every 4,096 steps). It keeps yielding while yields find work, up to 2 ms of the 4 ms, then dispatches finished tasks' messages and received values on GTK's thread, one `update` at a time. With nothing in flight the source removes itself. An OS thread per task is impossible in Lugha (one OS thread; callbacks only on the thread that registered them).

Requests for the lugha session (not worked around): runtime hooks so GLib waits exactly instead of a timer (DECISION-021), and blocking calls off the one thread (DECISION-022).

---

## What Is Forbidden

- No hardcoded colours, fonts, or pixel values inside kivuli. Numbers reach GTK only from app-supplied modifier arguments, and colours only from an app's theme.
- No custom CSS stylesheet shipped by kivuli, other than what PRP-001 approves for `size`/`bold`, and the stylesheet generated from an app's own theme.
- No style that bypasses the theme (inline `background`, `color`).
