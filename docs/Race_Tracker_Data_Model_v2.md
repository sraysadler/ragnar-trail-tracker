# Race Tracker — Data Model

A complete reference for every data field the app tracks, stores, or
calculates. Organized by data group. Use this during the JavaScript
build to ensure consistent field naming, types, and sources.

---

## Config Data

Stored in localStorage under the key `raceConfig`. Included in every
CSV export. Set by the user in the Config panel before the race begins.

| Field | Key | Type | Source | Default |
|-------|-----|------|--------|---------|
| Team name | `teamName` | String | User input (text) | `""` |
| Event name | `eventName` | String | User input (text) | `""` |
| Race start time | `raceStartTime` | String (hh:mm AM/PM) | User input (time keypad) | `"8:00 AM"` |
| Runner 1 name | `runner1` | String | User input (text) | `""` |
| Runner 2 name | `runner2` | String | User input (text) | `""` |
| Runner 3 name | `runner3` | String | User input (text) | `""` |
| Runner 4 name | `runner4` | String | User input (text) | `""` |
| Runner 5 name | `runner5` | String | User input (text) | `""` |
| Runner 6 name | `runner6` | String | User input (text) | `""` |
| Runner 7 name | `runner7` | String | User input (text) | `""` |
| Runner 8 name | `runner8` | String | User input (text) | `""` |
| Green loop miles | `greenMiles` | Number | User input (numeric keypad) | `4.0` |
| Yellow loop miles | `yellowMiles` | Number | User input (numeric keypad) | `5.0` |
| Red loop miles | `redMiles` | Number | User input (numeric keypad) | `6.0` |
| Estimated team avg pace | `avgPace` | Number (minutes per mile) | User input (pace keypad) | `12.0` |
| Night pace adjustment | `nightAdjustment` | Number (%) | User input (numeric keypad) | `5` |
| Night adjustment type | `nightAdjustmentType` | String (`"percent"`) | Fixed — percentage only in MVP | `"percent"` |
| Night hours start | `nightStart` | String (hh:mm AM/PM) | User input (time keypad) | `"8:00 PM"` |
| Night hours end | `nightEnd` | String (hh:mm AM/PM) | User input (time keypad) | `"7:00 AM"` |

### Notes on Config Storage
- All config values stored in `localStorage` under key `raceConfig`
- `nightAdjustmentType` is hardcoded to `"percent"` for MVP. The field
  is retained in the data structure to allow future support for MM:SS
  per mile format without a schema change.

---

## File Backup Handle Storage

The File System Access API folder handle **cannot** be stored in
`localStorage` — it is not JSON-serializable. It is stored separately
in `IndexedDB` under the database name `RaceTrackerBackup`, object
store `handles`, key `backupFolder`.

```
IndexedDB: RaceTrackerBackup
  → objectStore: handles
    → key: 'backupFolder'  →  value: FileSystemDirectoryHandle
```

This is separate from all other app data and persists independently
of localStorage.

---

## Leg Data

Stored in localStorage under the key `raceLegs` as an array of 24
leg objects. One object per leg, indexed 0–23 (displayed as Leg 1–24).

### Fixed Fields
Set at app initialization from the leg assignment table. Never change
unless a runner substitution is made.

| Field | Key | Type | Source |
|-------|-----|------|--------|
| Leg number | `legNumber` | Number (1–24) | Fixed — `LEG_DEFINITIONS` constant |
| Loop color | `loop` | String (`"Green"`, `"Yellow"`, `"Red"`) | Fixed — `LEG_DEFINITIONS` constant |
| Default runner | `defaultRunner` | String | Fixed — derived from config runner order at init |

### LEG_DEFINITIONS — Static Constant
Loop color, loop initial, and row background color for all 24 legs are
defined as a hardcoded constant array in the JavaScript. They are never
derived dynamically from config. The `rowColor` values are:

- Green: `#003300`
- Yellow: `#444400`
- Red: `#330000`

### Editable Fields
The only fields a user touches during the race.

| Field | Key | Type | Source |
|-------|-----|------|--------|
| Runner | `runner` | String | Defaults to `defaultRunner`; editable via Row Detail Modal custom dropdown |
| Actual finish time | `actualFinish` | String (hh:mm AM/PM) or `null` | User input via time entry keypad |

### Calculated Fields
Derived at runtime from fixed fields, editable fields, config data,
and other calculated fields. Never stored — always recomputed.

| Field | Key | Type | Calculation |
|-------|-----|------|-------------|
| Loop miles | `loopMiles` | Number | Lookup from config: Green → `greenMiles`, Yellow → `yellowMiles`, Red → `redMiles` |
| Start time | `startTime` | String (hh:mm AM/PM) | Leg 1: `raceStartTime`. All others: previous leg's `finishTime` |
| Estimated duration | `estimatedDuration` | Number (minutes) | `predictedPace × loopMiles × (1 + nightAdjustment/100)` if night leg |
| Finish time | `finishTime` | String (hh:mm AM/PM) | If `actualFinish` set: `actualFinish`. Otherwise: `startTime + estimatedDuration` |
| Est finish time | `estFinishTime` | String (hh:mm AM/PM) | Always `startTime + estimatedDuration`, regardless of actual |
| Actual duration | `actualDuration` | Number (minutes) or `null` | `actualFinish − startTime`. Null if no actual finish. |
| Actual pace | `actualPace` | Number (minutes per mile) or `null` | `actualDuration ÷ loopMiles`. Null if no actual finish. |
| Predicted pace | `predictedPace` | Number (minutes per mile) | See Forecasting Logic below |
| Status | `status` | String | See Status Logic below |
| Night leg | `isNightLeg` | Boolean | True if `startTime` falls within `nightStart`–`nightEnd` window (handles midnight crossing) |

---

## Forecasting Logic (Predicted Pace)

Runner pace history is built incrementally as legs are processed
in order 1→24. Each leg's predicted pace uses only that runner's
own completed leg history.

| Prior legs completed | Predicted Pace | Status |
|---------------------|----------------|--------|
| 0 | Config `avgPace` | `Estimated (Baseline)` |
| 1 | Runner's actual pace from their 1 completed leg | `Estimated (Live)` |
| 2+ | Simple equal-weight average of all runner's actual paces | `Estimated (Live)` |

No team loop-color average is used. The model relies solely on
each runner's own pace history.

---

## Status Logic

| Condition | Status |
|-----------|--------|
| `actualFinish` is set | `"Actual"` |
| `actualFinish` is null AND runner has 0 completed legs | `"Estimated (Baseline)"` |
| `actualFinish` is null AND runner has 1+ completed legs | `"Estimated (Live)"` |

---

## Derived Race Summary Data

Computed at runtime from leg data. Used in the status bar and CSV export.
Never stored — always recomputed from current leg state.

| Field | Calculation |
|-------|-------------|
| Projected team finish | `finishTime` of Leg 24 |
| Current leg number | First leg where `actualFinish` is null |
| Current runner | `runner` of the current leg |
| Next runner | `runner` of the leg after the current leg |
| Next start time | `startTime` of the leg after the current leg |
| Race complete | True when all 24 legs have `actualFinish` set |

### Race Complete Display
When race is complete:
- Status bar Row 1 label: "FINAL TEAM FINISH"
- Current Loop: "Complete"
- Next Runner: "—"
- Next Start: "—"

---

## Storage Summary

| Data | Storage | Key / Location |
|------|---------|----------------|
| Config values | `localStorage` | `raceConfig` |
| Leg state (runner + actualFinish) | `localStorage` | `raceLegs` |
| File backup folder handle | `IndexedDB` | `RaceTrackerBackup` → `handles` → `backupFolder` |
| Calculated fields | Memory only | Never persisted |
| Last backup timestamp | `localStorage` | `lastBackupTime` |

---

## localStorage Structure

```
raceConfig  →  { teamName, eventName, raceStartTime, runner1–8,
                 greenMiles, yellowMiles, redMiles, avgPace,
                 nightAdjustment, nightAdjustmentType,
                 nightStart, nightEnd }

raceLegs    →  [ { legNumber, loop, defaultRunner, runner,
                   actualFinish }, ... ]  (24 items)

lastBackupTime  →  ISO timestamp string of last successful backup write
```

Calculated fields are never stored. They are always recomputed from
stored raw values when the app loads or when any value changes.

---

## CSV Export Structure

Three sections, in this order: Race Summary, Race Data, Config.
Both automatic backup and manual export use the same `generateCSV()`
function.

### [Race Summary]
One header row + one data row.

```
Team Name, Event, Race Start, Finish Time, Total Duration, Status
```

Status values: `"In Progress"` or `"Complete"`

### [Race Data]
One header row + 24 data rows (one per leg).

```
Leg, Runner, Loop, Loop Miles, Start, Est Finish, Actual Finish,
Status, Actual Duration, Actual Pace, Predicted Pace, Est Duration
```

### [Config]
One header row + one data row.

```
Team Name, Event Name, Race Start Time, Avg Pace, Green Miles,
Yellow Miles, Red Miles, Night Adjustment, Night Adjustment Type,
Night Hours Start, Night Hours End
```

---

## Data Flow Summary

```
Config input (text + custom keypads)
    ↓
Loop miles lookup + avg pace + night settings
    ↓
LEG_DEFINITIONS (fixed: leg#, loop color, row color)
+ runner names from config rotation
    ↓
User edits runner (optional) + enters actual finish times via keypad
    ↓
calculateLegData() runs 1→24:
  Build runner pace history incrementally
  Per leg: startTime → predictedPace → estimatedDuration
         → finishTime → status → isNightLeg
         → actualDuration → actualPace (if actual exists)
    ↓
Race summary derived:
  projected finish, current leg, next runner, next start
    ↓
localStorage written (config + raw leg data only)
    ↓
IndexedDB folder handle checked → file backup written (CSV)
    ↓
renderTable() + renderStatusBar() called
```

---

## Field Naming Conventions

- camelCase for all JavaScript keys
- Times stored and displayed as strings in `h:mm AM/PM` format
- Durations and paces stored internally as decimal minutes
  (e.g. 1 hour 10 minutes = 70.0, pace of 13:22/mi = 13.367)
- `null` used (not empty string or zero) for fields with no value yet
- Loop color always stored as full string: `"Green"`, `"Yellow"`, `"Red"`
- Status always stored as full string: `"Actual"`,
  `"Estimated (Baseline)"`, `"Estimated (Live)"`
- Night adjustment always stored as a number (percentage value, e.g. `5`)

---

*Document version: 2.0 — Updated April 16, 2026*
*Companion to: Race Tracker App Plan v6, Race Tracker Wireframes v1,
Race Tracker Do Not Do v1, Race Tracker Decisions Log v1,
Race Tracker Build Log v2*
