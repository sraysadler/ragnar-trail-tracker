# Ragnar Trail Tracker — Code Review Actions

A prioritized list of improvements identified during a three-reviewer code
review conducted April 18, 2026. Reviewers: Gemini (GEM), Claude Code (CC),
Claude (CG). Each item includes full implementation detail for handoff to
Claude Code.

This document is a planning artifact — nothing here changes the working app
until explicitly actioned. All items marked Do Not Do are recorded for
reference and should not be implemented without explicit discussion.

---

## Category 1 — Actual Bugs (Fix Before Race Day)

---

### CR-01 — Runner name injection via inline JS string (HIGH) ✓ FIXED

**Flagged by:** CC, CG (both)

**What the bug is:**
Runner names are inserted directly into inline `onclick` handlers inside
template literals:

```js
onclick="selectRunner('${r}')"
```

A runner named `D'Angelo` produces `selectRunner('D'Angelo')` — broken JS.
The dropdown stops working entirely for that runner. A name with `"` breaks
the `data-runner` attribute. A `<` in a name (possible via paste) renders
as HTML. The same issue appears in `renderRunnerStats()` and in table row
HTML wherever `${leg.runner}` is used.

**Fix:**
Two-part fix.

Part 1 — Add an `escapeHtml()` helper and use it on every templated name:

```js
function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, c => (
    {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}
  )[c]);
}
```

Part 2 — Stop passing names through inline handlers entirely. Use
`data-*` attributes and `addEventListener` instead:

```js
// In the template:
<div class="modal-runner-option" data-runner-index="${i}">
  <span>${escapeHtml(r)}</span>
</div>

// In the listener:
document.querySelector('.runner-dropdown').addEventListener('click', (e) => {
  const option = e.target.closest('[data-runner-index]');
  if (!option) return;
  const index = Number(option.dataset.runnerIndex);
  selectRunner(config['runner' + (index + 1)]);
});
```

Apply `escapeHtml()` everywhere names are inserted into `innerHTML`,
including runner stats rows and the table body.

**Risk if not fixed:** Any runner with an apostrophe in their name
(common — O'Brien, D'Angelo, etc.) will silently break the runner
substitution dropdown on race day.

**Fixed April 18, 2026:** `escapeHtml()` added. All inline `onclick`
handlers replaced with `data-runner-index` attributes and a single
delegated listener. `escapeHtml()` applied to race table rows, modal
selected-name display, dropdown option labels, Runner Stats section
headers, and keypad header. Verified working with apostrophe names.

---

### CR-02 — CSV restore doesn't handle quoted fields (HIGH — data loss) ✓ FIXED

**Flagged by:** CC

**What the bug is:**
`restoreFromCSV()` splits rows with `row.split(',')`. The `generateCSV()`
export correctly wraps values containing commas in quotes via `escapeCSV()`.
But restore splits naively, so a team name like `Party Pace, Atlanta`
exports correctly and restores corrupted — every column to the right of the
comma is shifted.

**Fix — Option A (recommended):** Add a minimal RFC 4180 CSV row parser:

```js
function parseCSVRow(row) {
  const result = [];
  let current = '';
  let inQuotes = false;
  for (let i = 0; i < row.length; i++) {
    const c = row[i];
    if (c === '"') {
      if (inQuotes && row[i + 1] === '"') { current += '"'; i++; }
      else { inQuotes = !inQuotes; }
    } else if (c === ',' && !inQuotes) {
      result.push(current); current = '';
    } else {
      current += c;
    }
  }
  result.push(current);
  return result;
}
```

Replace all `row.split(',')` calls in `restoreFromCSV()` with
`parseCSVRow(row)`.

**Fix — Option B (simpler but limiting):** Sanitize commas out of team
and event name fields at save time — strip or replace with a safe
character before writing to localStorage and CSV. Less robust but
eliminates the root cause for the most likely affected fields.

**Risk if not fixed:** A restore operation on a backup containing a
team or event name with a comma will silently corrupt the restored data.

**Fixed April 18, 2026:** Option A implemented. `parseCSVRow()` RFC 4180
parser added. All four `row.split(',')` calls in `restoreFromCSV()`
replaced with `parseCSVRow()`. `generateCSV()` and `escapeCSV()`
untouched.

---

### CR-03 — Backup files accumulate without limit (MEDIUM — operational) ✓ FIXED

**Flagged by:** CC

**What the bug is:**
`buildBackupFilename()` is minute-granular. Every write within the same
minute overwrites, but every new minute creates a new file. A 30-hour
Ragnar race with frequent updates produces roughly 1,800 CSV files in the
backup folder. There is no cleanup. After two race weekends, the folder
scan in `restoreFromCSV()` becomes slow and overwhelming.

**Fix — Recommended approach:**
Use a single canonical filename for the live/automatic backup:

```
ragnar_tracker_current.csv
```

Overwrite it on every automatic write. Reserve timestamped filenames
(`ragnar_tracker_YYYYMMDD_HHMM.csv`) for manual Export CSV only.

This gives the user one clean, always-current backup file automatically,
and dated snapshots only when they explicitly want them.

**Alternative approach (if history is desired):**
Keep a rolling cap of N files (e.g. 50) inside `writeBackup()`. After
each write, list all backup files in the folder, sort by name, and delete
any beyond the 50 most recent.

**Tradeoff note:**
The current behavior (every write is its own file) provides a free
version history, but that was likely unintentional and creates more
problems than it solves for a kiosk app.

**Fixed April 18, 2026:** Recommended approach implemented. `writeBackup()`
now always writes to `ragnar_tracker_current.csv` (single overwritten
file). Manual export path (`exportCSV()` → `downloadCSV()` →
`buildExportFilename()`) still produces unique timestamped filenames —
unchanged.

---

## Category 2 — Architecture (High Value, Not Urgent)

---

### CR-04 — Centralize state — stop reading localStorage inside render and compute functions (HIGH VALUE)

**Flagged by:** CC, CG (both — highest priority item for both)

**What the problem is:**
`calculateLegData()`, `openRowModal()`, `keypadRender()`, and other
functions each call `loadConfig()` and `loadRaceLegs()` internally, which
parse localStorage every time. The source of truth is scattered across
DOM, localStorage, and transient JS variables. This makes bugs harder to
isolate and the data flow harder to reason about.

**Fix:**
Create a single in-memory `state` object. Load once at startup. Mutate
through small update functions. Persist after changes.

```js
const state = {
  config: loadConfigFromStorage(),
  raceLegs: loadRaceLegsFromStorage(),
  computedLegs: [],
  dirty: false
};

function recompute() {
  state.computedLegs = calculateLegData(state.config, state.raceLegs);
}

function commit() {
  persistState(state);
  recompute();
  render(state);
}
```

Every user action — time entry, runner substitution, config save — calls
`commit()` (or a wrapper). No function reads from localStorage during
normal app flow; they read from `state`.

**Relationship to CR-05 and CR-06:** These three items go together.
Implement CR-04 first, then CR-05 and CR-06 follow naturally.

---

### CR-05 — Make `calculateLegData()` a pure function (HIGH VALUE)

**Flagged by:** CG

**What the problem is:**
`calculateLegData()` is close to a pure domain function but calls
`loadConfig()` and `loadRaceLegs()` internally. It also mixes numeric
outputs with UI-ready display strings.

**Fix:**
- Change signature to `calculateLegData(config, raceLegs)` — accept
  arguments only, no internal storage reads
- Move display string formatting out — return normalized numbers only
- Format strings at the render layer

```js
// Before:
function calculateLegData() {
  const cfg = loadConfig();
  const legs = loadRaceLegs();
  // ...
}

// After:
function calculateLegData(config, raceLegs) {
  // Pure logic — no localStorage reads
  // Returns computed values as numbers
}
```

**Why this matters:**
Easier to test, easier to debug, easier to build future features
(undo, reset leg, import/export). Pairs with CR-04.

---

### CR-06 — Route all saves through one unified pipeline (HIGH VALUE)

**Flagged by:** CG

**What the problem is:**
Some update paths call `writeBackup()` and some don't. Keypad confirm
paths write backup; `saveRowModal()` runner changes may not; `saveConfig()`
may not. There is no guarantee that every meaningful state change reaches
the backup layer.

**Fix:**
Create one `updateState()` function that every write goes through:

```js
function updateState(mutatorFn) {
  mutatorFn(state);
  recompute();
  render(state);
  saveToLocalStorage(state);
  queueBackup(state);
}
```

Every user action that changes data calls `updateState()`. No path
bypasses backup or persistence. Pairs with CR-04.

---

### CR-07 — Store config as native types; format at the DOM layer only (HIGH VALUE)

**Flagged by:** CC, CG (both)

**What the problem is:**
Config values round-trip as display strings: miles as `"4.0 mi"`, pace as
`"12:00 /mi"`, night adjustment as `"5%"`, times as `"8:00 AM"`. This
forces constant parsing through `parseMiles()`, `parsePaceMinutes()`, and
`timeToMinutes()` everywhere the values are used. If a display format ever
changes (e.g. `"4.0 mi"` to `"4.0 miles"`), config storage silently breaks.

**Fix:**
Store config values as their native types — numbers as numbers, times as
strings in a canonical format. Only convert to display strings at the
render layer.

```js
// Store:
cfg.greenMiles = 4.2;           // number
cfg.avgPace = 12.0;             // number (minutes per mile)
cfg.nightAdjustment = 5;        // number (percent)
cfg.raceStartTime = "9:00 AM";  // string — keep as-is (canonical format)

// Render:
element.textContent = cfg.greenMiles.toFixed(1) + ' mi';
```

Remove `parseMiles()` call sites that are only needed because storage
format includes the unit string. Keep parsing only at the config panel
keypad input boundary.

---

## Category 3 — Polish (Do If Time Allows)

---

### CR-08 — Replace `alert()` / `confirm()` with custom modals

**Flagged by:** CC

On Android Chrome in Screen Pinning mode, native `alert()` and `confirm()`
dialogs are jarring and can interrupt the kiosk feel. The app already has
a modal pattern. Extend it with a lightweight custom `confirm()` that
matches the app's dark UI:

```js
function appConfirm(message, onConfirm, onCancel) {
  // Render a floating card with message, Confirm (green), Cancel (gray)
  // Calls onConfirm() or onCancel() based on tap
}
```

Replace all `confirm()` calls (Reset, Restore, runner substitution
cancel-with-changes) with `appConfirm()`.

---

### CR-09 — AM/PM entry sanity check on time entry

**Flagged by:** CC

If a runner starts at 7:40 PM and someone accidentally confirms 8:10 AM,
`extendTime()` silently produces a 12.5-hour leg. The app has no warning.
Pace will look wildly wrong but nothing surfaces the error immediately.

Add a duration sanity check after `actualFinish` is confirmed:

```js
const MAX_REASONABLE_DURATION = 180; // 3 hours in minutes
if (actualDuration > MAX_REASONABLE_DURATION) {
  appConfirm(
    `Duration is ${formatDuration(actualDuration)}. Is this correct?`,
    () => saveActualFinish(legIndex, confirmedTime),
    () => reopenKeypad(legIndex)
  );
}
```

Adjust the threshold as appropriate for the race (Red loop at 6 mi at
slow pace may approach 2+ hours).

---

### CR-10 — Backup write concurrency guard

**Flagged by:** CC

Two rapid confirms can call `writeBackup()` in parallel. The
`createWritable()` → `write()` → `close()` sequence is not atomic.
Unlikely to corrupt in practice but cheap to prevent:

```js
let _backupQueue = Promise.resolve();

function writeBackup() {
  _backupQueue = _backupQueue
    .then(() => _writeBackupInner())
    .catch(e => console.error('Backup error:', e));
  return _backupQueue;
}
```

---

### CR-11 — CSS variables for color tokens

**Flagged by:** GEM, CG (both)

Hex codes are hardcoded throughout 500+ lines of CSS. Declare a `:root`
block at the top of the style section:

```css
:root {
  --bg-main: #0d0d0d;
  --bg-card: #1a1a1a;
  --bg-surface: #333;
  --text-primary: #ffffff;
  --text-muted: #999999;
  --accent-blue: #4a9eff;
  --accent-green: #4caf50;
  --row-green: #003300;
  --row-yellow: #444400;
  --row-red: #330000;
  --border-input: #4a9eff;
  --bg-input: #0a1a2a;
}
```

Replace every hardcoded hex in the stylesheet with its variable. Makes
future visual changes (e.g. daylight high-contrast mode) a one-line edit.

---

### CR-12 — Strip `console.log` debug output

**Flagged by:** CC

Debug `console.log` calls are present in `restoreConfigFromBackup()` and
possibly elsewhere. Gate behind a flag or remove:

```js
const DEBUG = false;
function log(...args) { if (DEBUG) console.log(...args); }
```

Replace `console.log` with `log()` throughout.

---

### CR-13 — Minute timer boundary alignment

**Flagged by:** CC

`setInterval(updateCurrentTime, 60000)` drifts from the actual minute
boundary over time — it starts from whenever the page loaded, not from
the next `:00` second. In a glanceable kiosk clock this is noticeable.
Align to the next minute on startup:

```js
const msUntilNextMinute = 60000 - (Date.now() % 60000);
setTimeout(() => {
  updateCurrentTime();
  setInterval(updateCurrentTime, 60000);
}, msUntilNextMinute);
```

---

### CR-14 — Long-press discoverability for loop miles override

**Flagged by:** CC

The 3-second press-and-hold on Loop Miles to trigger the override keypad
has visual feedback only at 1.5 seconds, which means users trying for the
first time often give up before the keypad opens. Add a small hint label
on completed legs:

- Show a subtle "hold to override" hint beneath the Loop Miles field when
  `leg.actualFinish` is set
- Or add a tooltip-style label that appears on short tap (not 3s)
  directing the user to press and hold

---

## Do Not Do

These items were raised by reviewers but are either contrary to documented
architecture decisions or inappropriate for this project at this stage.

| Item | Reason |
|------|--------|
| Generate `LEG_DEFINITIONS` dynamically | Directly contradicts documented architecture decision: "LEG_DEFINITIONS is a hardcoded static constant — never derived." Do not change without explicit discussion. |
| Add a framework (React, Vue, etc.) | All three reviewers explicitly said not to. Single-file, no-dependency architecture is correct for this use case. |
| Add a build step | Same as above. Do not add build tooling. |
| Flat `runner1`–`runner8` → array refactor | Valid long-term improvement but touches initRaceLegs, renderRunnerStats, populateConfigPanel, saveConfig, resetConfigPanel, and boot. Too large a refactor for current stage. |
| Full event delegation table replacement | Useful in principle but entangled with the larger state/render refactor (CR-04). Don't do piecemeal; handle as part of CR-04 if at all. |
| Test harness for forecasting logic | Good idea for a multi-file project. Inappropriate for a single-file kiosk app at this stage. |
| Accessibility / ARIA improvements | Touch-first kiosk app, single team of known users. Not a priority. |

---

## Prioritized Action List

| # | Item | Category | Priority | Status |
|---|------|----------|----------|--------|
| CR-01 | Runner name HTML/JS injection fix | Bug | Fix before race day | ✓ Fixed April 18 |
| CR-02 | CSV restore quoted-field parser | Bug | Fix before race day | ✓ Fixed April 18 |
| CR-03 | Backup file accumulation | Bug/Ops | Fix before race day | ✓ Fixed April 18 |
| CR-04 | Centralize state object | Architecture | High value | Open |
| CR-05 | Pure `calculateLegData()` | Architecture | High value | Open |
| CR-06 | Unified save pipeline | Architecture | High value | Open |
| CR-07 | Config as native types | Architecture | High value | Open |
| CR-08 | Custom confirm modals | Polish | If time allows | Open |
| CR-09 | AM/PM sanity check | Polish | If time allows | Open |
| CR-10 | Backup write queue | Polish | If time allows | Open |
| CR-11 | CSS variables | Polish | If time allows | Open |
| CR-12 | Strip console.log | Polish | If time allows | Open |
| CR-13 | Timer boundary alignment | Polish | If time allows | Open |
| CR-14 | Long-press discoverability | Polish | If time allows | Open |

---

*Code Review Actions v2.0 — Updated April 18, 2026*
*CR-01, CR-02, CR-03 marked complete.*
*Ragnar Trail Tracker — github.com/sraysadler/ragnar-trail-tracker*
*Source reviews: GEM (Gemini), CC (Claude Code), CG (Claude)*
*Companion to: Race Tracker Build Log v5, Race Tracker Decisions Log v4*
