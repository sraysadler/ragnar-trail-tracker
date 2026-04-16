# Build Task: Step 9 — Row Detail Modal

## Objective
Build the Row Detail Modal — a full-screen overlay that opens when the
user taps anywhere on a race table row except the Actual Finish cell.
It serves as both a detail view for all leg data and the edit interface
for Actual Finish and Runner substitutions.

---

## Trigger

- Tapping the Actual Finish cell → opens time entry keypad (already built)
- Tapping anywhere else on a row → opens Row Detail Modal for that leg

Make sure the tap zones don't conflict. The Actual Finish cell should
have its own click handler. Everything else in the row triggers the modal.

---

## Modal Structure

Full-screen overlay, same approach as the keypad and Settings panel:

```
position: fixed
top: 0, left: 0
width: 100%, height: 100%
background: #111
z-index: 900 (below keypad at 1000, above main table)
display: flex
flex-direction: column
```

### Header
- Left: "Leg [N] — [Loop Color]" e.g. "Leg 6 — Red"
  - Font: same as other panel headers, white, 500 weight
- Right: "✕" close button — same style as other close buttons
- Background: `#1a1a1a`, border-bottom: `1px solid #333`

### Body (scrollable if needed)
All detail fields listed vertically. Each field is a row with:
- Left: field label (muted, `#666`, 9–10px)
- Right: field value (white or styled per type)
- Bottom border: `1px solid #1e1e1e`
- Comfortable padding for touch

### Footer
Two equal-width buttons:
- **Cancel** — muted style, discards any unsaved changes, closes modal
- **Save changes** — green style (`#4caf50`), saves editable fields

---

## Display Fields (read-only)

Show these fields in this order:

| Field | Value | Style |
|-------|-------|-------|
| Runner | Current runner name | White. Dropdown — editable (see below) |
| Loop miles | e.g. "6.0 mi" | White |
| Start | Start time | Bold white (confirmed) or italic `#999` |
| Est finish | Estimated finish time | Italic `#999` always |
| Actual finish | Actual time or "—" | Blue tappable box (see below) |
| Status | Actual / Estimated (Baseline) / Estimated (Live) | Color-coded (see below) |
| Actual duration | e.g. "1:14:00" or "—" | White or `#555` if null |
| Actual pace | e.g. "12:20/mi" or "—" | White or `#555` if null |
| Predicted pace | e.g. "12:36/mi" | White |
| Est duration | e.g. "1:20:12" | Italic `#999` |

### Status color coding
- `Actual` → green text (`#4caf50`)
- `Estimated (Baseline)` → muted gray (`#888`)
- `Estimated (Live)` → blue (`#4a9eff`)

---

## Editable Fields

### Actual Finish
Displayed as a tappable blue-bordered box:
- Background: `#0a1a2a`
- Border: `1px solid #4a9eff`
- Border-radius: 3px
- If no actual finish entered: shows Est Finish time in italic `#555`
  as a starting suggestion
- If actual finish entered: shows the actual time in white, non-italic
- Tapping this box opens the time entry keypad
- When keypad returns a confirmed time, it updates this field AND
  saves to the leg data immediately (same as tapping from the table)
- Context for keypad cancel: return to modal (not to table)

### Runner
Displayed as a dropdown selector:
- Shows current runner name
- Dropdown contains all 8 runner names from config
- Changing the runner here updates the leg's runner assignment
- This change takes effect immediately on Save
- Background: `#2a2a2a`, border: `1px solid #444`

---

## Save Behavior

When Save changes is tapped:
1. If Runner was changed: update `raceLegs[legIndex].runner`
2. If Actual Finish was changed via keypad: already saved to
   `raceLegs[legIndex].actualFinish` when keypad confirmed
   (keypad saves immediately — modal Save just closes the modal)
3. Write `raceLegs` to localStorage
4. Close modal
5. Call `calculateLegData()` → `renderTable()` → `renderStatusBar()`
6. Update Last Saved indicator

Note: Actual Finish changes made via the keypad inside the modal
are saved immediately when the keypad Confirms — they do not wait
for the modal Save button. The modal Save button is primarily for
Runner substitution changes. This is intentional.

---

## Cancel Behavior

- If no changes were made: close modal instantly, return to table
- If Runner was changed but not saved: show confirmation prompt —
  "Close without saving changes?"
  - Confirm: discard runner change, close modal
  - Cancel: stay in modal
- Actual Finish changes made via keypad are already saved —
  Cancel does not undo them

---

## Closing the Modal

- ✕ button in header: same behavior as Cancel button
- Tapping outside the modal (on the dimmed background): same as Cancel

---

## Clearing Actual Finish from Modal

If user opens keypad from inside the modal and clears the time
(empty buffer + Confirm):
- Show confirmation: "Clear finish time for Leg [N]?
  It will revert to estimated."
- On confirm: set actualFinish to null, update modal display,
  recalculate immediately
- Actual finish field in modal reverts to showing Est Finish
  time in italic blue box

---

## Time Format in Modal

All times displayed as `h:mm AM/PM` (e.g. `9:50 AM`, `11:08 AM`)
Durations displayed as `h:mm:ss` (e.g. `1:14:00`)
Pace displayed as `MM:SS/mi` (e.g. `12:20/mi`)

---

## Visual Reference

Refer to wireframe screen 2 in `Race_Tracker_Wireframes_v1.html`
in the /docs folder of the repository for the layout reference.

---

## Verification Steps

1. **Open modal:**
   Tap anywhere on a row (not the Actual Finish cell).
   Modal opens showing correct leg number and loop color in header.

2. **All fields display correctly:**
   Verify each field shows the right value and styling.
   For a completed leg: actual duration and pace should show.
   For an incomplete leg: those fields show "—".

3. **Status color coding:**
   Verify Actual shows green, Estimated (Baseline) shows gray,
   Estimated (Live) shows blue.

4. **Actual Finish tappable box:**
   For incomplete leg: tap the blue box. Keypad opens.
   Enter a time. Keypad confirms. Modal updates to show new time.
   Actual duration and pace recalculate and display in modal.

5. **Cancel from keypad returns to modal:**
   Open keypad from modal. Press Cancel. Returns to modal,
   not to the table.

6. **Runner substitution:**
   Open modal for a leg. Change Runner dropdown to a different name.
   Tap Save changes. Table updates with new runner name.
   Forecasting for that runner's future legs updates accordingly.

7. **Cancel discards runner change:**
   Open modal. Change Runner. Tap Cancel (or ✕).
   Confirmation appears. Confirm discard. Runner reverts.
   Table unchanged.

8. **Close on background tap:**
   Open modal. Tap outside it (on the dimmed background behind).
   Modal closes (with unsaved changes prompt if applicable).

9. **Predicted pace visible:**
   Open modal for a future leg where the runner has completed
   prior legs. Predicted pace should show their calculated pace,
   not the baseline.

10. **Full flow test:**
    Enter actual finish times for Legs 1–8 using the keypad
    opened from the modal. Verify each saves correctly and the
    table updates behind the modal.

---

## Key Constraints

- Tapping Actual Finish cell in table → keypad (not modal)
- Tapping anywhere else on row → modal
- Actual Finish changes in modal save immediately via keypad confirm
- Runner changes in modal save only on modal Save
- Cancel does not undo already-confirmed keypad entries
- Single HTML file, no external dependencies
- Modal z-index below keypad (keypad opens on top of modal)

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Row Detail Modal opening on row tap
- All fields displaying correctly with proper styling
- Actual Finish editable via keypad from within modal
- Runner substitution dropdown working
- Save / Cancel / ✕ all behaving correctly
- Keypad cancel returning to modal when opened from modal

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
