# Build Task: Step 5 — Core Calculation Logic

## Objective
Implement the core timing chain that drives all start times, finish times,
and status values across all 24 legs. Every time any actual finish time
changes, the full chain recalculates and the table re-renders in real time.

This step also includes wiring up actual finish time entry — tapping the
Actual Finish cell in the table opens a time entry keypad (built in Step 8),
but for now tapping it should trigger a simple browser `prompt()` as a
temporary placeholder so the calculation chain can be tested end-to-end.

---

## Data Structure

### Race legs state
Maintain a JavaScript array of 24 leg state objects in memory, stored
under the key `raceLegs` in localStorage:

```javascript
// Each leg object:
{
  legNumber: 1,           // 1–24 (from LEG_DEFINITIONS)
  loop: 'Green',          // from LEG_DEFINITIONS
  runner: 'Wendy',        // from config runner rotation
  actualFinish: null      // null or string 'hh:mm AM/PM' when entered
}
```

Only `runner` and `actualFinish` are mutable. Everything else derives
from `LEG_DEFINITIONS` or is calculated.

### Calculated values (never stored, always recomputed)
Per leg, computed fresh on every render:

| Field | Calculation |
|-------|-------------|
| loopMiles | config.greenMiles / yellowMiles / redMiles based on loop color |
| startTime | Leg 1: config.raceStartTime. All others: previous leg's finishTime |
| estimatedDuration | config.avgPace × loopMiles (in minutes) |
| estFinishTime | startTime + estimatedDuration |
| finishTime | actualFinish if set, otherwise estFinishTime |
| actualDuration | actualFinish − startTime (null if no actual) |
| actualPace | actualDuration ÷ loopMiles (null if no actual) |
| status | See Status Logic below |

---

## Time Handling

All times are stored as strings in `hh:mm AM/PM` format.
Internally, convert to minutes-since-midnight for arithmetic, then
convert back to `hh:mm AM/PM` for display.

```javascript
// String to minutes since midnight
function timeToMinutes(timeStr) {
  // Parse 'hh:mm AM/PM' → total minutes since midnight
  // Handle crossing midnight (times < race start on day 2)
}

// Minutes since midnight to string
function minutesToTime(minutes) {
  // Convert total minutes → 'hh:mm AM/PM'
  // Handle values >= 1440 (next day) gracefully
}
```

**Important:** The race crosses midnight. Times after midnight
(e.g. 1:06 AM on day 2) must be handled correctly. When computing
the chain, if a calculated finish time is earlier than the start time
of the same leg, it has crossed midnight — add 1440 minutes to resolve.

---

## Status Logic

Determine status for each leg:

```
If actualFinish is set:
  → status = 'Actual'

If actualFinish is null AND no actual finish exists anywhere in
legs 1 through (this leg - 1):
  → status = 'Estimated (Baseline)'

If actualFinish is null AND at least one actual finish exists in
any prior leg:
  → status = 'Estimated (Live)'
  (Note: 'Estimated (Live)' at this stage still uses baseline pace —
  the live forecasting model comes in Step 6. The status label is
  correct; the pace calculation will be upgraded in Step 6.)
```

---

## Calculation Chain Function

```javascript
function calculateLegData() {
  // Returns array of 24 computed leg objects
  // Each object contains all calculated fields listed above
  // Uses raceLegs state for actualFinish values
  // Uses config for pace, loop miles, start time
}
```

This function is the heart of the app. It runs top to bottom through
all 24 legs in sequence. Each leg's startTime depends on the previous
leg's finishTime, so they must be calculated in order, 1 through 24.

---

## Render Flow

Every update follows this sequence:

```
1. User enters actual finish time for a leg
2. Save actualFinish to raceLegs[legIndex] in memory
3. Write raceLegs to localStorage
4. Call calculateLegData() → get computed leg array
5. Call renderTable(computedLegs) → rebuild all 24 rows
6. Call renderStatusBar(computedLegs) → update status bar
   (status bar implementation comes fully in Step 4,
   but wire up the call here so it's ready)
7. Update 'Last saved' indicator in footer
```

---

## Temporary Time Entry (placeholder for Step 8)

Tapping the Actual Finish cell should trigger a `prompt()` dialog
for now:

```javascript
// Temporary — will be replaced by custom keypad in Step 8
const input = prompt('Enter finish time (e.g. 3:35 PM):');
if (input) {
  // validate basic format, then save and recalculate
}
```

This lets the full calculation chain be tested end-to-end before
the custom keypad is built.

---

## Table Row Rendering

Update `renderTable()` to accept the computed leg array and render
each row using calculated values:

### Time cell styles
- **Start time:** Bold white if leg status is 'Actual' OR if the leg
  has officially started (i.e. the previous leg's actual finish is set,
  or it is Leg 1 and the race has started). Otherwise regular weight.
- **Est Finish:** Always italic, `#999` — never bold
- **Actual Finish cell:**
  - If `actualFinish` is set: bold white, shows actual time
  - If `actualFinish` is null: dashed border tap target, shows `—`

### Active leg
The current active leg (first leg with no actual finish) gets:
`outline: 2px solid #888`

---

## Status Bar (partial — full wiring in Step 4)

Wire up the render call now so it's ready for Step 4:

```javascript
function renderStatusBar(computedLegs) {
  // Current loop: first leg where actualFinish is null
  // → show 'leg# – runnerName'
  // Next runner: runner of the leg after current
  // Projected finish: finishTime of leg 24
  // Next start: startTime of the leg after current
  // Update DOM elements for all four values
}
```

Call `renderStatusBar()` after every `renderTable()` call.

---

## localStorage Structure for Race Legs

```javascript
// Key: 'raceLegs'
// Array of 24 objects — only mutable fields stored:
[
  { legNumber: 1, loop: 'Green', runner: 'Wendy', actualFinish: null },
  { legNumber: 2, loop: 'Yellow', runner: 'Kristine', actualFinish: null },
  // ... 24 total
]
```

Initialize `raceLegs` in localStorage on first load if not present.
Re-initialize runner names whenever config is saved (re-render flow).

---

## Verification Steps

Test the following before committing:

1. App loads — all 24 rows show correct estimated times based on
   default config (12:00/mi avg pace, 4/5/6 mile loops, 8:00 AM start)
2. Tap Actual Finish for Leg 1 — enter `9:50 AM` via prompt.
   Leg 1 shows bold `9:50 AM`. Leg 2 start updates to `9:50 AM`.
   All downstream estimated times cascade correctly.
3. Enter actual finish times for Legs 1–5 using the sample data.
   Verify each subsequent start time updates correctly.
4. Status values: Legs with actual finish show 'Actual'. First leg
   without actual shows 'Estimated (Baseline)' if no prior actuals,
   'Estimated (Live)' if prior actuals exist.
5. Clear an actual finish time (enter blank in prompt). Leg reverts
   to estimated. Downstream times recascade.
6. Refresh page — entered actual finish times persist from localStorage.
7. Open Settings, change avg pace, save — all estimated times update.
8. Verify midnight crossing: if estimated times cross midnight they
   display correctly as AM times (e.g. 1:06 AM not 25:06).

---

## Sample Data for Testing

Use this data to verify the calculation chain:

| Leg | Runner | Actual Finish |
|-----|--------|--------------|
| 1 | Wendy | 9:50 AM |
| 2 | Kristine | 11:08 AM |
| 3 | Scott | 12:20 PM |
| 4 | Jeff | 1:13 PM |
| 5 | Jason | 2:21 PM |
| 6 | Ray | 3:35 PM |

After entering these, Leg 7 start should be 3:35 PM and estimated
finish times for Legs 7–24 should cascade from there.

---

## Key Constraints

- No forecasting logic yet — all estimates use flat baseline pace only
- Midnight crossing must be handled correctly
- Full re-render on every change (Option A) — no targeted cell updates
- All time arithmetic in minutes internally, display as hh:mm AM/PM
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Live calculation chain across all 24 legs
- Real-time table re-render on every actual finish entry
- Correct midnight crossing handling
- Status values correctly assigned
- localStorage persistence of actual finish times
- Temporary prompt-based time entry for testing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
