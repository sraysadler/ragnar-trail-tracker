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
| Team name | `teamName` | String | User input | `""` |
| Event name | `eventName` | String | User input | `""` |
| Race start time | `raceStartTime` | String (hh:mm AM/PM) | User input (keypad) | `"8:00 AM"` |
| Runner 1 name | `runner1` | String | User input | `""` |
| Runner 2 name | `runner2` | String | User input | `""` |
| Runner 3 name | `runner3` | String | User input | `""` |
| Runner 4 name | `runner4` | String | User input | `""` |
| Runner 5 name | `runner5` | String | User input | `""` |
| Runner 6 name | `runner6` | String | User input | `""` |
| Runner 7 name | `runner7` | String | User input | `""` |
| Runner 8 name | `runner8` | String | User input | `""` |
| Green loop miles | `greenMiles` | Number | User input (keypad) | `4.0` |
| Yellow loop miles | `yellowMiles` | Number | User input (keypad) | `5.0` |
| Red loop miles | `redMiles` | Number | User input (keypad) | `6.0` |
| Estimated team avg pace | `avgPace` | Number (minutes per mile) | User input (keypad) | `12.0` |
| Night pace adjustment | `nightAdjustment` | Number (% or min/mi) | User input (keypad) | `5` (%) |
| Night adjustment type | `nightAdjustmentType` | String (`"percent"` or `"minPerMile"`) | Derived from input format | `"percent"` |
| Night hours start | `nightStart` | String (hh:mm AM/PM) | User input (keypad) | `"8:00 PM"` |
| Night hours end | `nightEnd` | String (hh:mm AM/PM) | User input (keypad) | `"7:00 AM"` |

---

## Leg Data

Stored in localStorage under the key `raceLegs` as an array of 24
leg objects. One object per leg, indexed 0–23 (displayed as Leg 1–24).

### Fixed Fields
Set at app initialization from the leg assignment table. Never change
unless a runner substitution is made.

| Field | Key | Type | Source |
|-------|-----|------|--------|
| Leg number | `legNumber` | Number (1–24) | Fixed — leg assignment table |
| Loop color | `loop` | String (`"Green"`, `"Yellow"`, `"Red"`) | Fixed — leg assignment table |
| Default runner | `defaultRunner` | String | Fixed — derived from config runner order |

### Editable Fields
The only fields a user touches during the race.

| Field | Key | Type | Source |
|-------|-----|------|--------|
| Runner | `runner` | String | Defaults to `defaultRunner`; editable via Row Detail Modal dropdown |
| Actual finish time | `actualFinish` | String (hh:mm AM/PM) or `null` | User input via time entry keypad |

### Calculated Fields
Derived at runtime from fixed fields, editable fields, config data,
and other calculated fields. Never stored directly — always recomputed.

| Field | Key | Type | Calculation |
|-------|-----|------|-------------|
| Loop miles | `loopMiles` | Number | Lookup from config: Green → `greenMiles`, Yellow → `yellowMiles`, Red → `redMiles` |
| Start time | `startTime` | String (hh:mm AM/PM) | Leg 1: `raceStartTime`. All others: previous leg's `finishTime` |
| Estimated duration | `estimatedDuration` | Number (minutes) | `predictedPace × loopMiles` + night adjustment if applicable |
| Finish time | `finishTime` | String (hh:mm AM/PM) | If `actualFinish` is set: `actualFinish`. Otherwise: `startTime + estimatedDuration` |
| Est finish time | `estFinishTime` | String (hh:mm AM/PM) | Always `startTime + estimatedDuration`, regardless of actual |
| Actual duration | `actualDuration` | Number (minutes) or `null` | `actualFinish − startTime`. Null if no actual finish. |
| Actual pace | `actualPace` | Number (minutes per mile) or `null` | `actualDuration ÷ loopMiles`. Null if no actual finish. |
| Predicted pace | `predictedPace` | Number (minutes per mile) | Weighted formula — see Forecasting Logic in App Plan |
| Status | `status` | String | `"Actual"` if `actualFinish` set. `"Estimated (Baseline)"` if using config avg pace only. `"Estimated (Live)"` if using real race data. |
| Night leg | `isNightLeg` | Boolean | True if `startTime` falls within `nightStart`–`nightEnd` window |

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

---

## localStorage Structure

Two keys used. Both written after every entry.

```
raceConfig  →  { teamName, eventName, raceStartTime, runner1–8,
                 greenMiles, yellowMiles, redMiles, avgPace,
                 nightAdjustment, nightAdjustmentType,
                 nightStart, nightEnd }

raceLegs    →  [ { legNumber, loop, defaultRunner, runner,
                   actualFinish }, ... ]  (24 items)
```

Calculated fields are never stored in localStorage. They are always
recomputed from the stored raw values when the app loads or when
any value changes.

---

## CSV Export Structure

Three sections, in this order: Race Summary, Race Data, Config.

### [Race Summary]
One header row + one data row.

```
Team Name, Event, Race Start, Finish Time, Total Duration, Status
```

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
Config input
    ↓
Loop miles lookup + avg pace + night settings
    ↓
Leg assignment table (fixed: leg#, loop, default runner)
    ↓
User edits runner (optional) + enters actual finish times
    ↓
Calculated per leg:
  startTime → estimatedDuration → finishTime → status
  actualDuration → actualPace → predictedPace
    ↓
Race summary derived:
  projected finish, current leg, next runner, next start
    ↓
localStorage written (config + raw leg data only)
    ↓
File system backup written (full CSV)
```

---

## Field Naming Conventions

- camelCase for all JavaScript keys
- Times stored and displayed as strings in hh:mm AM/PM format
- Durations and paces stored internally as decimal minutes
  (e.g. 1 hour 10 minutes = 70.0, pace of 13:22/mi = 13.367)
- Null used (not empty string or zero) for fields with no value yet
- Loop color always stored as full string: `"Green"`, `"Yellow"`, `"Red"`
- Status always stored as full string: `"Actual"`,
  `"Estimated (Baseline)"`, `"Estimated (Live)"`

---

*Document version: 1.0 — April 15, 2026*
*Companion to: Race Tracker App Plan v5, Race Tracker Wireframes v1,
Race Tracker Do Not Do v1, Race Tracker Decisions Log v1*
