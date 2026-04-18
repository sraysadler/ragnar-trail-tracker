# Build Task: Step 14 — Config Panel Keypad Integration

## Objective
Wire all tappable config fields to appropriate custom keypads. The time
entry keypad already exists from Steps 7+8. This step extends and adapts
it to handle the different input types required by the config panel fields,
and ensures all config values are correctly saved to localStorage when the
Settings panel is saved.

---

## Background

The config panel fields are currently styled as tappable (blue border,
dark background) but tapping them does nothing. This step makes them
fully interactive. Runner name fields are the only exception — they use
the system keyboard and are already working.

---

## Keypad Variants Required

Three keypad variants are needed. The existing time keypad is Variant A.
Two new variants must be built.

### Variant A — Time Keypad (already exists)
Used for: Race Start Time, Night Hours From, Night Hours To

```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[AM/PM] [ 0 ] [ ⌫ ]
```

- Same behavior as race table time entry
- Returns hh:mm AM/PM formatted string
- No changes needed to existing keypad — just wire config fields to it

### Variant B — Numeric Keypad with Decimal
Used for: Green Miles, Yellow Miles, Red Miles, Night Adjustment

```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[ . ] [ 0 ] [ ⌫ ]
```

- No AM/PM button — replaced by decimal point "."
- Entry left-to-right (natural number entry, not calculator style)
- Maximum 5 characters total (e.g. "100.0" or "5.25")
- Only one decimal point allowed — pressing "." when one already exists
  is ignored
- Display shows current value building up as digits are entered
- Units suffix shown in display only, not stored:
  - Miles fields: display shows e.g. "4.05 mi" — stores 4.05
  - Night Adjustment: display shows e.g. "5%" — stores 5
- Confirm saves the numeric value
- Cancel returns to config panel without saving
- Backspace removes last character

### Variant C — Pace Keypad
Used for: Estimated Team Average Pace

```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[ : ] [ 0 ] [ ⌫ ]
```

- No AM/PM button — replaced by colon ":"
- Format: MM:SS (e.g. 12:00 for 12 minutes per mile)
- Display shows MM:SS format with "/mi" suffix: e.g. "12:00 /mi"
- Entry behavior:
  - Up to 4 digits entered, displayed as MM:SS
  - First two digits = minutes, last two = seconds
  - Seconds cannot exceed 59 — reject digit if it would create invalid seconds
  - Example: typing 1, 2, 0, 0 → displays "12:00 /mi"
  - Example: typing 1, 1, 3, 0 → displays "11:30 /mi"
- Stored internally as decimal minutes (e.g. 12:30 stored as 12.5)
- Confirm saves the value
- Cancel returns to config panel without saving
- Backspace removes last digit

---

## Field-to-Keypad Mapping

| Config Field | Keypad Variant | Display Format | Stored Format |
|-------------|----------------|----------------|---------------|
| Race Start Time | A (Time) | h:mm AM/PM | String "h:mm AM/PM" |
| Green Loop Miles | B (Numeric) | "4.05 mi" | Number 4.05 |
| Yellow Loop Miles | B (Numeric) | "5.2 mi" | Number 5.2 |
| Red Loop Miles | B (Numeric) | "5.7 mi" | Number 5.7 |
| Avg Pace | C (Pace) | "12:00 /mi" | Number 12.0 |
| Night Adjustment | B (Numeric) | "5%" | Number 5 |
| Night Hours From | A (Time) | h:mm AM/PM | String "h:mm AM/PM" |
| Night Hours To | A (Time) | h:mm AM/PM | String "h:mm AM/PM" |
| Runner 1–8 Names | System keyboard | Text | String |

---

## Keypad Context for Config Fields

The keypad needs to know it was opened from the config panel (not the
race table or row detail modal) so it can:
- Return to the config panel on Confirm or Cancel
- Save the value to the correct config field
- Display the appropriate field label at the top

Extend the existing keypad context system:

```javascript
function openKeypad(context, legIndex = null, configField = null) {
  // context: 'table' | 'modal' | 'config'
  // configField: field key string e.g. 'raceStartTime', 'greenMiles'
}
```

When opened from config:
- Header shows the field name e.g. "Race Start Time" / "Green Miles"
- On Confirm: update the config panel display field with the new value
  (do NOT save to localStorage yet — that happens when Settings Save is tapped)
- On Cancel: return to config panel, no change

---

## Config Panel Display Updates

When a keypad value is confirmed for a config field:
1. Update the displayed value in the tappable field in the config panel
2. Store the pending value in a temporary config state object
3. Do NOT write to localStorage until the Settings Save button is tapped
4. When Save is tapped: write all pending values + existing values to
   localStorage, re-render the race table, close the config panel

This means the config panel maintains a "pending changes" state while open.
If the user taps Cancel/Close on the config panel without saving, all
pending keypad changes are discarded.

---

## Keypad Display Header

When opened from config, the keypad header should show the field name
instead of leg/runner information:

```
Race Start Time          ← field name
  9:00 AM               ← current value large
[ 1 ] [ 2 ] [ 3 ]
...
```

Reuse the existing keypad overlay HTML — just update the header text
and time/value display dynamically based on context.

---

## Validation

### Miles fields (Variant B)
- Must be a positive number greater than 0
- Maximum reasonable value: 20.0 (no trail loop is longer than this)
- If invalid on Confirm: flash display, do not save

### Pace field (Variant C)
- Minutes must be between 5 and 30 (reasonable trail running pace range)
- Seconds must be 00–59
- If invalid on Confirm: flash display, do not save

### Time fields (Variant A)
- Existing time keypad validation applies

### Night Adjustment field (Variant B)
- Must be between 0 and 50 (0% to 50% adjustment)
- Decimal values acceptable (e.g. 2.5%)
- If invalid on Confirm: flash display, do not save

---

## Config Panel Save Flow (updated)

When Settings Save is tapped:
1. Collect all current config panel values (pending keypad values +
   unchanged existing values)
2. Validate all values (runner names can be blank, all others must
   be valid)
3. Write complete config object to localStorage under `raceConfig`
4. Re-initialize `raceLegs` with updated runner names (preserving
   existing actual finish times)
5. Call `calculateLegData()` → `renderTable()` → `renderStatusBar()`
6. Close config panel
7. Update "Last saved" indicator

Note on step 4: if runner names change, the `runner` field in each
leg should update to the new name for legs that still have the default
runner (i.e. haven't been substituted). Legs where the runner was
manually changed via the Row Detail Modal should be left as-is.

---

## Verification Steps

### Variant A — Time fields
1. Open Settings. Tap Race Start Time field. Time keypad opens with
   current value pre-populated. Enter a new time (e.g. 9:00 AM).
   Confirm. Config panel updates to show 9:00 AM.
2. Tap Save. Close settings. Leg 1 start time in race table updates
   to 9:00 AM. All downstream estimates recalculate.
3. Tap Night Hours From. Change to 9:00 PM. Tap Night Hours To.
   Change to 6:00 AM. Save. Verify night adjustment now applies to
   legs starting between 9:00 PM and 6:00 AM only.
4. Cancel from time keypad returns to config panel with no change.

### Variant B — Numeric fields
5. Tap Green Miles. Numeric keypad opens showing "4.0 mi". Enter
   "4.05". Confirm. Config shows "4.05 mi". Save. Race table
   estimated durations for Green legs update.
6. Tap Night Adjustment. Enter "10". Confirm. Config shows "10%".
   Save. Night legs should show ~10% longer estimated durations.
7. Decimal point: enter "5" then "." then "7". Display shows "5.7".
   Enter "." again — second decimal point ignored.
8. Cancel returns to config panel with no change.

### Variant C — Pace field
9. Tap Avg Pace. Pace keypad opens showing current value. Enter
   1, 4, 0, 0. Display shows "14:00 /mi". Confirm. Config shows
   "14:00 /mi". Save. All Estimated (Baseline) legs update their
   estimated times based on the new pace.
10. Verify invalid seconds rejected: enter 1, 2, 6, 5. The "6" in
    seconds tens position creates 65 seconds — should be rejected.
11. Cancel returns to config with no change.

### Config save behavior
12. Make changes to multiple fields. Close Settings WITHOUT tapping
    Save. Reopen Settings. All fields show original values — pending
    changes were discarded.
13. Make changes to multiple fields. Tap Save. Race table updates
    correctly to reflect all changed values simultaneously.
14. Change runner names. Save. Race table updates with new names.
    Any legs where the runner was manually substituted via Row Detail
    Modal retain their substituted runner.

### Persistence
15. Change avg pace to 14:00. Save. Refresh browser. Avg pace still
    shows 14:00 in Settings. Race table estimated times still reflect
    14:00 pace.

---

## Key Constraints

- Keypad values are pending until Settings Save is tapped
- Cancel/Close on Settings discards all pending keypad changes
- Runner name fields continue using system keyboard — do not change
- Single HTML file — all new keypad variants added to existing file
- Reuse existing keypad overlay structure — no new overlays
- No external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- All tappable config fields opening appropriate keypad variant
- Variant B (numeric with decimal) built and working
- Variant C (pace MM:SS) built and working
- Config panel pending state managed correctly
- Settings Save correctly writes all values and re-renders table
- All validation working with appropriate error feedback
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
