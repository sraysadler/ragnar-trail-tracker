# Ragnar Trail Tracker

A touchscreen kiosk app for tracking Ragnar Trail relay races in real time. Runs offline as a single HTML file on any Android device.

---

## What It Does

Ragnar Trail is a relay race format where 8 runners each run 3 loops (Green, Yellow, Red) in sequence, handing off a bib as each runner finishes. The race runs continuously for roughly 24–30 hours with no breaks between legs.

This app replaces the spreadsheet approach with a purpose-built touchscreen interface designed to be used at the race campsite — in the dark, by tired runners, with no internet connection.

**During the race, the app:**
- Displays a live 24-leg race table with start times, estimated finish times, and actual finish times
- Updates all downstream estimates automatically as actual times are entered
- Shows a real-time projected team finish time
- Tracks who is currently running and who is up next
- Gets smarter as the race progresses — the forecasting model learns each runner's actual pace

---

## Key Features

- **Fully offline** — no internet connection required during the race
- **Single file** — the entire app is one `.html` file, no installation needed
- **Dark mode** — optimized for night use and battery conservation
- **Custom time entry keypad** — no system keyboard, large touch targets, calculator-style entry
- **Live forecasting** — pace model uses each runner's own history, falling back to a configurable baseline
- **Night pace adjustment** — automatically applies a configurable pace adjustment to legs running between configurable night hours
- **Three-layer backup** — localStorage auto-save, automatic file backup (Android + Chrome), and manual CSV export
- **Row detail view** — tap any leg to see full stats including actual pace, predicted pace, and status
- **Runner substitution** — change the runner for any leg via the detail view
- **Race Complete state** — triggered automatically when all 24 legs are entered

---

## Tech Stack

- HTML, CSS, JavaScript — no frameworks, no external dependencies
- `localStorage` for primary data persistence
- `IndexedDB` for File System Access API folder handle storage
- Chrome File System Access API for automatic background backup (Android + Chrome only)

---

## Target Device

**Primary:** Android tablet running Android 15+, Chrome (latest)

The app is designed to sit at the race campsite handoff area as a shared kiosk. Screen Pinning in Android locks Chrome to the foreground to prevent accidental navigation.

**iOS:** Future consideration. The File System Access API is not available on iOS (any browser). Manual CSV export and periodic backup reminders are the fallback strategy on iOS.

---

## How to Use

### Before the race
1. Download `ragnar-trail-tracker.html` to your device
2. Open the file in Chrome
3. Tap **Settings** and fill in your team name, event name, runner names in order, race start time, and loop distances
4. Grant the one-time file backup permission when prompted
5. Enable Android Screen Pinning
6. Place the device at the campsite handoff area

### During the race
- Tap the **Actual Finish** cell for any leg to enter a finish time via the custom keypad
- Tap anywhere else on a row to open the leg detail view — see full stats and change runner if needed
- All estimates update automatically as times are entered
- Tap **Export** in the footer at any time for a manual CSV backup

### After the race
- Tap **Export** for a final race report CSV
- The CSV includes full race data, runner paces, and config settings

---

## Project Structure

```
ragnar-trail-tracker/
├── ragnar-trail-tracker.html    # The app (single file)
├── README.md
└── docs/
    ├── App_Plan_v6.md           # Full feature specification
    ├── Wireframes_v1.html       # UI wireframes (open in browser)
    ├── Do_Not_Do_v1.md          # Rejected approaches and why
    ├── Decisions_Log_v2.md      # Key decisions and reasoning
    ├── Data_Model_v2.md         # All data fields defined
    ├── Build_Log_v2.md          # Build decisions and current status
    └── Sample_Data_2025.csv     # Real 2025 race data for testing
```

---

## Race Format (Fixed)

This app is built specifically for the Ragnar Trail relay format:
- 8 runners per team
- 3 loops per runner: Green → Yellow → Red (always in this order)
- 24 total legs, run sequentially
- Loop distances configurable — defaults: Green 4 mi, Yellow 5 mi, Red 6 mi

---

## Build Status

**Steps 1–13 complete.** Core app is functional — layout, config panel,
race table, status bar, calculation chain, forecasting logic, time entry
keypad, and row detail modal are all built and working.

**Currently in progress:**
- Step 14: Config panel keypad integration (all config fields wired to
  custom keypads)
- Step 15: File System Access API automatic backup
- Step 16: Manual CSV export
- Step 17: Visual design refinement
- Step 18: UAT checklist
- Step 19: On-device testing
- Step 20: Race day kiosk setup verification

See `docs/Build_Log_v2.md` for full build history and remaining steps.

---

*Built for Party Pace — Ragnar Trail Atlanta 2025*
*Generalized for use by any Ragnar Trail team*
