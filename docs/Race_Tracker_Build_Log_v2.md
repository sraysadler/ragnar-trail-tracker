# Ragnar Trail Tracker — Build Log

A running record of decisions, changes, and deviations from the original
app plan made during the build. Use this to keep Claude Code sessions
aligned when starting fresh, and to track what has been built and what
remains.

---

## Build Session: April 15–16, 2026
### Steps 1–13 completed

---

## Step 1 — HTML/CSS Layout ✓

**Completed as specified.**

**Adjustments made:**
- Yellow row color changed from `#333300` to `#444400` — original value
  was too dark and indistinguishable from green rows at a glance
- Active leg outline brightened from `1px solid #555` to `2px solid #888`
  for better visibility against the red row background

---

## Step 2 — Config Panel ✓

**Completed as specified with adjustments:**

- Config panel implemented as a **floating overlay** on top of the main
  view — not a full-screen replacement panel
- Team name and Event name fields made approximately twice as wide as
  other tappable fields — user requested this explicitly
- All tappable fields use blue-bordered dark style (`#0a1a2a` background,
  `1px solid #4a9eff` border) consistently
- Text overflow in Team name and Event name fields is clipped
- Night Hours implemented as two separate tappable time fields with a
  dash separator, right-aligned
- Reset/Restore/Save buttons inside the overlay footer
- **Note:** Config fields are styled as tappable but not yet wired to
  keypads — that work is scheduled as a separate step (Config Panel
  Keypad Integration, now Step 14 in the build sequence)

---

## Step 3 — Race Table Generation ✓

**Completed as specified.**

- `LEG_DEFINITIONS` hardcoded as a static constant — loop colors,
  initials, and row backgrounds never derived dynamically
- Runner assignment: `(leg - 1) % 8` formula
- `renderTable()` called on page load and after every config Save
- Yellow row color in `LEG_DEFINITIONS`: `#444400`

---

## Step 4 — Status Bar ✓

**Completed after Step 5 due to dependency on calculation chain.**

**Implemented:**
- Projected Team Finish: `finishTime` of Leg 24
- Current Loop: `[legNumber] – [runnerName]` of first leg without actual finish
- Next Runner: runner of leg after current
- Next Start: start time of leg after current
- Current time in header: actual device time, updates every 60 seconds,
  displayed as `h:mm AM/PM` with "Current time:" label
- Last Saved: permanently visible in footer, updates on every entry and
  config Save, shows "Last saved: —" on first load with no data

**Race Complete state:**
- Projected Team Finish bar label → "FINAL TEAM FINISH"
- Current Loop → "Complete"
- Next Runner and Next Start → "—"
- Reverts automatically if any actual finish is cleared

**Decision — no flash message on restore:**
No separate "Race data restored" message on launch. The permanent
"Last saved: [time]" indicator serves this purpose. Data restores
silently and automatically.

---

## Step 5 — Core Calculation Logic ✓

**Completed as specified.**

- Time arithmetic in minutes-since-midnight internally
- Midnight crossing handled correctly
- `calculateLegData()` processes legs strictly 1→24
- Full re-render on every actual finish entry
- `prompt()` used temporarily for time entry testing — removed in Step 7+8

---

## Step 6 — Forecasting Logic ✓

**Simplified from original spec.**

**Team loop-color average removed from forecasting model.**

Final model:
- 0 prior legs → config baseline pace → `Estimated (Baseline)`
- 1 prior leg → runner's actual pace from that leg → `Estimated (Live)`
- 2+ prior legs → equal-weight average of runner's actual paces
  → `Estimated (Live)`

Night pace adjustment:
- Applied as a percentage to estimated duration (not to pace itself)
- Applied when leg's estimated start falls within night window
- Handles midnight crossing correctly
- Default: 5%

**Bug found and fixed:**
Initial implementation had a unit conversion error causing Leg 9
estimated duration to show ~165 minutes instead of ~75 minutes.
Fixed after adding console.log diagnostics. Root cause: pace history
not being passed correctly to the estimation function.

**Testing workaround:**
Config start time temporarily changed in code to 9:00 AM to test
against 2025 race data (config fields not yet interactive without keypad).

---

## Steps 7+8 — Time Entry Keypad ✓

**Combined into single step.**

**Keypad layout (confirmed):**
```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[AM/PM] [ 0 ] [ ⌫ ]
```
All buttons equal size — AM/PM is NOT wider than other buttons.

**Time display:**
- Shows as `h:mm` with AM/PM as a separate indicator
- PM shown as blue boxed indicator in display area; AM shown as blue
  text in keypad grid button
- This inconsistency noted and accepted by user — do not change

**Clear behavior:**
- When all digits backspaced from existing entry: display shows `--:-- AM`,
  Confirm button changes to red "Clear"
- Tapping Clear removes the actual finish time
- Beginning to type new digits reverts Clear back to Confirm
- No separate confirmation dialog — button state change serves as warning

**Digit entry:**
- Right-to-left calculator style
- Intermediate states (e.g. `--:65` while typing `6:54`) are expected
  and correct — not a bug

**`prompt()` removed entirely.**

---

## Step 9 — Row Detail Modal ✓

**Completed with adjustments.**

**Modal size:**
- Originally built as full-screen overlay
- User requested floating card pattern matching keypad/Settings
- **Final:** Floating card centered over dimmed background
- Race table visible but dimmed behind the modal
- Tapping dimmed backdrop closes modal

**Final field order:**
1. Runner (custom dropdown)
2. Loop Miles
3. Start
4. Est Finish
5. Actual Finish (blue tappable box)
6. Status
7. Est Duration
8. Actual Duration
9. Predicted Pace
10. Actual Pace

**Runner dropdown:**
- Native `<select>` replaced with custom dark UI dropdown
- Selected runner highlighted in blue with checkmark
- Chevron flips ▼/▲ on open/close
- Tapping outside dismisses without changing selection

**Save behavior:**
- Actual Finish: saves immediately when keypad confirms (does not wait
  for modal Save)
- Runner change: saves only when modal Save changes is tapped
- Cancel prompts if runner was changed but not saved

---

## Step 10 — Runner Substitution ✓

**Completed as part of Step 9.** Custom runner dropdown built directly
into the Row Detail Modal. No separate step needed.

---

## Step 11 — Deleted Finish Time Behavior ✓

**Completed as part of Steps 7+8.** Clear mechanism in keypad handles
deletion. Cascade behavior built in Step 5.

---

## Step 12 — Race Complete State ✓

**Completed as part of Step 4.** Verified with all 24 actual finish
times from 2025 race data:
- "FINAL TEAM FINISH" label appears
- Final time 9:56 AM shown correctly
- "Complete" in Current Loop
- "—" in Next Runner and Next Start
- Reverts correctly when any actual finish cleared

---

## Step 13 — localStorage Auto-Save and Restore ✓

**Completed as part of Step 5.**

- Data persists across browser tab closes and reopens
- All actual finish times restore correctly on reload
- Estimates recalculate from restored data
- Silent restore — no prompt, no flash message

---

## Remaining Build Steps

### Step 14 — Config Panel Keypad Integration (NEXT)
**Moved up in sequence** — config fields need to be interactive before
backup testing can be done properly.

Field-to-keypad mapping:
- Race Start Time → standard time keypad (AM/PM toggle)
- Green / Yellow / Red Miles → numeric keypad with "." replacing AM/PM
- Avg Pace → numeric keypad without AM/PM, `/mi` suffix shown in field
- Night Adjustment → numeric keypad with "." replacing AM/PM
- Night Hours From / To → standard time keypad (AM/PM toggle)
- Runner name fields → system keyboard (text input, acceptable here)

### Step 15 — File System Access API Automatic Backup
- Folder handle stored in **IndexedDB** (not localStorage — not serializable)
- Each backup creates a new timestamped file: `ragnar_tracker_YYYYMMDD_HHMM.csv`
- New file per backup — never overwrites existing backups
- Green dot indicator next to "Last saved" in footer after successful backup
- "Last backed up: [date], [time]" in Settings panel
- `generateCSV()` built as shared function used by both automatic backup
  and manual export (Step 16)
- Restore prompt appears only when localStorage is empty but backup exists
- Must be tested on Android tablet after desktop verification
- First launch: non-blocking banner prompts user to set up backup folder

### Step 16 — Manual CSV Export
- Uses shared `generateCSV()` from Step 15
- Export button always visible in footer
- CSV format: Race Summary → Race Data → Config

### Step 17 — Visual Design Refinement
- Loop colors, bold/italic polish, dark mode consistency review
- Any UI issues surfaced during testing addressed here

### Step 18 — UAT Checklist and Test Plan
- Written against the working app, not the spec
- To be produced in this planning conversation, not in Claude Code

### Step 19 — On-Device Testing
- Full 24-leg simulation on Android tablet
- File System Access API backup verification on Android
- Screen Pinning verification

### Step 20 — Race Day Kiosk Setup Verification
- End-to-end setup walkthrough on actual device

---

## Known Issues / Watch List

- **Tim's Leg 8 pace (8.20/mi)** — suspiciously fast for a trail race.
  Needs verification against actual race records before using 2025 data
  as a forecasting baseline.
- **Config fields not interactive** — fully addressed in Step 14.
  Any testing that requires changing config values (start time, loop
  distances, avg pace) should be done after Step 14 is complete.
- **File System Access API** — verified working in Chrome on Mac.
  Android-specific behavior (permission persistence after full Chrome
  close) must be verified on the ECOPAD tablet.

---

*Build Log version: 2.0 — Updated April 16, 2026*
*Covers: Steps 1–13 complete. Steps 14–20 remaining.*
*Companion to: Race Tracker App Plan v6, Race Tracker Data Model v2,
Race Tracker Decisions Log v2*
