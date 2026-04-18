# Build Task: Step 15 — Manual CSV Export

## Objective
Wire up the Export button in the footer to generate and download a
complete CSV file of the current race state. The `generateCSV()` function
should be built as a shared function used by both this manual export and
the automatic file backup in Step 16.

---

## Export Trigger

The Export button is already visible in the footer. Tapping it should:
1. Call `generateCSV()` to build the CSV string
2. Trigger a file download in the browser
3. Show a brief confirmation in the footer: "Exported successfully"
   that fades after 2 seconds

---

## File Download

Use the standard browser download approach:

```javascript
function downloadCSV(csvString) {
  const blob = new Blob([csvString], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = buildExportFilename();
  a.click();
  URL.revokeObjectURL(url);
}

function buildExportFilename() {
  const now = new Date();
  const YYYY = now.getFullYear();
  const MM = String(now.getMonth() + 1).padStart(2, '0');
  const DD = String(now.getDate()).padStart(2, '0');
  const HH = String(now.getHours()).padStart(2, '0');
  const mm = String(now.getMinutes()).padStart(2, '0');
  return `ragnar_tracker_${YYYY}${MM}${DD}_${HH}${mm}.csv`;
}
```

---

## generateCSV() Function

Build this as a standalone shared function. It reads from the current
app state (config + computed leg data) and returns a complete CSV string.

The CSV has three sections in this order:

### Section 1 — Race Summary
```
[Race Summary]
Team Name,Event,Race Start,Finish Time,Total Duration,Status
Party Pace,Ragnar Trail Atlanta 2025,9:00 AM,9:56 AM,24:56,Complete
```

**Field definitions:**
- Team Name: from config
- Event: from config eventName
- Race Start: from config raceStartTime
- Finish Time: finishTime of Leg 24 (actual if complete, estimated if not)
- Total Duration: time from race start to Leg 24 finish, formatted as
  h:mm (e.g. "24:56"). Blank if race has not started.
- Status: "Complete" if all 24 legs have actual finish times,
  otherwise "In Progress"

### Section 2 — Race Data
```
[Race Data]
Leg,Runner,Loop,Loop Miles,Start,Est Finish,Actual Finish,Status,Actual Duration,Actual Pace,Predicted Pace,Est Duration
1,Wendy,Green,4,9:00 AM,9:48 AM,9:50 AM,Actual,0:50,12:30/mi,12:00/mi,0:48
2,Kristine,Yellow,5,9:50 AM,10:50 AM,11:08 AM,Actual,1:18,15:36/mi,12:00/mi,1:00
...
```

**Field definitions per row:**
- Leg: leg number 1–24
- Runner: current runner name for that leg
- Loop: "Green", "Yellow", or "Red"
- Loop Miles: numeric value from config
- Start: start time (hh:mm AM/PM)
- Est Finish: estimated finish time (hh:mm AM/PM)
- Actual Finish: actual finish time if entered, empty string if not
- Status: "Actual", "Estimated (Baseline)", or "Estimated (Live)"
- Actual Duration: formatted as h:mm if available, empty if not
  (e.g. "0:50" for 50 minutes, "1:18" for 78 minutes)
- Actual Pace: formatted as MM:SS/mi if available, empty if not
  (e.g. "12:30/mi")
- Predicted Pace: formatted as MM:SS/mi (e.g. "12:00/mi")
- Est Duration: formatted as h:mm (e.g. "0:48")

### Section 3 — Config
```
[Config]
Team Name,Event Name,Race Start Time,Avg Pace,Green Miles,Yellow Miles,Red Miles,Night Adjustment,Night Adjustment Type,Night Hours Start,Night Hours End
Party Pace,Ragnar Trail Atlanta 2025,9:00 AM,12:00/mi,4,5,6,5%,percent,8:00 PM,7:00 AM
```

---

## Time and Duration Formatting Helpers

Build these as reusable formatting functions:

```javascript
function formatDuration(minutes) {
  if (minutes === null || minutes === undefined) return '';
  const h = Math.floor(minutes / 60);
  const m = Math.round(minutes % 60);
  return `${h}:${String(m).padStart(2, '0')}`;
}

function formatPace(minutesPerMile) {
  if (minutesPerMile === null || minutesPerMile === undefined) return '';
  const mins = Math.floor(minutesPerMile);
  const secs = Math.round((minutesPerMile - mins) * 60);
  return `${mins}:${String(secs).padStart(2, '0')}/mi`;
}
```

---

## CSV String Escaping

Any field value that contains a comma must be wrapped in double quotes.
Any field value that contains double quotes must have them escaped by
doubling (`"` → `""`).

```javascript
function escapeCSV(value) {
  if (value === null || value === undefined) return '';
  const str = String(value);
  if (str.includes(',') || str.includes('"') || str.includes('\n')) {
    return `"${str.replace(/"/g, '""')}"`;
  }
  return str;
}
```

Apply `escapeCSV()` to every field value before writing to the CSV string.

---

## Complete generateCSV() Structure

```javascript
function generateCSV() {
  const config = getConfig(); // read from localStorage
  const computedLegs = calculateLegData(); // use existing function

  const lines = [];

  // Section 1 — Race Summary
  lines.push('[Race Summary]');
  lines.push('Team Name,Event,Race Start,Finish Time,Total Duration,Status');
  // ... build summary row

  lines.push(''); // blank line between sections

  // Section 2 — Race Data
  lines.push('[Race Data]');
  lines.push('Leg,Runner,Loop,Loop Miles,Start,Est Finish,Actual Finish,Status,Actual Duration,Actual Pace,Predicted Pace,Est Duration');
  computedLegs.forEach(leg => {
    // ... build one row per leg
  });

  lines.push(''); // blank line between sections

  // Section 3 — Config
  lines.push('[Config]');
  lines.push('Team Name,Event Name,Race Start Time,Avg Pace,Green Miles,Yellow Miles,Red Miles,Night Adjustment,Night Adjustment Type,Night Hours Start,Night Hours End');
  // ... build config row

  return lines.join('\n');
}
```

---

## Verification Steps

1. **Basic export:**
   With no actual finish times entered, tap Export. A CSV file downloads.
   Open it and verify:
   - Race Summary shows Status = "In Progress"
   - All 24 Race Data rows present
   - Actual Finish column is empty for all rows
   - Config section shows correct current config values

2. **Mid-race export:**
   Enter actual finish times for Legs 1–12. Tap Export. Verify:
   - Legs 1–12 show actual finish times, durations, and paces
   - Legs 13–24 show empty Actual Finish, estimated values only
   - Status column correctly shows Actual / Estimated (Baseline) /
     Estimated (Live) per leg

3. **Race Complete export:**
   Enter all 24 actual finish times. Tap Export. Verify:
   - Race Summary shows Status = "Complete"
   - Total Duration shows correct elapsed time
   - All 24 rows show actual values
   - Finish Time shows Leg 24 actual finish time

4. **Filename format:**
   Verify downloaded filename matches `ragnar_tracker_YYYYMMDD_HHMM.csv`
   with correct current date and time.

5. **CSV opens correctly in Excel/Google Sheets:**
   Open the exported file in a spreadsheet app. Verify:
   - Three sections are clearly separated
   - All columns align correctly
   - No stray commas or broken rows
   - Duration and pace values are readable

6. **Special characters:**
   Change Team Name to something with a comma (e.g. "Run, Forrest").
   Export. Open CSV. Verify the team name is correctly quoted and
   does not break the CSV structure.

7. **Confirmation message:**
   After tapping Export, verify "Exported successfully" briefly
   appears in the footer and fades after 2 seconds.

---

## Key Constraints

- `generateCSV()` is a shared function — build it cleanly so Step 16
  (File System Access API) can call it directly without duplication
- All field values must go through `escapeCSV()` before output
- Duration format: h:mm (not hh:mm — hours are not zero-padded)
- Pace format: MM:SS/mi
- Blank line between each CSV section for readability
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Export button triggers CSV download
- `generateCSV()` function producing correct three-section output
- Correct filename with timestamp
- Brief export confirmation in footer
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
