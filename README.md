# Ragnar Trail Tracker

A touchscreen kiosk app for tracking Ragnar Trail relay races in real time. Runs offline as a single HTML file on any Android device.

**Live app:** https://sraysadler.github.io/ragnar-trail-tracker/ragnar-trail-tracker.html

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
- **Live forecasting** — pace model uses each runner's own pace history with night pace normalization
- **Night pace adjustment** — configurable pace penalty for legs running during night hours
- **Three-layer backup** — localStorage auto-save, automatic file backup (Android + Chrome), and manual CSV export in Settings
- **Row detail view** — tap any leg to see full stats, edit actual finish, change runner, or override loop miles
- **Runner Stats** — per-runner pace history and team totals, accessible from the footer
- **Loop miles override** — correct per-leg distances when a runner covers unexpected mileage
- **Runner substitution** — change the runner for any leg; all stats and forecasts follow the current assignment
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

**iOS:** Future consideration. The File System Access API is not available on iOS (any browser). Manual CSV export is the backup strategy on iOS.

---

## How to Use

### Before the race
1. Open the app URL in Chrome on your device
2. When "Race Setup" opens automatically, fill in your team name, event name, runner names in order, race start time, and loop distances
3. Grant the one-time file backup permission when prompted
4. Enable Android Screen Pinning
5. Place the device at the campsite handoff area

### During the race
- Tap the **Actual Finish** cell for any leg to enter a finish time via the custom keypad
- Tap anywhere else on a row to open the leg detail view — see full stats, edit runner, or override loop miles if needed
- Tap **Runner Stats** in the footer to see per-runner pace history and team totals
- All estimates update automatically as times are entered

### After the race
- Open **Settings** and tap **Export CSV** for a final race report
- The CSV includes full race data, runner paces, and config settings

---

## Project Structure

```
ragnar-trail-tracker/
├── ragnar-trail-tracker.html    # The app (single file)
├── README.md
└── docs/
    ├── App_Plan_v7.md                  # Full feature specification
    ├── Race_Tracker_Wireframes_v3.html # UI wireframes (open in browser)
    ├── Race_Tracker_Do_Not_Do_v1.md    # Rejected approaches and why
    ├── Race_Tracker_Decisions_Log_v3.md # Key decisions and reasoning
    ├── Race_Tracker_Data_Model_v3.md   # All data fields defined
    ├── Race_Tracker_Build_Log_v3.md    # Build history and current status
    ├── Race_Tracker_UAT_Checklist_v2.md # Test plan
    ├── Race_Tracker_Sample_Data_2025.csv # Real 2025 race data for testing
    └── build-briefs/                   # Per-step build task briefs
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

**Steps 1–25 complete.** The app is fully functional and has been tested on the target Android device.

**Completed features:**
- Layout, config panel, race table, status bar, calculation chain
- Forecasting with pace-based night adjustment and normalization
- Time entry keypad (all variants)
- Row Detail Modal with loop miles override
- Runner substitution
- Runner Stats modal with Team Totals
- localStorage auto-save and restore
- File System Access API automatic backup
- Manual CSV export (in Settings panel)
- Visual design refinement
- Fresh load / Race Setup experience

**Remaining:**
- UAT testing (checklist in docs/)
- Race day kiosk setup verification

See `docs/Race_Tracker_Build_Log_v3.md` for full build history.

---

*Built for Party Pace — Ragnar Trail Atlanta 2025*
*Generalized for use by any Ragnar Trail team*
