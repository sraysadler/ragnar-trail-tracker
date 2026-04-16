# Build Task: Step 4 — Status Bar and Footer (Fully Wired)

## Objective
Wire up all four status bar values and the footer Last Saved indicator
using live data from the calculation chain built in Step 5. The
`renderStatusBar()` function stub was created in Step 5 — this step
completes it fully.

---

## Status Bar Values

The status bar has two rows:

### Row 1 — Projected Team Finish (full width green bar)
- Value: `finishTime` of Leg 24 from the computed leg array
- This is either Leg 24's actual finish (if entered) or its estimated
  finish time
- Display format: hh:mm AM/PM
- Label: "PROJECTED TEAM FINISH" (uppercase, small, muted green)
- Value: larger green text (`#7dcc7d`, 500 weight)

### Row 2 — Three equal columns

**Current Loop**
- Label: "CURRENT LOOP" (small, muted, `#555`)
- Value: First leg in the computed array where `actualFinish` is null
- Display format: `[legNumber] – [runnerName]`
- Example: `6 – Ray`

**Next Runner**
- Label: "NEXT RUNNER" (small, muted, `#555`)
- Value: Runner name of the leg immediately after the current loop leg
- Example: `Jenny`

**Next Start**
- Label: "NEXT START" (small, muted, `#555`)
- Value: `startTime` of the leg immediately after the current loop leg
- Example: `3:35 PM`
- Display format: hh:mm AM/PM

---

## Race Complete State

Triggered when all 24 legs have `actualFinish` set (not null).

When Race Complete:
- Row 1: Show actual finish time of Leg 24 — label changes to
  "FINAL TEAM FINISH"
- Row 2 col 1: Show "Complete" instead of a leg number
- Row 2 col 2: Show "—"
- Row 2 col 3: Show "—"

Race Complete reverts automatically if any actual finish is cleared —
the status bar recalculates on every render, so this is handled
naturally by the existing render flow.

---

## Current Time Display

The header already shows a static "2:24 PM" placeholder. Wire this
up to show the actual current device time, updating every minute.

```javascript
function updateCurrentTime() {
  const now = new Date();
  let hours = now.getHours();
  const minutes = now.getMinutes().toString().padStart(2, '0');
  const ampm = hours >= 12 ? 'PM' : 'AM';
  hours = hours % 12 || 12;
  document.getElementById('current-time').textContent =
    `${hours}:${minutes} ${ampm}`;
}

// Call on load and every 60 seconds
updateCurrentTime();
setInterval(updateCurrentTime, 60000);
```

---

## Last Saved Indicator

Located in the footer center. Updates to current time whenever:
- An actual finish time is entered or cleared
- Config is saved from the Settings panel

Format: `Last saved: [hh:mm AM/PM]`

On first load with no localStorage data: show `Last saved: —`

---

## Edge Cases

**No actual finish times entered yet (race not started):**
- Current Loop: `1 – [Runner 1 name]`
- Next Runner: `[Runner 2 name]`
- Next Start: Race start time from config
- Projected Finish: Estimated finish of Leg 24 based on baseline pace

**All legs complete (Race Complete):**
- See Race Complete State above

**Leg 24 is the current loop (last runner out):**
- Next Runner: `—`
- Next Start: `—`

**Runner name is blank (config not fully set up):**
- Show `—` for any value that would display a blank runner name

---

## Render Flow (no changes needed, just confirming)

The existing render flow from Step 5 already calls `renderStatusBar()`.
This step just completes that function. No changes to the render
trigger sequence are needed:

```
1. Any actual finish entered or cleared
2. calculateLegData() → computed legs
3. renderTable(computedLegs)
4. renderStatusBar(computedLegs)   ← this step completes this function
5. updateLastSaved()
```

---

## Verification Steps

Test the following before committing:

1. App loads with no actual finish times — status bar shows Leg 1
   and Runner 1 as current, Runner 2 as next, race start time as
   next start, and baseline estimated finish for Leg 24 as projected
   finish
2. Enter actual finish for Leg 1 — Current Loop updates to Leg 2,
   Next Runner updates to Runner 3, Next Start updates to Leg 2's
   start time, Projected Finish updates
3. Enter actual finishes for all 24 legs — Race Complete state
   triggers, "FINAL TEAM FINISH" shows, "Complete" and "—" values
   appear in row 2
4. Clear Leg 24's actual finish — Race Complete reverts, normal
   status bar values restore
5. Current time in header updates correctly and matches device time
6. Last Saved updates when actual finish is entered and when config
   is saved
7. Test with Leg 24 as current loop — Next Runner and Next Start
   show "—"

---

## Key Constraints

- No new visual elements — this step only wires up existing DOM
  elements that are already in the HTML
- All values derive from the computed leg array — no separate
  state tracking for status bar
- Race Complete is derived, not stored — always calculated fresh
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- All four status bar values live and updating in real time
- Current time in header updating every minute
- Last Saved indicator updating on every entry and config save
- Race Complete state triggering and reverting correctly

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
