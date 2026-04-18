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

Accessed via a Settings button in the footer. On first load with no saved config,
the Settings panel opens automatically with the title "Race Setup" to prompt the
user to configure the race. Once config is saved, subsequent opens show "Settings".

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
| Green Loop Miles | Default: 4.0. Opens numeric keypad (implied decimal). |
| Yellow Loop Miles | Default: 5.0. Opens numeric keypad (implied decimal). |
| Red Loop Miles | Default: 6.0. Opens numeric keypad (implied decimal). |
| Estimated Team Average Pace | Format: MM:SS per mile. Default: 12:00. Opens pace keypad. |
| Night Pace Adjustment | Percentage. Default: 5%. Opens numeric keypad (implied decimal). |
| Night Hours | Start and end time for the night pace window. Default: 8:00 PM – 7:00 AM. Each field opens time entry keypad. |

### Config Panel Controls

- **Save** — saves all config values to localStorage and closes panel
- **Reset to Defaults** — resets all fields to default values. Requires confirmation.
- **Restore from Last Backup** — loads config and race data from the most recent
  automatic file backup. Requires confirmation before executing. Shows footer
  confirmation message after restore.
- **Export CSV** — downloads a CSV of all current race data (same as manual export).
  Located in the Backup Status section of the Settings panel.

### Config Field Input Methods

All config fields are tappable — none use the system keyboard for numeric/time input:

- **Time fields** (Race Start Time, Night Hours From/To): standard time entry
  keypad with AM/PM toggle
- **Distance fields** (Green/Yellow/Red Miles): numeric keypad with implied decimal
  (one place from right — type `4`, `6` to get `4.6`)
- **Pace field** (Avg Pace): pace keypad (MM:SS format, bottom-left blank)
- **Percentage field** (Night Adjustment): numeric keypad with implied decimal
- **Runner name fields** and **Team/Event name fields**: system keyboard (text input)

### Backup Status Section

Displayed at the bottom of the Settings panel, above the Reset/Restore/Save buttons:
- **Backup folder** — shows folder name as a tappable blue hyperlink. Tapping
  re-opens the folder picker to change the backup folder.
- **Last backed up** — timestamp of most recent successful backup write
- **Export CSV** — full-width button to trigger manual CSV download

### Notes on Config

- All config values are stored in localStorage and survive browser restarts
- All config values are included in every CSV backup export
- Loop distances should be updated to actuals once confirmed at the race venue
- Night Pace Adjustment default of 5% is a working value — to be validated

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

The app is also hosted on GitHub Pages:
`https://sraysadler.github.io/ragnar-trail-tracker/ragnar-trail-tracker.html`

### iOS (Future Consideration)

Not in scope for MVP. Key limitations on iOS:
- File System Access API not supported — automatic background file backup not possible
- localStorage subject to OS memory pressure clearing — cannot be fully prevented
- iOS mitigation strategy: periodic in-app backup reminder, plus manual Export
  button always visible in Settings panel.

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
  launch. (Android + Chrome only)
- Manual export: Export CSV button in the Settings panel

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
- **Runner Stats** button (left) — opens the Runner Stats modal
- **"Last saved: [time]"** indicator with green dot after successful file backup (center)
- **Settings** button (right)

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

### Display Fields
Displayed in this order:

| Field | Notes |
|-------|-------|
| Runner | Custom dropdown — editable |
| Loop Miles | Dotted border — press and hold 3 seconds to override (completed legs only) |
| Start | Bold white if confirmed |
| Est Finish | Always italic `#999` |
| Actual Finish | Blue tappable box — editable via keypad |
| Status | Color-coded: green / gray / blue |
| Est Duration | Italic `#999` — always estimated |
| Actual Duration | White if available, `—` if not |
| Predicted Pace | White |
| Actual Pace | White if available, `—` if not |

### Loop Miles Override
- The Loop Miles field always shows a dotted border indicating it is interactable
- **Press and hold for 3 seconds** (completed legs only) opens the numeric keypad
  pre-populated with the current effective miles
- Entering a new value overrides the config default for that leg only
- Overridden miles display in green (`#4caf50`) to indicate the value has been changed
- To reset: press and hold again and enter the config default value
- Override is only available on legs where `actualFinish` is set
- All calculations for that leg use the override miles via `getEffectiveMiles()`

### Status Color Coding
- `Actual` → green (`#4caf50`)
- `Estimated (Baseline)` → gray (`#888`)
- `Estimated (Live)` → blue (`#4a9eff`)

### Editable Fields
- **Actual Finish** — blue tappable box. Tapping opens time entry keypad.
  Keypad saves immediately on confirm — does not wait for modal Save.
- **Runner** — custom dark-styled dropdown showing all 8 runner names.
  Currently selected runner highlighted in blue with checkmark.
  Changes save only when modal Save changes is tapped.
- **Loop Miles** — press and hold 3 seconds to open numeric keypad (completed
  legs only). Override stored as `overrideMiles` in leg data.

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

## Runner Stats Modal

Opened by tapping the "Runner Stats" button in the footer.
Full-screen scrollable overlay. Header shows "Runner Stats" with ✕ Close button.

### Structure
8 sections, one per runner in race order. No collapse/expand — scroll to see all.
A Team Totals section appears at the bottom.

### Runner Section
Each section contains:

**Header (2 lines):**
- Line 1: Runner name (bold, white, 11px)
- Line 2: `[N] of 3 complete · avg [pace]/mi`
  - Runner with 1+ completed legs: regular weight, `#888`
  - Runner with 0 completed legs: italic, `#666` (signals baseline estimate)

**Column header row:** Leg | Loop | Start | Finish | Dur | Pace

**Three leg rows**, one per leg assigned to this runner:
- Color bar on left edge (4px, muted loop color)
- Completed legs: bold white data
- Unstarted legs: italic `#999` data showing estimates
- Pace column shows `MM:SS/mi*` asterisk if loop miles were overridden for that leg

### Runner Grouping
All legs are grouped by the **current assigned runner** (`runner` field), not the
original default runner (`defaultRunner`). If a runner takes over another runner's
legs via substitution, all those legs appear under the substituting runner's section.

### Team Totals Section
Displayed at the bottom with `#1a1a1a` background:
- **Legs complete** — total count of actual finish times entered (out of 24)
- **Total run time** — sum of all actual durations (h:mm format)
- **Team avg pace** — average of all runners' actual paces (MM:SS/mi format)
- If no legs completed: show `0` for legs complete, `—` for time and pace

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

### Keypad Variants

**Variant A — Time keypad:** AM/PM toggle in bottom-left. Used for Actual Finish,
Race Start Time, Night Hours.

**Variant B — Numeric keypad (implied decimal):** Bottom-left blank. Digits fill
right-to-left with one implied decimal place. Type `4`, `6` to get `4.6`. Used
for loop miles (config and loop miles override) and night adjustment.

**Variant C — Pace keypad:** Bottom-left blank. MM:SS format. Used for Avg Pace.

### Display Behavior
- On open: shows current field value, or current device time if empty
- As digits typed: display rebuilds right-to-left (calculator style)
- AM/PM updates automatically when entered hour exceeds 12

### Clear Behavior
- When all digits backspaced from an existing entry: display shows
  `--:-- AM` and Confirm button changes to red "Clear"
- Tapping Clear removes the actual finish time
- Beginning to type new digits reverts Clear back to Confirm

### Saving / Cancelling
- Confirm saves and returns to previous context (table, modal, or config)
- Cancel returns to previous context without saving

---

## Data Fields (Internal)

Tracked per leg. Not shown in the main table. Visible in the Row Detail
Modal and included in CSV exports.

| Field | Calculation |
|-------|------------|
| Loop Miles | `getEffectiveMiles()` — returns `overrideMiles` if set, else config default |
| Actual Duration | Actual Finish − Start |
| Actual Pace | Actual Duration ÷ `getEffectiveMiles()` |
| Predicted Pace | See Forecasting Logic |
| Estimated Duration | Predicted Pace × `getEffectiveMiles()` |
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
- If the next leg has its own actual finish time, it re-anchors from there
- The Race Complete state reverts automatically to in-progress if any actual
  finish is cleared

### Race Complete State
- Triggered automatically when all 24 legs have an Actual Finish entered
- Status bar Row 1 label changes to "FINAL TEAM FINISH"
- Current Loop shows "Complete"
- Next Runner and Next Start show "—"
- Reverts automatically if any actual finish is cleared

---

## Forecasting Logic

All predictions use **pace per mile** as the core unit.

### Night Pace Adjustment
Applied as a **pace penalty** to any leg whose estimated start time falls within
the configured night hours window (default: 8:00 PM – 7:00 AM).

- Format: percentage (e.g. 5%)
- Default: 5% (working value — to be validated against historical race data)
- Applied to predicted pace, not duration
- Night window correctly handles crossing midnight

### Pace Normalization
All pace history is stored as **normalized day-equivalent pace**:
- Night leg actual pace is normalized before storage: `actualPace / (1 + nightAdjustment/100)`
- This allows day and night leg paces to be averaged accurately
- When predicting a night leg, normalized pace is re-inflated: `normalizedPace × (1 + nightAdjustment/100)`

### Predicted Pace Formula

**Runner has completed 0 prior legs:**
- Use Config Average Pace (treated as day pace)
- → Status: Estimated (Baseline)

**Runner has completed 1 prior leg:**
- Use that runner's normalized actual pace from their completed leg
- → Status: Estimated (Live)

**Runner has completed 2 or more prior legs:**
- Use simple equal-weight average of all runner's normalized actual paces
- → Status: Estimated (Live)

### Effective Miles
All pace and duration calculations use `getEffectiveMiles()` which returns
`overrideMiles` if set for that leg, or the config loop default otherwise.
This ensures loop miles overrides flow through all calculations automatically.

### Estimated Duration
```
Estimated Duration = Predicted Pace × getEffectiveMiles()
```
Night adjustment is already baked into Predicted Pace — no additional multiplier.

### Runner Substitutions
- Runner field for any leg is editable via the Row Detail Modal
- All forecasting uses the current `runner` field — not `defaultRunner`
- Pace history is attributed to whoever is currently assigned as `runner`
- If a runner takes over multiple legs (e.g. due to injury), all their completed
  legs contribute to their pace history regardless of who was originally scheduled

---

## Visual Design

### Color Scheme — Dark Mode
- Background: pure black (`#000000`)
- Text: light / white
- Optimized for night use and battery conservation

### Loop Row Colors
- Green loop: `#003300`
- Yellow loop: `#444400`
- Red loop: `#330000`

### Time Display
- Actual confirmed times: **bold white**
- Estimated times: *italic*, `#999` gray

### Loop Miles Override Indicator
- Overridden loop miles in Row Detail Modal display in green (`#4caf50`)
- Asterisk appended to pace in Runner Stats for legs with overridden miles: `12:30/mi*`

---

## Data Backup & Safety

### Primary Safety Net
- `localStorage` auto-saves after every single entry
- On app launch: if localStorage contains race data, it is restored silently
- "Last saved: [time]" in the footer is the permanent indicator of save state
- A restore prompt only appears if localStorage is empty but a file backup exists

### Automatic File Backup (Android + Chrome only)
- On first launch, a non-blocking banner prompts user to set up backup folder
- Folder handle stored in IndexedDB for persistence across sessions
- After every Actual Finish entry, app silently writes a new CSV to that folder
- CSV filename format: `ragnar_tracker_YYYYMMDD_HHMM.csv` (new file per backup)
- Green dot appears next to "Last saved" in footer after successful backup
- Backup folder name shown as tappable hyperlink in Settings — tap to change folder

### Manual Export
- Export CSV button in the Settings panel (Backup Status section)
- Generates downloadable CSV of all current race data on demand

### CSV Export Format

```
[Race Summary]
Team Name, Event, Race Start, Finish Time, Total Duration, Status

[Race Data]
Leg, Runner, Loop, Loop Miles, Start, Est Finish, Actual Finish, Status,
Actual Duration, Actual Pace, Predicted Pace, Estimated Duration

[Config]
Team Name, Event Name, Race Start Time, Avg Pace, Green Miles, Yellow Miles,
Red Miles, Night Adjustment, Night Adjustment Type, Night Hours Start,
Night Hours End
```

Loop Miles in Race Data reflects `getEffectiveMiles()` — override value if set,
config default otherwise.

---

## Kiosk Setup (Race Day)

1. Load the HTML file onto the device before arriving at the venue
2. Open in Chrome — grant File System Access permission and confirm backup folder
3. Open Settings — enter all team and race details
4. Verify and update loop distances once confirmed at venue
5. Enable Android Screen Pinning
6. Plug into battery pack if needed
7. Place device at campsite handoff area

---

## Build Sequence

1. ✓ HTML/CSS dark mode layout and touch interface
2. ✓ Config panel — all setup fields, defaults, reset, and restore controls
3. ✓ Race table with auto-generated leg assignments from config
4. ✓ Status bar and footer
5. ✓ Core calculation logic (start times, finish values, status)
6. ✓ Forecasting logic (pace model + night pace adjustment)
7. ✓ Actual Finish direct tap from table → time entry keypad
8. ✓ Time entry keypad (all variants)
9. ✓ Row Detail Modal (floating card)
10. ✓ Runner substitution custom dropdown within modal
11. ✓ Deleted finish time behavior
12. ✓ Race Complete state and auto-revert
13. ✓ localStorage auto-save and silent restore on launch
14. ✓ Config panel keypad integration (all variants)
15. ✓ File System Access API automatic backup
16. ✓ Manual CSV export (in Settings panel)
17. ✓ Visual design refinement
18. ✓ Fresh load experience (Race Setup title, auto-open Settings)
19. ✓ Reset and Restore buttons with feedback
20. ✓ Night pace adjustment — pace-based with normalization
21. ✓ Runner Stats modal
22. ✓ Export button moved to Settings panel
23. ✓ Backup folder as tappable link
24. ✓ Loop Miles override per leg
25. ✓ Runner Stats grouping by current runner (not default runner)
26. → UAT testing
27. → Race day kiosk setup verification

---

## Post-MVP / Future Items

- **Instruction document** — user-facing setup and race day guide
- **In-app help / info buttons** — contextual help within the app
- **iOS support** — backup strategy via manual export and periodic reminders
- **Night Pace Adjustment validation** — default 5% to be validated against
  historical race data before next race

---

*Document version: 7.0 — Updated April 17, 2026*
*Based on 2025 Ragnar Trail Atlanta build and subsequent planning and build discussion*
