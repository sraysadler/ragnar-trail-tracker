# Build Task: Step 16 — File System Access API Automatic Backup

## Objective
Implement automatic background CSV backup using the File System Access API.
After every Actual Finish entry, the app silently writes an updated CSV to
a user-selected folder on the device. A one-time folder permission is
requested on first launch via a user-initiated prompt.

This feature works in Chrome on Android and Chrome on desktop (Mac/Windows).
It does not work on iOS regardless of browser.

---

## Important — generateCSV() Already Exists

The `generateCSV()` function was built in Step 15 (Manual CSV Export).
Do NOT build a new one. The automatic backup should call the existing
`generateCSV()` function directly. This is the shared function that
both manual export and automatic backup use.

---

## First Launch — Folder Permission Request

On app load, check if a folder handle has been stored in IndexedDB.
See Storage Note below.

If no folder handle exists:
- Show a non-blocking banner at the top or bottom of the screen:
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
async function saveFolderHandle(handle) {
  const db = await openBackupDB();
  const tx = db.transaction('handles', 'readwrite');
  tx.objectStore('handles').put(handle, 'backupFolder');
  await tx.done;
}

async function getFolderHandle() {
  const db = await openBackupDB();
  const tx = db.transaction('handles', 'readonly');
  return tx.objectStore('handles').get('backupFolder');
}

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

Each backup write creates a new file — never overwrites existing backups.
Filename format:

```
ragnar_tracker_YYYYMMDD_HHMM.csv
```

Example: `ragnar_tracker_20260415_0622.csv`

This format sorts chronologically alphabetically, making it easy to
identify the most recent backup.

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
call `writeBackup()`. This is in addition to — not instead of — the
existing localStorage save:

```javascript
async function writeBackup() {
  const handle = await getFolderHandle();
  if (!handle) return; // no folder set up, skip silently

  const hasPermission = await verifyFolderPermission(handle);
  if (!hasPermission) return; // permission lost, skip silently

  try {
    const filename = buildBackupFilename();
    const csv = generateCSV(); // call existing shared function from Step 15
    const fileHandle = await handle.getFileHandle(filename, { create: true });
    const writable = await fileHandle.createWritable();
    await writable.write(csv);
    await writable.close();

    lastBackupTime = new Date();
    updateBackupIndicators(true);
  } catch (e) {
    console.error('Backup write failed:', e);
    updateBackupIndicators(false);
  }
}
```

---

## Visual Indicators

### Footer — green dot next to Last Saved
The green dot placeholder is already in the footer. Wire it up:

- After successful backup write: dot is green (`#4caf50`)
- If backup has never run or last write failed: dot absent or red (`#e53935`)
- Dot stays visible — does not fade

```
● Last saved: 6:22 AM
```

### Settings panel — Backup Status section
Add a section at the bottom of the Settings panel body, below all
config fields and above the Reset/Restore/Save buttons:

```
BACKUP STATUS
Last backed up: 04/15/2026, 6:22 AM
Backup folder: RaceTracker/
```

- If no backup has been written yet: show "Last backed up: Never"
- If no folder is configured: show "Automatic backup not configured."
  with a tappable "Tap to set up" link that triggers the folder picker
- Store `lastBackupTime` in localStorage so it persists across sessions

---

## Restore from Backup

If on app load localStorage is empty (no race data) but a folder handle
exists and permission is granted:

1. Read the folder directory listing
2. Find all files matching `ragnar_tracker_*.csv`
3. Sort by filename descending to find the most recent
4. Show prompt:
   "No saved data found. Restore from last backup?
   Last backup: [filename timestamp]"
   - "Restore" button: parse the CSV and restore race data
   - "Start Fresh" button: dismiss and start with empty race

CSV parsing for restore:
- Read the [Race Data] section — restore `actualFinish` values per leg
- Read the [Config] section — restore all config values
- Re-run `calculateLegData()` and `renderTable()` and `renderStatusBar()`

---

## Settings Panel — Manual Backup Setup

In the Backup Status section of Settings, if no folder is configured,
show a tappable "Set up automatic backup" option that triggers the
folder picker. This allows setup after dismissing the first-launch prompt.

---

## Verification Steps

Test in Chrome on Mac first, then verify on Android tablet.

1. **First launch — no folder configured:**
   Clear IndexedDB if needed. Open app.
   Banner appears prompting backup setup.
   Tap "Set Up". Folder picker opens.
   Select or create RaceTracker/ folder in Downloads.
   Confirmation message appears. Banner dismisses.

2. **Automatic backup on entry:**
   Enter an actual finish time.
   Navigate to RaceTracker/ folder on the device.
   Verify a new CSV file exists with correct timestamp filename.
   Open it — verify content matches current race state including
   the Race Data, Race Summary, and Config sections.

3. **Green dot appears:**
   After entering a time with backup configured, verify green dot
   appears next to "Last saved" in footer.

4. **Settings backup status:**
   Open Settings. Verify "Last backed up: [date], [time]" shows
   correct timestamp. Verify folder name shown.

5. **New file per backup:**
   Enter another actual finish time one minute later.
   Verify a second CSV file appears in the folder —
   new file, first file unchanged.

6. **Restore flow:**
   Note current race state.
   Clear localStorage manually (DevTools → Application →
   localStorage → clear all).
   Refresh the app.
   Restore prompt appears with most recent backup filename.
   Tap Restore. All actual finish times restore correctly.
   Race table re-renders and estimates recalculate.

7. **"Not Now" then set up via Settings:**
   Clear IndexedDB. Refresh app.
   Tap "Not Now" on backup prompt.
   Open Settings. "Automatic backup not configured" shown.
   Tap "Set up" link. Folder picker opens.
   Select folder. Subsequent entries now backup automatically.

8. **Permission persistence (Mac first, then Android):**
   Set up backup folder. Close browser completely.
   Reopen app. Enter a finish time.
   Verify backup writes successfully without re-prompting for permission.

---

## Key Constraints

- `generateCSV()` already exists from Step 15 — call it, do not rebuild it
- Folder handle stored in IndexedDB, not localStorage
- Each backup is a new file — never overwrite existing backups
- Backup failures are silent to the user (no error alerts)
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- First launch backup setup prompt
- Automatic CSV write after every Actual Finish entry
- New timestamped file per backup write
- Green dot indicator in footer after successful backup
- Backup status section in Settings panel with last backup time
- Restore from backup prompt when localStorage is empty
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
