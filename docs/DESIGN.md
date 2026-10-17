# Design System — kivuli

kivuli draws nothing. Every visual comes from the user's GTK theme (Adwaita by default). This file maps kivuli's style vocabulary onto GTK, and it is the only place those mappings are decided.

---

## Tokens: kivuli vocabulary → GTK

### Button styles (`.style(…)`)

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

### Disabled

`on_click == None` maps to `gtk_widget_set_sensitive(false)`. The theme greys it out.

---

## Component Rules

- A modifier that doesn't apply to a node is ignored silently (decision 5).
- A view's root carries `.window(title, w, h)`. Window size is the default size, not a fixed size.
- Correct:

      button("Delete", Msg.Delete).style(Danger)

- Wrong:

      // never reach for custom CSS colours, even for "danger"
      button("Delete", Msg.Delete).css("color: #c01c28")

---

## What Is Forbidden

- No hardcoded colours, fonts, or pixel values inside kivuli. Numbers reach GTK only from app-supplied modifier arguments.
- No custom CSS stylesheet shipped by kivuli, other than what PRP-001 approves for `size`/`bold`.
- No style that bypasses the theme (inline `background`, `color`).
