# Build Task: Step 3 — Race Table with Auto-Generated Leg Assignments

## Objective
Convert the hardcoded race table HTML into a dynamically generated table
driven by config data. The 24-leg structure (loop colors, loop initials,
row backgrounds) is fixed and defined as a static lookup in the JavaScript.
Only the runner names are dynamic, populated from the config values saved
in localStorage.

---

## What Changes in This Step

### 1. Static leg definition array
Define a JavaScript constant — an array of 24 objects — that contains the
fixed, never-changing properties of each leg:

```javascript
const LEG_DEFINITIONS = [
  { leg: 1,  loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 2,  loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 3,  loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 4,  loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 5,  loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 6,  loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 7,  loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 8,  loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 9,  loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 10, loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 11, loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 12, loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 13, loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 14, loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 15, loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 16, loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 17, loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 18, loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 19, loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 20, loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 21, loop: 'Red',    initial: 'R', rowColor: '#330000' },
  { leg: 22, loop: 'Green',  initial: 'G', rowColor: '#003300' },
  { leg: 23, loop: 'Yellow', initial: 'Y', rowColor: '#444400' },
  { leg: 24, loop: 'Red',    initial: 'R', rowColor: '#330000' },
];
```

This array is defined once and never changes. It is the single source of
truth for leg structure.

### 2. Runner assignment mapping
The runner for each leg is determined by the fixed Ragnar rotation:
- Leg 1 → Runner 1, Leg 2 → Runner 2 ... Leg 8 → Runner 8
- Leg 9 → Runner 1, Leg 10 → Runner 2 ... Leg 16 → Runner 8
- Leg 17 → Runner 1, Leg 18 → Runner 2 ... Leg 24 → Runner 8

This mapping is also fixed and can be computed as:
`runnerIndex = (leg - 1) % 8`

Runner names come from config (localStorage). If a runner name is blank,
the cell renders empty.

### 3. Table generation function
Write a `renderTable()` function that:
- Reads the 8 runner names from localStorage (or defaults if not set)
- Iterates over `LEG_DEFINITIONS`
- For each leg, generates a table row with:
  - Correct row background color from `LEG_DEFINITIONS`
  - Leg number
  - Runner name (from config, using the rotation mapping above)
  - Loop initial (G/Y/R) from `LEG_DEFINITIONS`
  - Placeholder Start time (keep existing placeholder data for now)
  - Placeholder Est Finish time (keep existing placeholder data for now)
  - Placeholder Actual Finish cell (keep existing dashed tap target style)

### 4. Config Save wires up
When the user taps Save in the Settings panel:
- Write all config values to localStorage under the key `raceConfig`
- Close the Settings overlay
- Call `renderTable()` to re-render the table with updated runner names
- Table re-renders from scratch — no need to preserve state at this stage

### 5. On app load
Call `renderTable()` on page load so the table always reflects whatever
is currently in localStorage (or defaults if nothing is saved yet).

---

## localStorage Structure for Config (from Data Model)

```javascript
// Key: 'raceConfig'
{
  teamName: 'Party Pace',
  eventName: 'Ragnar Trail Atlanta 2026',
  raceStartTime: '8:00 AM',
  runner1: 'Wendy',
  runner2: 'Kristine',
  runner3: 'Scott',
  runner4: 'Jeff',
  runner5: 'Jason',
  runner6: 'Ray',
  runner7: 'Jenny',
  runner8: 'Tim',
  greenMiles: 4.0,
  yellowMiles: 5.0,
  redMiles: 6.0,
  avgPace: 12.0,
  nightAdjustment: 5,
  nightAdjustmentType: 'percent',
  nightStart: '8:00 PM',
  nightEnd: '7:00 AM'
}
```

---

## What Does NOT Change in This Step

- Start times, Est Finish times remain as placeholder static data
- Actual Finish cells remain as dashed tap targets with no functionality
- No time calculations of any kind
- No forecasting logic
- The visual appearance of the table should be identical to Step 1/2 —
  only the underlying generation mechanism changes

---

## Verification Steps

After building, verify the following before committing:

1. Open the app — table renders correctly with default runner names
2. Open Settings, clear all runner names, tap Save — table renders
   with empty runner cells but correct loop colors and leg numbers
3. Open Settings, enter 8 new runner names, tap Save — table updates
   immediately with the new names in the correct leg order
4. Refresh the page — runner names persist from localStorage
5. The yellow row color should be `#444400` (matching what was set
   in Step 1)

---

## Key Constraints

- `LEG_DEFINITIONS` array is a constant — never modified at runtime
- No frameworks, no external libraries
- Single HTML file — all JavaScript in a `<script>` block
- Do not recalculate or derive loop colors or loop order from any
  config value — they come only from `LEG_DEFINITIONS`

---

## Deliverable

Updated `ragnar-trail-tracker.html` with dynamic table generation.
Visual output should be identical to the end of Step 2.
Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
