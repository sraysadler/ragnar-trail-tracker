# Build Task: Steps 7+8 — Time Entry Keypad

## Objective
Replace the temporary `prompt()` time entry with a custom numeric keypad.
Wire the keypad to open when the Actual Finish cell is tapped in the race
table. The keypad is a full-screen overlay that handles all time input
for the app.

---

## Keypad Layout

The keypad is a full-screen overlay consisting of three sections stacked
vertically:

### Section 1 — Time Display
Large time display above the keypad grid showing the time currently
being entered.
- Background: `#0a0a0a`
- Border-bottom: `1px solid #333`
- Time value: large, white, 500 weight, letter-spacing
- Format: `hh:mm AM/PM`
- Padding: comfortable vertical padding

### Section 2 — Keypad Grid
4 rows × 3 columns of equal-size buttons. All buttons the same size.
No button is wider than any other.

```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[AM/PM] [ 0 ] [ ⌫ ]
```

Button styling:
- Background: `#1a1a1a`
- Separator lines: `1px solid #333` (use gap or borders between buttons)
- Number buttons: font-size 18px, color `#ddd`
- AM/PM button: font-size 11px, color `#999` — same physical size as others
- Backspace (⌫): color `#777`
- All buttons equal width and height — critical requirement

### Section 3 — Action Bar
Two equal-width buttons across the full width:
- **Cancel** — left, muted style (`#777`)
- **Confirm** — right, green style (`#4caf50`, 500 weight)

---

## Time Display Behavior

### On open
- If the Actual Finish cell already has a time entered: display that time
- If the cell is empty (no actual finish yet): display the current
  device time
- The displayed time is the "current value" — it gets cleared and
  rebuilt as the user types

### Digit entry (right-to-left, calculator style)
Digits fill from the right and shift left as new digits are entered.

The display always shows a valid-looking time as digits are entered:

```
Initial (empty entry): 12:00 AM  ← or current time
After pressing 3:      12:03 AM  ← 3 appears in minutes ones place
After pressing 3, 5:   12:35 AM  ← shifts left
After pressing 3, 5, 7: 7:35 AM  ← shifts into hours
After pressing 3, 5, 7, 1: 1:35 AM ← wait — see note below
```

**Digit buffer approach:**
Maintain an array of up to 4 digits. As digits are added they push
left through the positions: MM ones → MM tens → HH ones → HH tens.

```
Buffer: []         → display: 12:00
Buffer: [3]        → display: 12:03
Buffer: [3,5]      → display: 12:35
Buffer: [3,5,7]    → display:  7:35
Buffer: [3,5,7,1]  → display:  1:35 (reading as 1:35, not 13:5)
```

Maximum 4 digits in buffer. Once 4 digits are entered, additional
digit presses are ignored until backspace clears one.

### Backspace
Removes the last digit entered (pops from right of buffer).

```
Buffer: [3,5,7,1]  → backspace → [3,5,7]  → display: 7:35
```

### AM/PM display
- Defaults to match current device time (AM if morning, PM if afternoon)
- Updates automatically if the hour value built from the digit buffer
  would be 13 or higher (24-hour input auto-converts)
- AM/PM button tap toggles between AM and PM manually at any point

### 24-hour input handling
If the hour portion of the built time exceeds 12:
- Subtract 12 from the hour
- Set AM/PM to PM automatically
- Example: buffer [2,3,1,5] → raw = 23:15 → display = 11:15 PM

### Hour value = 12
- Set AM/PM to PM (noon)
- Example: buffer [1,2,0,0] → 12:00 PM

---

## Keypad Context

The keypad needs to know which leg it was opened for so it can save
the time to the correct leg when Confirm is tapped.

Pass the leg index (0–23) when opening the keypad:

```javascript
function openKeypad(legIndex) {
  currentKeypadLeg = legIndex;
  // populate display with existing actual finish or current time
  // show keypad overlay
}
```

---

## Confirm Behavior

When Confirm is tapped:
1. Read the current time from the display
2. Validate it is a real time (hours 1–12, minutes 00–59)
3. Save to `raceLegs[currentKeypadLeg].actualFinish`
4. Write `raceLegs` to localStorage
5. Close keypad overlay
6. Call `calculateLegData()` → `renderTable()` → `renderStatusBar()`
7. Update Last Saved indicator

If the time is invalid (e.g. minutes > 59), do not save —
show a brief visual error state on the display (e.g. flash red)
and let the user correct it.

---

## Cancel Behavior

- If opened from the race table → close keypad, return to table
- If opened from the Row Detail Modal → close keypad, return to modal
  (modal will be built in Step 9 — for now, always return to table)
- Do not save any changes

---

## Clearing an Existing Time

If the Actual Finish cell already has a time and the user opens the
keypad, they can clear it by using backspace to remove all 4 digits.
If Confirm is tapped with an empty buffer:
- Show confirmation prompt: "Clear finish time for Leg [N]?
  It will revert to estimated."
- On confirm: set `actualFinish` to null, recalculate, re-render
- On cancel: return to keypad without saving

---

## Wiring to Race Table

Remove the existing `prompt()` call entirely.

Tapping the Actual Finish cell (the dashed `—` box or existing time)
in any table row calls `openKeypad(legIndex)`.

The Actual Finish cell must have a clearly tappable style:
- Minimum touch target: 44px height (already set from Step 1)
- Dashed border when empty, bold white when filled
- Cursor pointer

---

## Keypad Overlay Structure

The keypad takes over the full screen when open:

```
position: fixed
top: 0, left: 0
width: 100%, height: 100%
background: #000
z-index: 1000
display: flex
flex-direction: column
```

The three sections (display, grid, action bar) fill the screen
vertically. The grid should take up the majority of the height
so buttons are large enough for comfortable touch use.

---

## Visual Reference

Refer to wireframe screen 3 in `Race_Tracker_Wireframes_v1.html`
in the /docs folder of the repository for the exact layout.

---

## Verification Steps

1. **Open keypad from empty cell:**
   Tap a dashed `—` cell. Keypad opens showing current device time.
   
2. **Digit entry:**
   Press 9, 5, 0. Display should show `9:50 AM` (or PM depending
   on current time/AM-PM state). Press Confirm. Table updates,
   cell shows bold `9:50 AM`, calculation chain recalculates.

3. **Open keypad from filled cell:**
   Tap the cell just filled. Keypad opens showing `9:50 AM`.
   Press backspace 3 times. Display shows `12:00 AM` (or similar
   with only one digit remaining). Press Confirm with new value.

4. **24-hour input:**
   Press 2, 3, 1, 5. Display should show `11:15 PM`.
   Press Confirm. Cell saves as `11:15 PM`.

5. **AM/PM toggle:**
   Press 9, 5, 0. Display shows `9:50 AM`.
   Tap AM/PM button. Display changes to `9:50 PM`.
   Press Confirm. Cell saves as `9:50 PM`.

6. **Cancel:**
   Open keypad, enter some digits, press Cancel.
   Table cell is unchanged.

7. **Clear existing time:**
   Open a cell with a time entered. Press backspace 4 times
   (empty buffer). Press Confirm. Prompt appears:
   "Clear finish time for Leg [N]? It will revert to estimated."
   Confirm clear. Cell reverts to dashed `—` and estimates update.

8. **Calculation chain:**
   Enter actual finish times for Legs 1–5 using the keypad.
   Verify downstream times update correctly after each entry.

9. **Full entry test:**
   Enter all 24 actual finish times from the 2025 sample data.
   Verify Race Complete state triggers after Leg 24.

---

## Sample Data for Full Entry Test

| Leg | Actual Finish |
|-----|--------------|
| 1  | 9:50 AM  |
| 2  | 11:08 AM |
| 3  | 12:20 PM |
| 4  | 1:13 PM  |
| 5  | 2:21 PM  |
| 6  | 3:35 PM  |
| 7  | 4:21 PM  |
| 8  | 5:02 PM  |
| 9  | 6:10 PM  |
| 10 | 6:55 PM  |
| 11 | 7:53 PM  |
| 12 | 9:06 PM  |
| 13 | 9:56 PM  |
| 14 | 11:11 PM |
| 15 | 12:26 AM |
| 16 | 1:06 AM  |
| 17 | 2:20 AM  |
| 18 | 3:33 AM  |
| 19 | 4:20 AM  |
| 20 | 5:25 AM  |
| 21 | 6:55 AM  |
| 22 | 7:49 AM  |
| 23 | 8:56 AM  |
| 24 | 9:56 AM  |

---

## Key Constraints

- All keypad buttons must be equal size — AM/PM is NOT wider
- No system keyboard — custom keypad only
- Single HTML file — keypad HTML/CSS/JS added to existing file
- No external dependencies
- Remove prompt() completely — no trace of it should remain

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Full custom keypad overlay
- Tapping Actual Finish cell opens keypad
- Digit entry, backspace, AM/PM toggle all working
- 12-hour and 24-hour input both handled
- Confirm saves and recalculates
- Cancel discards
- Clear existing time with confirmation prompt
- prompt() removed entirely

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
