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
| Race Start Time | Format: hh:mm AM/PM. Default: 8:00 AM. Opens time entry keypad. |
| Runner 1–8 Names | Entered in running order — establishes leg assignments |
| Green Loop Miles | Default: 4.0. Opens numeric keypad. |
| Yellow Loop Miles | Default: 5.0. Opens numeric keypad. |
| Red Loop Miles | Default: 6.0. Opens numeric keypad. |
| Estimated Team Average Pace | Format: MM:SS per mile. Default: 12:00. Opens pace keypad. |
| Night Pace Adjustment | Percentage. Default: 5%. Opens numeric keypad. |
| Night Hours | Start and end time for the night pace window. Default: 8:00 PM – 7:00 AM. Each field opens time entry keypad. |

### Config Panel Controls

- **Save** — saves all config values to localStorage and closes panel
- **Reset to Defaults** — resets all fields to default values. Requires confirmation:
  "Reset all settings to defaults? Are you sure?"
- **Restore from Last Backup** — loads config and race data from the most recent
  automatic file backup. Requires confirmation before executing.

### Config Field Input Methods

All config fields are tappable — none use the system keyboard. Each field type
opens an appropriate custom keypad:

- **Time fields** (Race Start Time, Night Hours From/To): standard time entry
  keypad with AM/PM toggle
- **Distance fields** (Green/Yellow/Red Miles): numeric keypad with decimal
  point button instead of AM/PM
- **Pace field** (Avg Pace): numeric keypad without AM/PM; displays `/mi` suffix
- **Percentage field** (Night Adjustment): numeric keypad with decimal point
- **Runner name fields**: standard text input (system keyboard acceptable here)

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
- Folder handle for file backup: stored in `IndexedDB` (not localStorage —
  folder handles are not serializable to localStorage)
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
- Current time (label + value, right-aligned, updates every 60 seconds)

### Zone 2 — Status Bar (fixed, below header)
Always visible. Read-only race summary:
- Row 1: Projected Team Finish (full-width green highlighted bar)
  - Label changes to "FINAL TEAM FINISH" when Race Complete
- Row 2: Three equal columns — Current Loop | Next Runner | Next Start

### Zone 3 — Race Table (scrollable)
- Column headers fixed at the top of the scroll area — they do not scroll
- 24 rows, one per leg
- Tapping the Actual Finish cell opens the time entry keypad directly
- Tapping anywhere else on a row opens the Row Detail Modal
- Rows are visually distinct by loop color

### Zone 4 — Footer (bottom, fixed)
- Export Backup button (always visible)
- Settings button
- "Last saved: [time]" indicator with green dot after successful file backup

---

## Race Table — Visible Columns

| Column | Type | Notes |
|--------|------|-------|
| Leg # | Static | Pre-filled 1–24 |
| Runner | Display | From config; editable via Row Detail Modal |
| Loop | Static | G / Y / R |
| Start | Calculated | Previous leg's Finish value. Bold when leg has started. |
| Est Finish | Calculated | Start + Estimated Duration. Always italic `#999`. |
| Actual Finish | Tappable | Opens time entry keypad directly. Bold white when entered; dashed tap target when empty. |

---

## Row Detail Modal

Opened by tapping anywhere on a row except the Actual Finish cell.
Implemented as a **floating card centered over a dimmed background** —
not a full-screen panel. The race table is visible but dimmed behind it.
Tapping the dimmed background closes the modal.

### Display Fields (read-only)
Displayed in this order:

| Field | Notes |
|-------|-------|
| Runner | Custom dropdown — editable |
| Loop Miles | Read-only |
| Start | Bold white if confirmed |
| Est Finish | Always italic `#999` |
| Actual Finish | Blue tappable box — editable via keypad |
| Status | Color-coded: green / gray / blue |
| Est Duration | Italic `#999` — always estimated |
| Actual Duration | White if available, `—` if not |
| Predicted Pace | White |
| Actual Pace | White if available, `—` if not |

### Status Color Coding
- `Actual` → green (`#4caf50`)
- `Estimated (Baseline)` → gray (`#888`)
- `Estimated (Live)` → blue (`#4a9eff`)

### Editable Fields
- **Actual Finish** — blue tappable box. When empty, shows Est Finish time
  in italic as a starting suggestion. Tapping opens time entry keypad.
  Keypad saves immediately on confirm — does not wait for modal Save.
- **Runner** — custom dark-styled dropdown showing all 8 runner names.
  Currently selected runner highlighted in blue with checkmark.
  Changes save only when modal Save changes is tapped.

### Closing the Modal
- X button in header closes the modal
- Tapping dimmed background closes the modal
- If no changes: closes instantly
- If unsaved runner change: "Close without saving changes?" prompt

### Clearing an Actual Finish Time
- Backspacing all digits in keypad shows `--:-- AM` and changes Confirm
  to a red "Clear" button
- Tapping Clear removes the actual finish time and reverts leg to estimated

---

## Time Entry Keypad

Triggered by tapping the Actual Finish cell in the table, or the Actual
Finish field in the Row Detail Modal. Also used for time fields in Config.

### Layout

All buttons are equal size.

```
[  1  ] [  2  ] [  3  ]
[  4  ] [  5  ] [  6  ]
[  7  ] [  8  ] [  9  ]
[AM/PM] [  0  ] [  ⌫  ]
```

Time display sits above the keypad. Leg number and runner name shown
at top of keypad overlay for context.

### Display Behavior
- On open: shows current field value, or current device time if empty
- As digits typed: display rebuilds right-to-left (calculator style)
- Intermediate states (e.g. `--:65` while typing `6:54`) are expected
  and correct — digits resolve as entry completes
- AM/PM updates automatically when entered hour exceeds 12

### Clear Behavior
- When all digits backspaced from an existing entry: display shows
  `--:-- AM` and Confirm button changes to red "Clear"
- Tapping Clear removes the actual finish time
- Beginning to type new digits reverts Clear back to Confirm

### Saving / Cancelling
- Confirm saves and returns to previous context (table or modal)
- Cancel returns to previous context without saving

---

## Data Fields (Internal)

Tracked per leg. Not shown in the main table. Visible in the Row Detail
Modal and included in CSV exports.

| Field | Calculation |
|-------|------------|
| Loop Miles | Pulled from config based on loop color |
| Actual Duration | Actual Finish − Start |
| Actual Pace | Actual Duration ÷ Loop Miles |
| Predicted Pace | See Forecasting Logic |
| Estimated Duration | Predicted Pace × Loop Miles |
| Status | Actual / Estimated (Baseline) / Estimated (Live) |

---

## Core Behavior

### Start Times
- Leg 1 Start = Race Start Time from config
- Every subsequent Start = previous leg's Finish
- Start time is bold when the leg has officially begun

### Finish Values
- Actual Finish entered → Finish = Actual Finish
- No Actual Finish → Finish = Start + Estimated Duration

### Status Values
- **Actual** — Actual Finish has been entered for this leg
- **Estimated (Baseline)** — Runner has no completed legs; using config average pace
- **Estimated (Live)** — Runner has completed at least one leg; using runner's actual pace history

### Deleted Finish Time Behavior
If an actual finish time is deleted mid-race:
- That leg immediately reverts to an italic estimated finish time
- The next leg's start time adjusts to reflect the new estimated finish
- If the next leg has its own actual finish time, it re-anchors from there —
  impact is contained to the single gap
- If multiple consecutive actual finish times are deleted, estimated times
  cascade through the gap until the next leg with a real actual finish time
- The Race Complete state reverts automatically to in-progress if any actual
  finish is cleared

### Race Complete State
- Triggered automatically when all 24 legs have an Actual Finish entered
- Status bar Row 1 label changes from "PROJECTED TEAM FINISH" to
  "FINAL TEAM FINISH"
- Current Loop shows "Complete"
- Next Runner and Next Start show "—"
- If any Actual Finish is subsequently cleared, Race Complete reverts
  automatically to in-progress

---

## Forecasting Logic

All predictions use **pace per mile** as the core unit, allowing the model to
adapt cleanly to different loop distances.

### Night Pace Adjustment
A configurable pace adjustment is automatically applied to any leg whose estimated
start time falls within the configured night hours window (default: 8:00 PM – 7:00 AM).
The adjustment is applied to estimated duration after multiplying pace × loop miles.

- Format: percentage (e.g. 5%)
- Default: 5% (working value — to be validated against historical race data)
- Night hours window start and end times are configurable in the config panel
- Night window correctly handles crossing midnight

### Predicted Pace Formula

**Runner has completed 0 prior legs:**
- Use Config Average Pace
- → Status: Estimated (Baseline)

**Runner has completed 1 prior leg:**
- Use that runner's actual pace from their completed leg
- → Status: Estimated (Live)

**Runner has completed 2 or more prior legs:**
- Use simple equal-weight average of all runner's actual paces
- → Status: Estimated (Live)

*Note: No team loop-color average is used. The model relies solely on each
runner's own pace history, falling back to the config baseline only when
the runner has no completed legs. This simplification was chosen because
runner-specific history is more predictive than a small team average given
the 24-leg dataset.*

### Estimated Duration
```
Estimated Duration = Predicted Pace × Loop Miles
(× 1.05 if leg start falls within configured night hours, using default 5% adjustment)
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
- Optimized for night use and battery conservation

### Loop Row Colors
- Green loop: `#003300`
- Yellow loop: `#444400`
- Red loop: `#330000`

### Time Display
- Actual confirmed times: **bold white**
- Estimated times: *italic*, `#999` gray

### Status Bar and Footer
High contrast text — readable in daylight and at night

---

## Data Backup & Safety

### Primary Safety Net
- `localStorage` auto-saves after every single entry
- On app launch: if localStorage contains race data, it is restored silently
  and automatically — no prompt, no flash message
- "Last saved: [time]" in the footer is the permanent indicator of save state
- A restore prompt only appears if localStorage is empty but a file backup
  exists: "No saved data found. Restore from last file backup?"

### Automatic File Backup (Android + Chrome only)
- On first launch, a non-blocking banner prompts user to set up backup folder
- User taps "Set Up", selects or creates a folder (suggested: `RaceTracker/`
  in Downloads)
- Folder handle stored in IndexedDB for persistence across sessions
- After every Actual Finish entry, app silently writes a new CSV to that folder
- CSV filename format: `ragnar_tracker_YYYYMMDD_HHMM.csv`
  (sortable chronologically, new file per backup — never overwrites)
- Uses Chrome File System Access API
- Green dot appears next to "Last saved" in footer after successful backup
- "Last backed up: [date], [time]" shown in Settings panel

### Manual Export
- Export Backup button always visible in footer
- Generates downloadable CSV of all current race data on demand
- On iOS: a periodic in-app reminder prompt appears every 3 completed legs —
  "Back up race data? [Export Now] [Cancel]"

### CSV Export Format
Race data and race summary appear first. Config/settings appear at the bottom.

```
[Race Summary]
Team Name, Event, Race Start, Projected/Actual Finish, Total Duration, Status

[Race Data]
Leg, Runner, Loop, Loop Miles, Start, Est Finish, Actual Finish, Status,
Actual Duration, Actual Pace, Predicted Pace, Estimated Duration

[Config]
Team Name, Event Name, Race Start Time, Avg Pace, Green Miles, Yellow Miles,
Red Miles, Night Adjustment, Night Adjustment Type, Night Hours Start,
Night Hours End
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

1. ✓ HTML/CSS dark mode layout and touch interface
2. ✓ Config panel — all setup fields, defaults, reset, and restore controls
3. ✓ Race table with auto-generated leg assignments from config
4. ✓ Status bar and footer (projected finish, current leg, next runner, next
   start, last saved indicator)
5. ✓ Core calculation logic (start times, finish values, status)
6. ✓ Forecasting logic (simplified pace model + night pace adjustment)
7. ✓ Actual Finish direct tap from table → time entry keypad
8. ✓ Time entry keypad (equal-size buttons, calculator-style, 12/24-hour
   input, AM/PM toggle, confirm/cancel/clear)
9. ✓ Row Detail Modal (floating card, display + edit interface)
10. ✓ Runner substitution custom dropdown within modal
11. ✓ Deleted finish time behavior (clear mechanism in keypad)
12. ✓ Race Complete state and auto-revert
13. ✓ localStorage auto-save and silent restore on launch
14. → Config panel keypad integration (all tappable config fields wired
    to appropriate keypads)
15. File System Access API automatic backup
16. Manual CSV export with race summary and config sections
17. Visual design refinement (loop colors, bold/italic, dark mode polish)
18. UAT checklist and test plan document
19. On-device testing — full 24-leg simulation
20. Race day kiosk setup and Screen Pinning verification

---

## Open Items

- **Night Pace Adjustment** — default set to 5% as a working value. To be
  validated and refined after reviewing historical race data from past events.
- **Row Detail Modal future enhancements** — additional stats views, charts,
  or per-runner summaries are post-MVP considerations.
- **iOS support** — future consideration after Android MVP is complete.
  Backup strategy will rely on manual export to Files app and periodic
  in-app export reminders every 3 completed legs.

---

*Document version: 6.0 — Updated April 16, 2026*
*Based on 2025 Ragnar Trail Atlanta build and subsequent planning and build discussion*
