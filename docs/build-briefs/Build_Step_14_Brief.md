# Build Task: Step 14 — File System Access API Automatic Backup

## Objective
Implement automatic background CSV backup using the File System Access API.
After every Actual Finish entry, the app silently writes an updated CSV to
a user-selected folder on the device. A one-time folder permission is
requested on first launch via a user-initiated prompt.

This feature works in Chrome on Android and Chrome on desktop (Mac/Windows).
It does not work on iOS regardless of browser.

---

## First Launch — Folder Permission Request

On app load, check if a folder handle has been stored in localStorage
under the key `backupFolderHandle`. Note: folder handles must be stored
using IndexedDB, not localStorage — see Storage note below.

If no folder handle exists:
- Show a non-blocking banner or prompt at the top or bottom of the screen:
  "Set up automatic backup? Tap to choose a backup folder."
- Include a "Set Up" button and a "Not Now" button
- Tapping "Set Up" triggers the folder picker via File System Access API
- User selects or creates a folder (suggest `RaceTracker/` in Downloads)
- On successful selection, store the folder handle in IndexedDB
- Show brief confirmation: "Backup folder set. Backups will save automatically."
- Dismiss the banner

If user taps "Not Now":
- Dismiss the banner
- Do not request again automatically during this session
- User can set up backup via Settings panel later

---

## Storage Note — IndexedDB for Folder Handle

The File System Access API folder handle cannot be stored in localStorage
(it is not serializable). Use IndexedDB to persist the folder handle
across sessions:

```javascript
// Store folder handle
async function saveFolderHandle(handle) {
  const db = await openBackupDB();
  const tx = db.transaction('handles', 'readwrite');
  tx.objectStore('handles').put(handle, 'backupFolder');
  await tx.done;
}

// Retrieve folder handle
async function getFolderHandle() {
  const db = await openBackupDB();
  const tx = db.transaction('handles', 'readonly');
  return tx.objectStore('handles').get('backupFolder');
}

// Open/create IndexedDB
function openBackupDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('RaceTrackerBackup', 1);
    request.onupgradeneeded = e => {
      e.target.result.createObjectStore('handles');
    };
    request.onsuccess = e => resolve(e.target.result);
    request.onerror = reject;
  });
}
```

On app load, retrieve the folder handle and verify permission is still
granted:

```javascript
async function verifyFolderPermission(handle) {
  if (!handle) return false;
  const permission = await handle.queryPermission({ mode: 'readwrite' });
  if (permission === 'granted') return true;
  const request = await handle.requestPermission({ mode: 'readwrite' });
  return request === 'granted';
}
```

---

## Backup File Naming

Each backup write creates a new file. Filename format:

```
ragnar_tracker_YYYYMMDD_HHMM.csv
```

Example: `ragnar_tracker_20260415_0622.csv`

This format sorts chronologically alphabetically, allowing the app to
always identify the most recent backup by sorting filenames.

Build the filename from current date/time at moment of write:

```javascript
function buildBackupFilename() {
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

## Automatic Backup Trigger

After every Actual Finish entry (same point where localStorage is written),
call `writeBackup()`:

```javascript
async function writeBackup() {
  const handle = await getFolderHandle();
  if (!handle) return; // no folder set up, skip silently

  const hasPermission = await verifyFolderPermission(handle);
  if (!hasPermission) return; // permission lost, skip silently

  try {
    const filename = buildBackupFilename();
    const csv = generateCSV(); // see CSV Format section below
    const fileHandle = await handle.getFileHandle(filename, { create: true });
    const writable = await fileHandle.createWritable();
    await writable.write(csv);
    await writable.close();

    // Update backup timestamp
    const now = new Date();
    lastBackupTime = now;
    updateBackupIndicators();
  } catch (e) {
    // Fail silently — do not interrupt the user
    console.error('Backup write failed:', e);
    updateBackupIndicators(false); // signal failure
  }
}
```

---

## Visual Indicators

### Footer — green dot next to Last Saved
Add a small colored dot (`●`) immediately before or after the
"Last saved: [time]" text in the footer:

- After successful backup write: dot is green (`#4caf50`)
- If backup has never run or last write failed: dot is absent or red (`#e53935`)
- Dot stays visible until next action (does not fade)

```
● Last saved: 6:22 AM
```

### Settings panel — Last Backed Up
Add a line at the bottom of the Settings panel body, below all
config fields and above the Reset/Restore/Save buttons:

```
BACKUP STATUS
Last backed up: 04/15/2026, 6:22 AM
Backup folder: RaceTracker/
```

If no backup has been written yet: "Last backed up: Never"
If no folder is set up: "Automatic backup not configured. Tap to set up."
The "Tap to set up" text should be tappable and trigger the folder picker.

---

## CSV Format

The CSV written to the backup folder follows the same format as the
manual export (to be built in Step 15). Build a shared `generateCSV()`
function that both the automatic backup and the manual export button
will use.

```
[Race Summary]
Team Name,Event,Race Start,Projected/Actual Finish,Total Duration,Status

[Race Data]
Leg,Runner,Loop,Loop Miles,Start,Est Finish,Actual Finish,Status,
Actual Duration,Actual Pace,Predicted Pace,Est Duration

[Config]
Team Name,Event Name,Race Start Time,Avg Pace,Green Miles,Yellow Miles,
Red Miles,Night Adjustment,Night Adjustment Type,Night Hours Start,
Night Hours End
```

Race Summary section:
- Status: "In Progress" or "Complete"
- Projected/Actual Finish: Leg 24 finish time
- Total Duration: time from race start to Leg 24 finish (blank if not complete)

Race Data section:
- One row per leg, all 24 legs
- Null/empty fields shown as empty string

Config section:
- One row of current config values

---

## Restore from Backup

If on app load localStorage is empty (no race data) but a folder handle
exists and permission is granted:

1. Read the folder directory
2. Find all files matching `ragnar_tracker_*.csv`
3. Sort by filename (descending) to find the most recent
4. Show prompt: "No saved data found. Restore from last backup?
   Last backup: [filename timestamp]"
   - "Restore" button: parse the CSV and restore race data
   - "Start Fresh" button: dismiss and start with empty race

CSV parsing for restore:
- Read the [Race Data] section
- Restore `actualFinish` values for each leg
- Re-run `calculateLegData()` and `renderTable()`
- Restore config values from [Config] section

---

## Settings Panel — Manual Backup Setup

In the Settings panel backup status section, if no folder is configured,
show a tappable "Set up automatic backup" option that triggers the folder
picker. This allows the user to set up backup after dismissing the
first-launch prompt.

---

## Verification Steps

Test in Chrome on Mac first, then verify on Android tablet.

1. **First launch — no folder configured:**
   Open app fresh (clear IndexedDB if needed).
   Banner appears prompting backup setup.
   Tap "Set Up". Folder picker opens.
   Select Downloads or create RaceTracker/ folder.
   Confirmation message appears briefly.
   Banner dismisses.

2. **Automatic backup on entry:**
   Enter an actual finish time.
   Navigate to the RaceTracker/ folder on the device.
   Verify a new CSV file was created with correct timestamp filename.
   Open the file and verify content matches current race state.

3. **Green dot appears:**
   After entering a time, verify green dot appears next to
   "Last saved" in footer.

4. **Settings backup status:**
   Open Settings. Verify "Last backed up: [date], [time]" shows
   correct timestamp. Verify folder name shown.

5. **Second entry:**
   Enter another actual finish time.
   Verify a second CSV file appears in the folder (new file,
   not overwriting the first).

6. **Restore flow:**
   Note current race state.
   Clear localStorage manually (DevTools → Application → localStorage → clear).
   Refresh the app.
   Restore prompt appears with filename of most recent backup.
   Tap Restore. Race data restores correctly.
   Verify all actual finish times are back and estimates recalculate.

7. **"Not Now" on first launch:**
   Clear IndexedDB. Refresh app.
   Tap "Not Now" on backup prompt.
   Banner dismisses. No folder selected.
   Enter a finish time. No backup written (no folder).
   Open Settings. "Automatic backup not configured" shown.
   Tap "Set up" link in Settings. Folder picker opens.
   Select folder. Subsequent entries now backup automatically.

8. **Permission lost (Android specific):**
   Set up backup folder.
   Close Chrome completely.
   Reopen app.
   Verify permission is re-granted automatically (Chrome on Android
   typically remembers this).
   Enter a finish time. Backup writes successfully.

---

## Key Constraints

- File System Access API only — no server, no cloud
- Folder handle stored in IndexedDB, not localStorage
- Each backup is a new file — never overwrite existing backups
- Backup failures are silent to the user (no error alerts)
- generateCSV() should be a shared function used by both
  automatic backup and manual export (Step 15)
- Single HTML file — no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- First launch backup setup prompt
- Automatic CSV write after every Actual Finish entry
- New timestamped file per backup write
- Green dot indicator in footer after successful backup
- Backup status section in Settings panel
- Restore from backup prompt when localStorage is empty
- generateCSV() function ready for Step 15 manual export

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
