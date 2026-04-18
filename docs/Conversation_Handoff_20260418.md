# Ragnar Trail Tracker — Conversation Handoff

Use this document to orient a new Claude conversation. Read this first,
then reference the project documents as needed.

---

## What This Project Is

A single-file offline HTML/JS/CSS touchscreen kiosk app for tracking
Ragnar Trail relay races. Primary target: Android tablet, Chrome.

- **GitHub repo:** github.com/sraysadler/ragnar-trail-tracker
- **Live app:** sraysadler.github.io/ragnar-trail-tracker/ragnar-trail-tracker.html
- **Single file:** `ragnar-trail-tracker.html` in repo root

---

## Current Status (April 17, 2026)

**The app is feature-complete and has been tested on the target Android
device.** Steps 1–25 of the build sequence are complete.

### What's done:
- Full race table with 24 legs, cascading start times, estimated finishes
- Config panel (Race Setup on first load, Settings thereafter)
- All custom keypads — time, numeric (implied decimal), pace (MM:SS)
- Time entry keypad from race table and Row Detail Modal
- Row Detail Modal with runner substitution and loop miles override
- Forecasting: runner pace history, night pace adjustment (pace-based,
  normalized), `getEffectiveMiles()` for per-leg overrides
- Runner Stats modal (footer button) with Team Totals
- localStorage auto-save, File System Access API backup, manual CSV export
- Race Complete state, Reset, Restore with feedback
- Visual design refinement

### What remains:
- UAT testing (checklist exists in docs/)
- Race day kiosk setup verification on Android tablet
- Screen Pinning on ECOPAD tablet — still being figured out, not blocking

### Post-MVP items (not yet scheduled):
- Instruction document for end users
- In-app help / info buttons
- iOS support

---

## Key People and Context

- **User:** Ray Sadler (sraysadler on GitHub)
- **Team:** Party Pace — Ragnar Trail Atlanta
- **Runners (2025):** Wendy, Kristine, Scott, Jeff, Jason, Ray, Jenny, Tim
- **2025 race:** 9:00 AM start, 9:56 AM finish (24h 56m)
- **Test device:** ECOPAD Android 15 tablet, Chrome

---

## Architecture Decisions (do not change without discussion)

- Single .html file — no frameworks, no CDN, no build step
- localStorage for all race data (`raceConfig`, `raceLegs`, `lastBackupTime`)
- IndexedDB for File System Access API folder handle only
- Full re-render (`renderTable()` + `renderStatusBar()`) on every change
- `LEG_DEFINITIONS` is a hardcoded static constant — never derived
- `getEffectiveMiles(leg, config)` used everywhere loop miles are needed
- Pace history stores normalized day-equivalent paces only
- Runner Stats and all forecasting uses `runner` field, not `defaultRunner`

---

## Data Model Summary

### localStorage keys:
```
raceConfig  → { teamName, eventName, raceStartTime, runner1–8,
                greenMiles, yellowMiles, redMiles, avgPace,
                nightAdjustment, nightAdjustmentType,
                nightStart, nightEnd }

raceLegs    → [ { legNumber, loop, defaultRunner, runner,
                  actualFinish, overrideMiles }, ... ] (24 items)

lastBackupTime → ISO timestamp string
```

### IndexedDB:
```
RaceTrackerBackup → handles → backupFolder → FileSystemDirectoryHandle
```

---

## Forecasting Logic Summary

Night adjustment is pace-based with normalization:
- Night actual pace stored as: `actualPace / (1 + nightAdjustment/100)`
- 0 prior legs → config avgPace → Estimated (Baseline)
- 1+ prior legs → average of normalized paces
- Night leg prediction → `normalizedAvg × (1 + nightAdjustment/100)`
- Day leg prediction → normalizedAvg

Loop miles per leg always via `getEffectiveMiles(leg, config)` —
returns `overrideMiles` if set, else config default for that loop color.

---

## UI Layout

Four fixed zones + one scrollable:
- **Zone 1:** Header (team name, event name, current time)
- **Zone 2:** Status bar (projected finish, current loop, next runner,
  next start)
- **Zone 3:** Race table (scrollable, column headers fixed)
- **Zone 4:** Footer (Runner Stats button | ● Last saved | Settings button)

**Overlays (full-screen):** Settings panel, keypad, Runner Stats modal
**Overlays (floating card):** Row Detail Modal

---

## Style Constants

- Loop row colors: Green `#003300`, Yellow `#444400`, Red `#330000`
- Active leg outline: `2px solid #fff`
- Estimated times: italic, `#999`
- Actual times: bold, white
- Tappable config fields: `#0a1a2a` bg, `1px solid #4a9eff` border
- Status colors: Actual = `#4caf50`, Baseline = `#888`, Live = `#4a9eff`
- Override miles in modal: `#4caf50` (green)
- Backup dot: `#4caf50` (green), immediately left of "Last saved"

---

## Recent Changes (since last major doc update)

All of these are reflected in v3 documents:
- Night pace adjustment updated to pace-based with normalization
- Runner Stats modal added (footer button)
- Export button moved from footer to Settings panel (Backup Status section)
- Backup folder shown as tappable link to change folders
- Loop miles override per leg (`overrideMiles`, press-and-hold 3s)
- Runner Stats grouping fixed to use `runner` not `defaultRunner`
- Fresh load experience: auto-open Race Setup, blank fields
- Reset/Restore buttons working with feedback messages

---

## Project Documents

All in `/docs` folder of GitHub repo:

| Document | Version | Purpose |
|----------|---------|---------|
| `Race_Tracker_App_Plan_v7.md` | v7 | Full feature spec |
| `Race_Tracker_Data_Model_v3.md` | v3 | All data fields |
| `Race_Tracker_Decisions_Log_v3.md` | v3 | Key decisions + reasoning |
| `Race_Tracker_Build_Log_v3.md` | v3 | Build history + status |
| `Race_Tracker_UAT_Checklist_v2.md` | v2 | 16 sections, 110 test cases |
| `Race_Tracker_Wireframes_v3.html` | v3 | 5-screen UI wireframes |
| `Race_Tracker_Do_Not_Do_v1.md` | v1 | Rejected approaches |
| `Race_Tracker_Sample_Data_2025.csv` | — | 2025 actual race data |
| `build-briefs/` | — | Per-step Claude Code task briefs |

---

## How This Project Works

Planning and documentation happens in **Claude.ai** (this conversation).
Building happens in **Claude Code** (separate terminal sessions).

Workflow:
1. Discuss feature/fix here → produce a build brief document
2. Take brief to Claude Code → build and commit
3. Screenshot results → review here → iterate or sign off
4. Update documentation here when significant changes are made

---

## Known Issues / Watch List

- **Tim's Leg 8 pace (8.20/mi)** — suspiciously fast. Verify against
  actual 2025 race records before using as forecasting baseline.
- **Night Pace Adjustment default (5%)** — working value, not yet
  validated against historical data.
- **Android Screen Pinning** — not yet fully verified on ECOPAD tablet.
  App works correctly without it.
- **File System Access API on Android** — permission persistence after
  full Chrome close needs verification on ECOPAD.

---

*Handoff document — April 17, 2026*
