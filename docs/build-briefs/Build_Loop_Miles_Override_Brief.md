# Build Task: Loop Miles Override

## Objective
Allow the loop miles for any completed leg to be overridden on a per-leg
basis via the Row Detail Modal. This handles edge cases where a runner
ran more or fewer miles than the configured default for that loop color
(e.g. getting lost and running an extra mile). The override corrects
that runner's actual pace and all downstream estimates for their
future legs.

---

## Data Model Change

Add a new optional field to each leg object in `raceLegs`:

```javascript
{
  legNumber: 1,
  loop: 'Green',
  defaultRunner: 'Wendy',
  runner: 'Wendy',
  actualFinish: null,
  overrideMiles: null  // ← new field. null = use config default
}
```

**`overrideMiles`** is `null` by default. When set, it replaces the
config loop distance for all calculations on that specific leg only.
All other legs continue to use the config default unless they also
have an override.

---

## Effective Miles Helper

Add a helper function that returns the miles to use for any given leg:

```javascript
function getEffectiveMiles(leg, config) {
  if (leg.overrideMiles !== null && leg.overrideMiles !== undefined) {
    return leg.overrideMiles;
  }
  if (leg.loop === 'Green') return config.greenMiles;
  if (leg.loop === 'Yellow') return config.yellowMiles;
  if (leg.loop === 'Red') return config.redMiles;
}
```

Replace all direct config miles lookups in `calculateLegData()` with
calls to `getEffectiveMiles()`. This ensures every calculation
automatically uses the override when present.

---

## Formula Impact

### Actual pace (completed legs)
```
actualPace = actualDuration ÷ getEffectiveMiles(leg, config)
```
If a runner ran 7 miles instead of 6, their actual pace reflects
the true 7-mile distance, giving an accurate pace for pace history.

### Pace history storage
The pace stored in the runner's history is the actual pace calculated
with effective miles. No special treatment needed — using effective
miles already produces the correct pace.

### Night pace normalization
Night normalization uses the same actual pace (already calculated with
effective miles). No change needed to normalization logic.

### Estimated duration
```
estimatedDuration = predictedPace × getEffectiveMiles(leg, config)
```
If an override is set on an incomplete leg (edge case — normally not
possible per UI rules), it uses override miles. For all normal cases,
uses config default.

### All other calculations
Any formula that currently reads `config.greenMiles`, `config.yellowMiles`,
or `config.redMiles` directly for a specific leg should instead call
`getEffectiveMiles(leg, config)`. This includes:
- Loop miles display in Row Detail Modal
- Loop miles display in Runner Stats modal
- Estimated duration
- Actual pace
- Predicted pace
- CSV export loop miles column

---

## UI — Row Detail Modal

### Loop Miles field appearance
The Loop Miles field in the Row Detail Modal should have a dotted border
to signal it is interactable (like the dashed border on unfinished
Actual cells in the main table):

```css
border: 1px dotted #555;
border-radius: 3px;
padding: 3px 6px;
```

This dotted border should always be present on the Loop Miles field
in the modal, regardless of whether the leg is complete or not —
but the press-and-hold action is only available on completed legs
(see Availability below).

### Color when overridden
If `overrideMiles` is set for this leg, display the miles value
in green (`#4caf50`) instead of the default white.
If using config default, display in normal white.

Example: overridden → `7.0 mi` in green
         default    → `6.0 mi` in white

### Press-and-hold interaction
**Availability:** Only available when the leg has an actual finish
time entered (`actualFinish` is not null). Do not allow override
on incomplete legs.

**Trigger:** User presses and holds the Loop Miles field for 3 seconds.

**Implementation:**
```javascript
let holdTimer = null;

milesField.addEventListener('pointerdown', () => {
  if (!leg.actualFinish) return; // only allow on completed legs
  holdTimer = setTimeout(() => {
    openMilesKeypad(legIndex);
  }, 3000);
});

milesField.addEventListener('pointerup', () => clearTimeout(holdTimer));
milesField.addEventListener('pointerleave', () => clearTimeout(holdTimer));
```

**Visual feedback during hold:** Optionally pulse or briefly highlight
the field after 1.5 seconds to indicate the hold is registered and
the keypad is about to open.

### Keypad on open
Opens the existing Variant B numeric keypad (implied decimal,
right-to-left entry) — same keypad used for loop distances in Settings.

Pre-populate with:
- The current `overrideMiles` value if one is set, OR
- The config default miles for that loop color if no override exists

Context label at top of keypad: "Loop Miles — Leg [N]"

### On Confirm
1. Save the entered value to `raceLegs[legIndex].overrideMiles`
2. Write `raceLegs` to localStorage
3. Close keypad, return to Row Detail Modal
4. Re-run `calculateLegData()` → `renderTable()` → `renderStatusBar()`
5. Refresh Row Detail Modal to show updated values (miles in green,
   updated actual pace, updated predicted pace)
6. Update Last Saved indicator

### On Cancel
Return to Row Detail Modal with no change.

### Resetting the override
No separate reset button. User holds 3 seconds again and enters the
config default value for that loop color. This sets `overrideMiles`
to that value — functionally equivalent to the default.

Alternatively: if the entered value exactly matches the config default,
set `overrideMiles` back to `null` to keep the data clean.

---

## UI — Runner Stats Modal

### Asterisk on adjusted paces
For any completed leg where `overrideMiles` is not null, append an
asterisk immediately after the pace value in the Runner Stats modal:

```
12:30/mi*
```

No space between the pace and the asterisk. Color: same as the pace
text (bold white for completed legs).

No asterisk on estimated/future leg paces.

No footnote or explanation needed in the modal — the asterisk is the
signal.

---

## CSV Export

The Loop Miles column in the [Race Data] section should export
`getEffectiveMiles(leg, config)` for each leg — the override value
if set, config default otherwise.

No special flag or column needed to indicate an override in the CSV.
The miles value is simply correct.

---

## Verification Steps

Setup: Use Party Pace config, start time 9:00 AM, 4/5/6 mile loops.
Enter actual finish times for Legs 1–9 using 2025 sample data.

**Test 1 — Availability on incomplete leg:**
Open Row Detail Modal for Leg 10 (incomplete).
Press and hold the Loop Miles field for 3+ seconds.
Nothing should happen — keypad should NOT open on incomplete legs.

**Test 2 — Override on completed leg:**
Open Row Detail Modal for Leg 9 (Wendy, Red, completed).
Loop Miles shows "6.0 mi" in white with dotted border.
Press and hold for 3 seconds.
Keypad opens pre-populated with 6.0.
Enter 7.0 (type 7, 0). Confirm.
Modal updates: Loop Miles shows "7.0 mi" in green.
Actual pace recalculates using 7.0 miles (should be lower than before).

**Test 3 — Pace history correction:**
After Test 2, open Row Detail Modal for Leg 17 (Wendy's next leg).
Predicted pace should now reflect Wendy's corrected pace from Leg 9
(calculated with 7.0 miles), not the original inflated pace.
Estimated finish time for Leg 17 should update accordingly.

**Test 4 — Other runners unaffected:**
Open Row Detail Modal for any other runner's leg.
Their loop miles should show the config default (e.g. 6.0 for Red).
Their estimates should be unchanged.

**Test 5 — Runner Stats asterisk:**
Open Runner Stats modal.
Wendy's Leg 9 pace should show an asterisk: e.g. `10:45/mi*`
No other paces should have an asterisk.

**Test 6 — Persistence:**
Refresh the browser.
Reopen Row Detail Modal for Leg 9.
Override miles should still show 7.0 mi in green — persisted in
localStorage.

**Test 7 — CSV export:**
Export CSV.
Leg 9 in the Race Data section should show 7.0 in the Loop Miles
column, not 6.0.

**Test 8 — Reset override:**
Open Row Detail Modal for Leg 9.
Press and hold Loop Miles for 3 seconds.
Enter 6.0 (type 6, 0). Confirm.
Miles returns to white (no longer green).
Actual pace recalculates with 6.0 miles.
`overrideMiles` set back to null in localStorage.

---

## Key Constraints

- Override applies to one leg only — all other legs use config default
- Override only available on completed legs (actualFinish not null)
- `getEffectiveMiles()` helper used everywhere miles are read per leg
- Pace history automatically correct because actual pace uses
  effective miles — no special handling needed
- No visual change on main race table
- Asterisk in Runner Stats on overridden leg paces only
- CSV exports effective miles (override if set, default otherwise)
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- `overrideMiles` field in raceLegs data structure
- `getEffectiveMiles()` helper used throughout calculateLegData()
- Row Detail Modal: dotted border on Loop Miles, green color when
  overridden, press-and-hold opens distance keypad
- Runner Stats: asterisk on paces from overridden legs
- CSV export: uses effective miles
- localStorage persistence of override
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
