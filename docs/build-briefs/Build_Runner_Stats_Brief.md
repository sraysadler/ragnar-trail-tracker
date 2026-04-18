# Build Task: Runner Stats Modal + Footer + Settings Updates

## Objective
Three related changes in one pass:
1. Add a Runner Stats modal accessible from the main view footer
2. Move the Export button from the footer into the Settings panel
3. Make the backup folder name in Settings a tappable link to change folders

---

## Change 1 — Footer Updates

### Remove Export button
Remove the Export button from the footer entirely.

### Add Runner Stats button
In the footer where Export was (left side), add a "Runner Stats" button.
Style it as a blue-accented button to distinguish it from the Settings
button:
- Background: `#0a1a2a`
- Border: `1px solid #4a9eff`
- Text: `#4a9eff`, 9px
- Border-radius: 3px
- Padding: 3px 7px

Tapping it opens the Runner Stats modal (see Change 2 below).

**Updated footer layout:**
```
[ Runner Stats ]    ● Last saved: 2:23 PM    [ Settings ]
```

---

## Change 2 — Runner Stats Modal

### Overview
A full-screen overlay that scrolls as a whole page. Same overlay pattern
as the keypad and Settings panel. Opens when "Runner Stats" is tapped
in the footer. Closes via ✕ button in the header.

### Header
- Left: "Runner Stats" (same style as other panel headers)
- Right: "✕ Close" button

### Body
8 sections, one per runner, in race order (Runner 1 through Runner 8).
No collapse/expand — the page scrolls vertically.

A thin divider (`2px solid #2a2a2a`) separates each runner section.

---

### Runner Section Structure

Each runner section contains:

**1. Runner header (2 lines)**

Line 1 — Runner name:
- Font: 11px, white, font-weight 500

Line 2 — Summary stats:
- Format: `[N] of 3 complete · avg [pace]/mi`
- Example: `2 of 3 complete · avg 11:55/mi`
- Font: 9px, `#888`
- **If runner has 1+ completed legs:** regular weight, not italic
- **If runner has 0 completed legs:** italic, `#666`
  (signals the avg pace shown is the baseline estimate)

Header background: `#1a1a1a`
Padding: 7px 11px

**2. Column header row**
Small uppercase labels across all columns:
`[color bar] | Leg | Loop | Start | Finish | Dur | Pace`
Background: `#161616`
Font: 6px, `#555`, uppercase, letter-spacing

**3. Three leg rows** (one per leg for this runner)

Column layout: `4px color bar | leg# | loop name | start | finish | dur | pace`

**Color bar:** 4px wide, full row height, no border-radius:
- Green loop: `#1a6b1a`
- Yellow loop: `#6b6b00`
- Red loop: `#6b1a1a`

**Completed legs (actual finish entered):**
- All data: bold, white (`#fff`)
- Leg number, loop name, start, finish, duration, pace all bold

**Future/unstarted legs:**
- All data: italic, `#999`
- Shows estimated start, estimated finish, estimated duration,
  predicted pace

Row padding: 5px 11px
Row border-bottom: `1px solid #1a1a1a`

---

### Data to Display Per Leg Row

| Column | Completed leg | Unstarted leg |
|--------|--------------|---------------|
| Color bar | Loop color | Loop color |
| Leg | Leg number, bold | Leg number, muted |
| Loop | "Green"/"Yellow"/"Red", bold | Loop name, muted |
| Start | Actual start time, bold | Estimated start, italic |
| Finish | Actual finish time, bold | Estimated finish, italic |
| Dur | Actual duration (h:mm), bold | Estimated duration, italic |
| Pace | Actual pace (MM:SS/mi), bold | Predicted pace (MM:SS/mi), italic |

Duration format: `h:mm` (e.g. `0:50`, `1:18`)
Pace format: `MM:SS/mi` (e.g. `12:30/mi`)

---

### Summary Stats Calculation

**Legs completed:** count of legs where `actualFinish` is not null

**Average pace (for runners with completed legs):**
Simple average of actual pace values across all completed legs.
Format as MM:SS/mi.

**Average pace (for runners with no completed legs):**
Show config `avgPace` formatted as MM:SS/mi, italic.

---

### Runner Order
Display runners in race order: Runner 1 (Leg 1) through Runner 8 (Leg 8),
matching the order in config.

---

## Change 3 — Settings Panel Updates

### Export CSV button
Add an "Export CSV" button in the Backup Status section, below the
"Last backed up" row and above the Reset/Restore/Save footer buttons.

Style: full width of the Backup Status section, muted style:
- Background: `#222`
- Border: `1px solid #444`
- Text: `#aaa`, 9px
- Label: "⬆ Export CSV"
- Border-radius: 3px
- Padding: 6px

Tapping it calls the existing `downloadCSV()` / export function —
same behavior as the old footer Export button.

### Backup folder as tappable link
In the Backup Status section, the folder name is currently displayed
as plain text: `Backup folder: [name]`

Change the folder name portion to a tappable blue hyperlink style:
- Color: `#4a9eff`
- Text decoration: underline
- Tapping it re-opens the File System Access API folder picker
- On successful new folder selection: update IndexedDB with new handle,
  update the displayed folder name in Settings

If no folder is configured, keep the existing "Automatic backup not
configured" + "Set up" button behavior unchanged.

---

## Verification Steps

### Footer
1. Main view footer shows "Runner Stats" (blue) on left, "Settings"
   on right, no Export button
2. Tapping Runner Stats opens the modal

### Runner Stats modal
3. Modal opens full screen, header shows "Runner Stats" and ✕ Close
4. All 8 runner sections visible by scrolling
5. Each section has correct runner name, legs completed count,
   and avg pace
6. Runner with 0 completed legs: summary line is italic
7. Runner with 1+ completed legs: summary line is regular weight
8. Completed leg rows: all data bold white
9. Unstarted leg rows: all data italic gray
10. Color bars correctly reflect loop color for each leg row
11. Leg rows appear in correct race order within each runner section
12. Duration and pace formatted correctly (h:mm and MM:SS/mi)
13. ✕ Close returns to main view

### Settings Export button
14. "⬆ Export CSV" button visible in Backup Status section
15. Tapping it downloads a CSV file — same behavior as before

### Backup folder link
16. Folder name displays as blue underlined link
17. Tapping it opens the folder picker
18. Selecting a new folder updates the displayed name and future backups
    write to the new folder

---

## Key Constraints

- Runner Stats modal scrolls as a whole page — no internal scroll containers
- Color bars use the muted loop colors (#1a6b1a, #6b6b00, #6b1a1a) —
  not the full-brightness row colors from the main table
- Export functionality unchanged — same generateCSV() and downloadCSV()
  functions, just triggered from Settings instead of footer
- Folder picker uses existing File System Access API code — just call it
  from a new tap target
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Footer updated: Export removed, Runner Stats button added
- Runner Stats modal built and functional
- Export CSV button in Settings panel
- Backup folder name as tappable link to change folder
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
