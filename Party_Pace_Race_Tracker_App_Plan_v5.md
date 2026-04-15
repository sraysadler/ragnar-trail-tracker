# Ragnar Trail Relay Race Tracker — App Plan

## Overview

A JavaScript application delivered as a single self-contained HTML file for tracking
a Ragnar Trail relay race in real time. The app requires no internet connection and
runs on a device that serves as a touchscreen kiosk at the race campsite.

The app is designed to be reusable across teams and race years. All team-specific
information is entered through a config panel before the race begins. The app is
built and optimized for Android + Chrome as the primary target platform.

---

## Race Format (Fixed)

This app is built for the standard Ragnar Trail relay format:
- 8 runners per team
- 3 loops per runner (Green, Yellow, Red)
- 24 total legs, run sequentially with no breaks between runners
- Loop order is always: Green → Yellow → Red, repeating
- Total race duration is typically 24–30 hours

These structural values are fixed in the app. Runner substitutions (due to injury,
absence, etc.) are handled during the race via the Row Detail Modal.

---

## Config Panel

Accessed via a Settings button visible in the header and footer. Should be completed
before the race begins. Until runner names are entered, the race table displays with
default values and blank runner name fields.

### Default Values (pre-filled before any config input)

- Race Start Time: 8:00 AM
- Estimated Team Average Pace: 12:00 per mile
- Green Loop Miles: 4.0
- Yellow Loop Miles: 5.0
- Red Loop Miles: 6.0
- Night Pace Adjustment: 5% (working value — pending validation against historical
  race data)
- Night Hours: 8:00 PM to 7:00 AM

### Required Config Fields

| Field | Notes |
|-------|-------|
| Team Name | Displayed in app header |
| Event Name | e.g. "Ragnar Trail Atlanta 2026" |
| Race Start Time | Format: hh:mm AM/PM. Default: 8:00 AM |
| Runner 1–8 Names | Entered in running order — establishes leg assignments |
| Green Loop Miles | Default: 4.0 |
| Yellow Loop Miles | Default: 5.0 |
| Red Loop Miles | Default: 6.0 |
| Estimated Team Average Pace | Format: MM:SS per mile. Default: 12:00. Used as baseline before race data exists |
| Night Pace Adjustment | Format: MM:SS per mile OR percentage. Default: 5%. Applied to legs whose estimated start falls within night hours |
| Night Hours | Start and end time for the night pace window. Default: 8:00 PM – 7:00 AM |

### Config Panel Controls

- **Save** — saves all config values to localStorage and closes panel
- **Reset to Defaults** — resets all fields to default values. Requires confirmation:
  "Reset all settings to defaults? Are you sure?"
- **Restore from Last Backup** — loads config and race data from the most recent
  automatic file backup. Requires confirmation before executing.

### Notes on Config

- All config values are stored in localStorage and survive browser restarts
- All config values are included in every CSV backup export
- Loop distances should be updated to actuals once confirmed at the race venue
- Night Pace Adjustment default of 5% is a working value — to be validated and
  refined after reviewing historical race data from past events

---

## Leg Assignment Table

Generated automatically from the runner order entered in config. R1–R8 are populated
from runner names entered in config. The 24-leg sequence follows the fixed Ragnar
format:

| Leg | Runner | Loop   |
|-----|--------|--------|
| 1   | [R1]   | Green  |
| 2   | [R2]   | Yellow |
| 3   | [R3]   | Red    |
| 4   | [R4]   | Green  |
| 5   | [R5]   | Yellow |
| 6   | [R6]   | Red    |
| 7   | [R7]   | Green  |
| 8   | [R8]   | Yellow |
| 9   | [R1]   | Red    |
| 10  | [R2]   | Green  |
| 11  | [R3]   | Yellow |
| 12  | [R4]   | Red    |
| 13  | [R5]   | Green  |
| 14  | [R6]   | Yellow |
| 15  | [R7]   | Red    |
| 16  | [R8]   | Green  |
| 17  | [R1]   | Yellow |
| 18  | [R2]   | Red    |
| 19  | [R3]   | Green  |
| 20  | [R4]   | Yellow |
| 21  | [R5]   | Red    |
| 22  | [R6]   | Green  |
| 23  | [R7]   | Yellow |
| 24  | [R8]   | Red    |

---

## Target Device

**Primary Build Target:** Android tablet, Chrome (latest)
**Orientation:** Portrait
**Connectivity:** Offline — app must function with zero network connection
**Power:** External battery pack recommended for extended race duration
**Kiosk Lock:** Android Screen Pinning

### Loading the App onto the Device

The HTML file can be loaded onto the device in several ways — it does not need to
happen immediately before leaving for the race, only before arrival at the venue:
- Downloaded directly in Chrome while on WiFi or mobile data
- Transferred via USB from a computer
- Copied from an SD card
- Shared via AirDrop, Bluetooth, or any file transfer method

Once loaded and opened in Chrome, no further internet connection is required.

### iOS (Future Consideration)

Not in scope for MVP. Key limitations on iOS:
- File System Access API not supported — automatic background file backup not possible
- localStorage subject to OS memory pressure clearing — cannot be fully prevented
- These limitations apply regardless of which browser is used on iOS, as all iOS
  browsers are required to use WebKit (including Chrome for iOS)
- iOS mitigation strategy: periodic in-app backup reminder (every 3 completed legs),
  plus manual Export button always visible. Exported CSV saved to the Files app
  and can be used to restore if needed.

---

## Technical Architecture

**Format:** JavaScript application delivered as a single self-contained `.html` file
- No server, no installation, no internet dependency
- All logic, styles, and data in one file

**Data Storage:**
- Primary: `localStorage` — auto-saves after every entry, survives browser restarts
- Automatic backup: File System Access API — silently writes updated CSV to a folder
  on the device after every Actual Finish entry. One-time permission granted on first
  launch. App suggests a default folder name (`RaceTracker/`) within Downloads.
  (Android + Chrome only)
- Manual export: Always-visible Export button for on-demand CSV download

**Languages:** HTML, CSS, JavaScript — no frameworks, no external dependencies

---

## Screen Layout (Portrait)

### Zone 1 — Header (top, fixed)
- Team Name / Event Name
- Settings button

### Zone 2 — Status Bar (fixed, below header)
Always visible. Read-only race summary:
- Projected Team Finish time
- Current Leg number
- Next Runner name
- Next Start Time

### Zone 3 — Race Table (scrollable)
- Column headers fixed at the top of the scroll area — they do not scroll
- 24 rows, one per leg
- Tapping the Actual Finish cell opens the time entry keypad directly
- Tapping anywhere else on a row opens the Row Detail Modal
- Rows are visually distinct by loop color

### Zone 4 — Footer (bottom, fixed)
- Export Backup button (always visible)
- Settings button
- "Last saved: [time]" indicator

---

## Race Table — Visible Columns

| Column | Type | Notes |
|--------|------|-------|
| Leg # | Static | Pre-filled 1–24 |
| Runner | Display | From config; editable via Row Detail Modal |
| Loop | Static | Green / Yellow / Red |
| Start | Calculated | Previous leg's Finish value |
| Est Finish | Calculated | Start + Estimated Duration |
| Actual Finish | Tappable | Opens time entry keypad directly. Bold when entered; italic and gray when estimated |

---

## Row Detail Modal

Opened by tapping anywhere on a row except the Actual Finish cell. Serves as the
detail view and secondary edit interface for that leg.

### Display Fields (read-only)
- Leg number, Runner, Loop, Loop Miles
- Start time
- Est Finish
- Actual Finish
- Status (Actual / Estimated Baseline / Estimated Live)
- Actual Duration
- Actual Pace
- Predicted Pace
- Estimated Duration

### Editable Fields
- **Actual Finish** — opens time entry keypad. Returns to modal on confirm/cancel.
- **Runner** — dropdown of all 8 team members; used for substitutions

### Closing the Modal
- X button in corner closes the modal
- If no changes were made: closes instantly, returns to table
- If unsaved changes exist: confirmation prompt — "Close without saving changes?"

### Clearing an Actual Finish Time
- Clearing a previously entered Actual Finish triggers a confirmation prompt:
  "Clear finish time for Leg [N]? It will revert to estimated."
- On confirm: leg reverts to italic estimated time; downstream legs re-anchor
  from the nearest remaining actual finish time

---

## Time Entry Keypad

Triggered by tapping the Actual Finish cell in the table, or the Actual Finish
field in the Row Detail Modal.

### Layout

All buttons are equal size. The bottom row follows the same width as the rows above.

```
[  1  ] [  2  ] [  3  ]
[  4  ] [  5  ] [  6  ]
[  7  ] [  8  ] [  9  ]
[AM/PM] [  0  ] [  X  ]
```

Time display sits above the keypad, showing the time being entered.

### Display Behavior
- On open: shows the current value of the field, formatted as hh:mm AM/PM
- If the field is empty: defaults to the current device time
- As digits are typed: display clears and rebuilds right-to-left (calculator style)
- AM/PM portion of the display updates automatically only when the entered value
  requires it (e.g. hour value exceeds 12)

### Entry Behavior
- Digits fill right-to-left as tapped
- Time is always displayed in 12-hour format (hh:mm AM/PM)
- Both 12-hour and 24-hour input are accepted:
  - 12-hour: user types digits and uses AM/PM toggle as needed
  - 24-hour: if the resulting hour value exceeds 12, the display automatically
    converts to 12-hour notation and flips AM/PM accordingly
  - Example: typing `2315` resolves and displays as `11:15 PM`
- AM/PM toggle: tapping the AM/PM button switches between AM and PM at any point
- If the entered hour value is 13 or higher, AM/PM is automatically set to PM
- If the entered hour value is exactly 12, AM/PM is set to PM
- Backspace (X): removes the last entered digit, shifting display back right

### Saving / Cancelling
- Confirm button saves the entry and returns to previous context
- Cancel returns to previous context without saving:
  - If opened from table → returns to table
  - If opened from modal → returns to modal

---

## Data Fields (Internal)

Tracked per leg. Not shown in the main table. Visible in the Row Detail Modal
and included in CSV exports.

| Field | Calculation |
|-------|------------|
| Loop Miles | Pulled from config based on loop color |
| Actual Duration | Actual Finish − Start |
| Actual Pace | Actual Duration ÷ Loop Miles |
| Predicted Pace | Weighted formula (see Forecasting Logic) |
| Estimated Duration | Predicted Pace × Loop Miles |
| Status | Actual / Estimated (Baseline) / Estimated (Live) |

---

## Core Behavior

### Start Times
- Leg 1 Start = Race Start Time from config
- Every subsequent Start = previous leg's Finish

### Finish Values
- Actual Finish entered → Finish = Actual Finish
- No Actual Finish → Finish = Start + Estimated Duration

### Status Values
- **Actual** — Actual Finish has been entered for this leg
- **Estimated (Baseline)** — Using config average pace; no real race data yet
- **Estimated (Live)** — Using actual race data from completed legs

### Deleted Finish Time Behavior
If an actual finish time is deleted mid-race:
- That leg immediately reverts to an italic estimated finish time
- The next leg's start time adjusts to reflect the new estimated finish
- If the next leg has its own actual finish time, it re-anchors from there —
  impact is contained to the single gap
- If multiple consecutive actual finish times are deleted, estimated times
  cascade through the gap until the next leg with a real actual finish time,
  which re-anchors everything downstream
- The Race Complete state reverts automatically to in-progress if any actual
  finish is cleared

### Race Complete State
- Triggered automatically when all 24 legs have an Actual Finish entered
- Status bar updates to show final race summary
- Footer displays option to export final race report CSV
- If any Actual Finish is subsequently cleared, Race Complete state reverts
  automatically to in-progress — no manual intervention required

---

## Forecasting Logic

All predictions use **pace per mile** as the core unit, allowing the model to
adapt cleanly to different loop distances.

### Night Pace Adjustment
A configurable pace adjustment is automatically applied to any leg whose estimated
start time falls within the configured night hours window (default: 8:00 PM – 7:00 AM).
The adjustment is additive to the predicted pace.

- Format: MM:SS per mile OR percentage
- Default: 5% (working value — to be validated against historical race data)
- Night hours window start and end times are configurable in the config panel

### Predicted Pace Formula

**Runner has completed 0 prior legs:**
- No team loop-color data exists → use Config Average Pace
  → Status: Estimated (Baseline)
- Team loop-color data exists → 70% Team Loop-Color Average + 30% Config Average Pace
  → Status: Estimated (Live)

**Runner has completed 1 prior leg:**
- Team loop-color data exists → 60% Runner Average + 40% Team Loop-Color Average
- No team loop-color data → 60% Runner Average + 40% Config Average Pace
  → Status: Estimated (Live)

**Runner has completed 2 or more prior legs:**
- Team loop-color data exists → 75% Runner Average + 25% Team Loop-Color Average
- No team loop-color data → 75% Runner Average + 25% Config Average Pace
  → Status: Estimated (Live)

*Note on pace variance across runners: the weighted model naturally self-corrects
for fast and slow runners after their first leg. A runner who completes their first
loop significantly faster or slower than the team average will immediately shift
their own predicted pace toward their actual performance, while still blending with
team data. No per-runner pace input is required in setup.*

### Estimated Duration
```
Estimated Duration = Predicted Pace × Loop Miles
(+ Night Pace Adjustment if leg start falls within configured night hours)
```

### Runner Substitutions
- Runner field for any leg is editable via the Row Detail Modal
- All forecasting uses whoever is currently shown as the runner for that leg
- Stats from substituted legs count toward the actual runner shown

---

## Visual Design

### Color Scheme — Dark Mode
- Background: pure black (`#000000`)
- Text: light / white
- Borders and highlights: lighter tones appropriate for dark background
- Loop row colors are visible against the black background
- Optimized for night use and battery conservation

### Loop Row Colors
- Green loop: `#003300`
- Yellow loop: `#333300`
- Red loop: `#330000`

### Time Display
- Actual confirmed times: **bold**
- Estimated times: *italic*, gray text

### Status Bar and Footer
High contrast text — readable in daylight and at night

---

## Data Backup & Safety

### Primary Safety Net
- `localStorage` auto-saves after every single entry
- On app launch: if localStorage contains race data, it is restored silently
  and automatically. A brief confirmation appears in the footer:
  "Race data restored — last saved [time]"
- A restore prompt only appears if localStorage is empty but a file backup
  exists: "No saved data found. Restore from last file backup?"

### Automatic File Backup (Android + Chrome only)
- On first launch, user grants one-time permission to a backup folder
- App suggests a default folder name (`RaceTracker/`) within Downloads
- After every Actual Finish entry, app silently writes an updated CSV to that folder
- CSV filename format: `ragnar_tracker_backup_YYYYMMDD_HHMM.csv`
- Uses Chrome File System Access API

### Manual Export
- Export Backup button always visible in footer
- Generates downloadable CSV of all current race data on demand
- On iOS: a periodic in-app reminder prompt appears every 3 completed legs —
  "Back up race data? [Export Now] [Cancel]"

### CSV Export Format
Race data and race summary appear first. Config/settings appear at the bottom.

```
[Race Summary]
Team Name, Event, Race Start, Projected/Actual Finish, Total Duration

[Race Data]
Leg, Runner, Loop, Loop Miles, Start, Est Finish, Actual Finish, Status,
Actual Duration, Actual Pace, Predicted Pace, Estimated Duration

[Config]
Team Name, Event Name, Race Start Time, Avg Pace, Green Miles, Yellow Miles,
Red Miles, Night Pace Adjustment, Night Hours
```

---

## Kiosk Setup (Race Day)

1. Load the HTML file onto the device before arriving at the venue
2. Open in Chrome — grant File System Access permission and confirm backup folder
3. Open config panel — enter all team and race details
4. Verify and update loop distances once confirmed at venue
5. Enable Android Screen Pinning
6. Plug into battery pack if needed
7. Place device at campsite handoff area

---

## Build Sequence

1. HTML/CSS dark mode layout and touch interface
2. Config panel — all setup fields, defaults, reset, and restore controls
3. Race table with auto-generated leg assignments from config
4. Status bar and footer (projected finish, current leg, next runner, next start,
   last saved indicator)
5. Core calculation logic (start times, finish values, status)
6. Forecasting logic (weighted pace model + night pace adjustment)
7. Actual Finish direct tap from table → time entry keypad
8. Time entry keypad (equal-size buttons, calculator-style, 12/24-hour input,
   AM/PM toggle, confirm/cancel)
9. Row Detail Modal (display + edit interface)
10. Runner substitution dropdown within modal
11. Deleted finish time behavior and confirmation prompt
12. Race Complete state and auto-revert
13. localStorage auto-save and silent restore on launch
14. File System Access API automatic backup
15. Manual CSV export with race summary and config sections
16. Visual design refinement (loop colors, bold/italic, dark mode polish)
17. UAT checklist and test plan document
18. On-device testing — full 24-leg simulation
19. Race day kiosk setup and Screen Pinning verification

---

## Open Items

- **Night Pace Adjustment** — default set to 5% as a working value. To be validated
  and refined after reviewing historical race data. Needs to be confirmed before
  build is finalized.
- **Row Detail Modal future enhancements** — additional stats views, charts, or
  per-runner summaries are post-MVP considerations.
- **iOS support** — future consideration after Android MVP is complete. Backup
  strategy will rely on manual export to Files app and periodic in-app export
  reminders every 3 completed legs.

---

*Document version: 5.0 — Updated April 14, 2026, late evening*
*Based on 2025 Ragnar Trail Atlanta build and subsequent planning discussion*
