# Build Task: Step 2 — Config Panel

## Objective
Add the Settings/Config panel to `ragnar-trail-tracker.html`. This is a
full-screen panel that replaces the main view when the Settings button is
tapped. It is not a modal or overlay — it takes over the entire screen.
The panel scrolls as a whole page if content exceeds the screen height.
No JavaScript functionality yet — layout and styling only.

---

## Behavior (visual only at this stage)
- Tapping Settings in the footer shows the Config panel (full screen)
- Tapping ✕ Close returns to the main view
- For this step, toggling between views can be done with a simple
  JavaScript show/hide on the two containers — this is the one exception
  to the no-JS rule for this step, purely to allow visual evaluation

---

## Layout

### Header (fixed at top)
- Left: "Settings" in the same style as the main app header title
- Right: "✕ Close" button — same style as other small buttons in the app
- Background: `#1a1a1a`, border-bottom: `1px solid #333`

### Body (scrollable as full page)
Grouped sections with a small uppercase muted section label above each group.
All content flows naturally — no internal scroll container.

### Footer (fixed at bottom)
Three equal-width buttons across the full footer width:
- **Reset** — muted style (dark background, gray text, gray border)
- **Restore** — blue accent style (dark background, `#4a9eff` text and border)
- **Save** — green accent style (`#1a3a1a` background, `#4caf50` text and border)

---

## Field Sections

### Race Info
Three fields: Team name, Event name, Start time

### Runners (in order)
Label: "Runners (in order)"
Eight runner name fields displayed in a 2-column grid.
Each field shows a number prefix (1–8) in muted gray, followed by a text
input area. Style: dark background (`#2a2a2a`), subtle border (`#333`),
rounded corners. These are text inputs, not tappable keypad fields.

### Loop Distances
Three fields: Green, Yellow, Red

### Pace Settings
Two fields: Avg pace, Night adjustment

### Night Hours
One row with a "From" label, two tappable time fields, and a "–" separator
between them:
`From  [ 8:00 PM ]  –  [ 7:00 AM ]`

---

## Field Styles

### Tappable fields (all fields except runner name inputs)
These will eventually open a custom keypad. Style them to look like
interactive inputs:
- Background: `#0a1a2a`
- Border: `1px solid #4a9eff`
- Border radius: 3px
- Text: `#ccc`, right-aligned
- Width: consistent across all fields (same width for every tappable field)
- Padding: comfortable for touch

Fields using this style:
- Team name
- Event name
- Start time
- Green / Yellow / Red loop distances
- Avg pace
- Night adjustment
- Both Night hours fields

### Runner name inputs
Plain text input style:
- Background: `#2a2a2a`
- Border: `1px solid #333`
- Border radius: 3px
- Text: `#ccc`
- Number prefix: `#444`, small

---

## Default Values (placeholder content for visual evaluation)

| Field | Value |
|-------|-------|
| Team name | Party Pace |
| Event name | Ragnar Trail Atlanta 2026 |
| Start time | 8:00 AM |
| Runner 1 | Wendy |
| Runner 2 | Kristine |
| Runner 3 | Scott |
| Runner 4 | Jeff |
| Runner 5 | Jason |
| Runner 6 | Ray |
| Runner 7 | Jenny |
| Runner 8 | Tim |
| Green miles | 4.0 mi |
| Yellow miles | 5.0 mi |
| Red miles | 6.0 mi |
| Avg pace | 12:00 /mi |
| Night adjustment | 5% |
| Night hours from | 8:00 PM |
| Night hours to | 7:00 AM |

---

## Key Constraints

- No internal scroll containers — the whole page scrolls naturally
- Full screen panel, not a floating overlay or modal
- Single HTML file — all styles added to the existing `<style>` block
- No additional JavaScript beyond the simple show/hide toggle needed
  to evaluate both views
- All tappable fields must be the same width
- Do not use any external libraries or frameworks

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Config panel visible when Settings is tapped
- Main view restored when ✕ Close is tapped
- All fields styled correctly with default placeholder values
- Consistent field widths across all tappable inputs
- Footer buttons correctly styled (Reset / Restore / Save)

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
