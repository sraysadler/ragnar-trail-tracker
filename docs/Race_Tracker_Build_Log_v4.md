# Ragnar Trail Tracker — Build Log

A running record of decisions, changes, and deviations from the original
app plan made during the build. Use this to keep Claude Code sessions
aligned when starting fresh, and to track what has been built and what
remains.

---

## Build Session: April 15–17, 2026
### Steps 1–25 completed

---

## Step 1 — HTML/CSS Layout ✓

**Completed as specified.**

**Adjustments made:**
- Yellow row color changed from `#333300` to `#444400` — original value
  was too dark and indistinguishable from green rows at a glance
- Active leg outline brightened from `1px solid #555` to `2px solid #fff`
  (white) for visibility against all three loop row colors

---

## Step 2 — Config Panel ✓

**Completed as specified with adjustments:**

- Config panel implemented as a **floating overlay** on top of the main view
- Team name and Event name fields made approximately twice as wide as other
  tappable fields — user requested this explicitly
- All tappable fields use blue-bordered dark style (`#0a1a2a` background,
  `1px solid #4a9eff` border) consistently
- Text overflow in Team name and Event name fields is clipped
- Night Hours implemented as two separate tappable time fields with a
  dash separator, right-aligned — shrink proportionally on narrow screens
- Reset/Restore/Save buttons inside the overlay footer

---

## Step 3 — Race Table Generation ✓

**Completed as specified.**

- `LEG_DEFINITIONS` hardcoded as a static constant
- Runner assignment: `(leg - 1) % 8` formula
- `renderTable()` called on page load and after every config Save
- Yellow row color in `LEG_DEFINITIONS`: `#444400`

---

## Step 4 — Status Bar ✓

**Completed after Step 5 due to dependency on calculation chain.**

**Implemented:**
- Projected Team Finish: `finishTime` of Leg 24
- Current Loop: `[legNumber] – [runnerName]` of first leg without actual finish
- Next Runner: runner of leg after current
- Next Start: start time of leg after current
- Current time in header: actual device time, updates every 60 seconds
- Last Saved: permanently visible in footer with green dot after successful
  backup write

**Race Complete state:**
- Projected Team Finish bar label → "FINAL TEAM FINISH"
- Current Loop → "Complete"
- Next Runner and Next Start → "—"
- Reverts automatically if any actual finish is cleared

**Decision:** No flash message on restore. Permanent "Last saved" indicator
serves this purpose.

---

## Step 5 — Core Calculation Logic ✓

**Completed as specified.**

- Time arithmetic in minutes-since-midnight internally
- Midnight crossing handled correctly
- `calculateLegData()` processes legs strictly 1→24
- Full re-render on every actual finish entry
- `prompt()` used temporarily for time entry testing — removed in Step 7+8

---

## Step 6 — Forecasting Logic ✓

**Simplified from original spec. Subsequently updated to pace-based
night adjustment with normalization.**

**Final model:**
- 0 prior legs → config baseline pace → `Estimated (Baseline)`
- 1 prior leg → runner's normalized actual pace → `Estimated (Live)`
- 2+ prior legs → equal-weight average of normalized actual paces
  → `Estimated (Live)`

**Night pace adjustment (updated after initial build):**
- Applied as a percentage to **predicted pace** (not to estimated duration)
- Night leg actual paces stored as normalized day-equivalent:
  `normalizedPace = actualPace / (1 + nightAdjustment/100)`
- When predicting night leg: `predictedPace = normalizedAvg × (1 + nightAdjustment/100)`
- When predicting day leg: `predictedPace = normalizedAvg`
- Handles midnight crossing correctly
- Default: 5%

**Bug found and fixed during initial build:**
Initial implementation had a unit conversion error causing Leg 9
estimated duration to show ~165 minutes instead of ~75 minutes.
Fixed after adding console.log diagnostics.

---

## Steps 7+8 — Time Entry Keypad ✓

**Combined into single step.**

**Keypad layout (confirmed):**
```
[ 1 ] [ 2 ] [ 3 ]
[ 4 ] [ 5 ] [ 6 ]
[ 7 ] [ 8 ] [ 9 ]
[AM/PM] [ 0 ] [ ⌫ ]
```
All buttons equal size.

**Time display:**
- Shows as `h:mm` with AM/PM as a separate indicator
- This format accepted by user — do not change

**Clear behavior:**
- When all digits backspaced from existing entry: display shows `--:-- AM`,
  Confirm button changes to red "Clear"
- Tapping Clear removes the actual finish time
- Beginning to type new digits reverts Clear back to Confirm

**Digit entry:**
- Right-to-left calculator style
- Intermediate states (e.g. `--:65` while typing `6:54`) are correct —
  not a bug

**`prompt()` removed entirely.**

---

## Step 9 — Row Detail Modal ✓

**Completed with adjustments.**

**Modal:** Floating card centered over dimmed background. Race table
visible but dimmed behind it. Tapping dimmed backdrop closes modal.

**Final field order:**
1. Runner (custom dropdown)
2. Loop Miles (dotted border — press and hold to override on completed legs)
3. Start
4. Est Finish
5. Actual Finish (blue tappable box)
6. Status
7. Est Duration
8. Actual Duration
9. Predicted Pace
10. Actual Pace

**Runner dropdown:** Custom dark UI dropdown replacing native `<select>`.
Selected runner highlighted in blue with checkmark. Chevron flips ▼/▲.

**Save behavior:**
- Actual Finish: saves immediately when keypad confirms
- Runner change: saves only when modal Save changes is tapped

---

## Step 10 — Runner Substitution ✓

**Completed as part of Step 9.** Custom runner dropdown built directly
into the Row Detail Modal.

**Important:** All forecasting and Runner Stats use the current `runner`
field — not `defaultRunner`. Substituted legs count toward the substituting
runner's pace history and stats.

---

## Step 11 — Deleted Finish Time Behavior ✓

**Completed as part of Steps 7+8.** Clear mechanism in keypad handles
deletion. Cascade behavior built in Step 5.

---

## Step 12 — Race Complete State ✓

**Completed as part of Step 4.** Verified with all 24 actual finish
times from 2025 race data.

---

## Step 13 — localStorage Auto-Save and Restore ✓

**Completed as part of Step 5.** Silent restore, no flash message.

---

## Step 14 — Config Panel Keypad Integration ✓

**Completed.**

**Keypad variants:**
- Race Start Time, Night Hours From/To → Variant A (time keypad, AM/PM toggle)
- Green/Yellow/Red Miles → Variant B (numeric, implied decimal, blank bottom-left)
- Avg Pace → Variant C (pace MM:SS format, blank bottom-left)
- Night Adjustment → Variant B (numeric, implied decimal)

**Implied decimal:** Miles and Night Adjustment keypads use right-to-left
implied decimal (one place from right). Type `4`, `6` to get `4.6`.
No decimal button needed.

**Pending state:** Keypad values do not write to localStorage until
Settings Save is tapped. Cancel/Close discards pending changes.

**Additional fixes made during this step:**
- Team Name and Event Name fields made editable (text input)
- Header updates immediately when Team/Event name saved
- Night Hours fields shrink proportionally on narrow screens (no overflow)

---

## Step 15 — File System Access API Automatic Backup ✓

**Completed. Verified on Chrome/Mac. Android verification pending.**

- Folder handle stored in IndexedDB (not localStorage)
- Each backup creates new timestamped file: `ragnar_tracker_YYYYMMDD_HHMM.csv`
- Never overwrites existing backups
- First launch: non-blocking banner prompts folder setup
- Green dot in footer after successful backup write
- Backup Status section in Settings: folder name (tappable link), last
  backup timestamp
- Restore prompt when localStorage empty but backup folder exists
- `generateCSV()` built as shared function for both backup and manual export

**Backup folder as tappable link:**
Folder name in Backup Status section is a blue hyperlink. Tapping
re-opens the File System Access API folder picker to change folders.

---

## Step 16 — Manual CSV Export ✓

**Completed.** Export button moved to Settings panel (Backup Status
section) — no longer in the footer.

- Uses shared `generateCSV()` from Step 15
- CSV format: Race Summary → Race Data → Config
- Brief "Exported successfully" footer message after download
- Fixed: Avg Pace was showing "NaN:NaN/mi" in Config section — corrected

---

## Step 17 — Visual Design Refinement ✓

**Completed.**

- Footer: green dot moved immediately left of "Last saved" text
- Column headers: slightly larger font (10px) and taller row (36px)
- Active leg outline: changed to `2px solid #fff` (white) for visibility
  against all loop colors
- Night Hours fields: shrink proportionally on narrow screens

---

## Step 18 — Fresh Load Experience ✓

**Completed.**

- On first load with no localStorage data: Settings panel opens
  automatically with title "Race Setup"
- Once config is saved: subsequent opens show "Settings"
- Blank runner names and team/event name fields on fresh load
- Default numeric values still pre-filled (loop distances, pace, etc.)

---

## Step 19 — Reset and Restore Buttons ✓

**Completed.**

- Reset: confirmation dialog, reverts config fields to defaults in panel
  (does not save until Save is tapped)
- Restore: confirmation dialog with backup filename, closes Settings,
  shows "Restored from backup [filename]" in footer, re-renders table
- Both buttons were previously non-functional (no feedback) — fixed

---

## Step 20 — Night Pace Adjustment Logic Update ✓

**Updated from duration-based to pace-based with normalization.**

Original implementation applied night adjustment as a duration multiplier.
Updated to apply it as a pace penalty with normalized pace history storage.
See Step 6 for full details of final implementation.

---

## Step 21 — Runner Stats Modal ✓

**Completed.**

- "Runner Stats" button added to footer (left side, blue style)
- Full-screen scrollable modal with 8 runner sections + Team Totals
- Each section: 2-line header (name + stats), column headers, 3 leg rows
- Runner with 0 completed legs: summary line italic
- Runner with 1+ completed legs: summary line regular weight
- Completed legs: bold white data; unstarted legs: italic gray
- Color bar (4px, muted loop color) on left edge of each leg row
- Asterisk on pace (`12:30/mi*`) for legs with overridden loop miles
- Team Totals: legs complete, total run time, team avg pace
- **Grouping:** by current `runner` field, not `defaultRunner`

**Bug fixed:** Runner Stats was initially grouping by `defaultRunner`
instead of `runner`, causing substituted legs to appear under the wrong
runner. Fixed to use `runner` throughout.

---

## Step 22 — Export Button Moved to Settings ✓

**Completed.** Export CSV button moved from footer to Backup Status
section in Settings panel. Footer now shows Runner Stats button instead.

---

## Step 23 — Backup Folder as Tappable Link ✓

**Completed as part of Step 15 implementation.** Folder name in Settings
Backup Status section is a tappable blue link that re-opens the folder
picker.

---

## Step 24 — Loop Miles Override ✓

**Completed.**

- `overrideMiles` field added to leg data (null by default)
- `getEffectiveMiles(leg, config)` helper used throughout all calculations
- Loop Miles field in Row Detail Modal shows dotted border
- Press and hold 3 seconds (completed legs only) opens numeric keypad
- Override value displayed in green (`#4caf50`) when set
- Reset: press and hold again, enter config default value
- Asterisk appended to pace in Runner Stats for overridden legs
- CSV exports effective miles (override if set, default otherwise)
- Persisted in localStorage under `raceLegs`

---

## Step 25 — Runner Stats Grouping Fix ✓

**Completed as part of Step 21 bug fix.** All Runner Stats grouping,
pace history, and stat calculations use `runner` field (current assigned
runner), not `defaultRunner`. Substituted legs correctly attributed to
the substituting runner.

---

## Build Session: April 18, 2026
### Bug fix + code review

---

## Bug Fix — Runner Substitution Lost on Settings Save ✓

**Discovered during UAT. Fixed by Claude Code. Commit: bfdea7f**

**Bug:** Substituting a runner via the Row Detail Modal worked correctly
and persisted as expected — until the user opened Settings and tapped
Save (even without making any changes). On Save, all runner assignments
reverted to their config defaults, erasing any substitutions.

**Root cause:** `initRaceLegs(preserveActuals=true)` — called on every
Settings Save — already preserved `actualFinish` and `overrideMiles`
from existing leg data, but unconditionally recalculated the `runner`
field from config defaults, overwriting any manual substitutions.

**Fix:** Extended the `preserveActuals` pattern to include `runner`.
When `preserveActuals` is true and an existing `runner` value is present,
keep it. Otherwise fall back to the default slot assignment.

**Scope:** Settings Save path only. Fresh load and explicit Reset still
derive runner names from config as expected.

**UAT note:** Section 14 of the UAT Checklist does not currently cover
this scenario. Test 14.8 should be added: substitute a runner, open
Settings, save without changes, verify substitution persists in the
table and Runner Stats.

---

## Code Review — April 18, 2026

A three-reviewer code review was conducted on the full source file.
Reviewers: Gemini, Claude Code, Claude. Findings documented in full in
`Race_Tracker_Code_Review_Actions_v1.md`.

**Summary of findings:**

Three items are flagged as bugs to fix before race day:

- **CR-01 — Runner name HTML/JS injection (HIGH):** Runner names inserted
  directly into `onclick="selectRunner('${r}')"` template strings. A name
  containing an apostrophe (e.g. O'Brien) breaks the dropdown entirely.
  Fix: use `data-*` attributes and `addEventListener`; add `escapeHtml()`
  helper for all name insertions into `innerHTML`.

- **CR-02 — CSV restore naive comma split (HIGH):** `restoreFromCSV()`
  splits rows with `row.split(',')`, which breaks on team or event names
  containing commas. `generateCSV()` correctly quotes these values on
  export, but restore doesn't handle them. Fix: replace with a proper
  RFC 4180 CSV row parser.

- **CR-03 — Backup files accumulate without limit (MEDIUM):** Every write
  to a new minute creates a new timestamped CSV file. A 30-hour race
  produces ~1,800 files with no cleanup. Fix: use a single canonical
  filename (`ragnar_tracker_current.csv`) for automatic backup; reserve
  timestamped names for manual Export CSV only.

Four architecture improvements are identified as high value but not urgent:

- **CR-04:** Centralize app state into one in-memory object; stop reading
  localStorage inside render and compute functions.
- **CR-05:** Make `calculateLegData()` a pure function — accept config
  and raceLegs as arguments, no internal storage reads.
- **CR-06:** Route all saves through one unified pipeline so no write
  path can bypass backup or localStorage.
- **CR-07:** Store config as native types (numbers); format to display
  strings at the render layer only.

Seven polish items are noted (CR-08 through CR-14): custom confirm
modals, AM/PM sanity check on time entry, backup write concurrency
guard, CSS variables, console.log cleanup, minute timer alignment, and
long-press discoverability for loop miles override.

**Architecture decisions reaffirmed during review:**
Three reviewers independently suggested generating `LEG_DEFINITIONS`
dynamically. This is contrary to the documented decision that
`LEG_DEFINITIONS` is a hardcoded static constant — never derived. That
decision stands. See Decisions Log for rationale.

---

## Remaining Steps

### Step 26 — UAT Testing
- UAT Checklist v2 written and available in /docs
- Testing to be conducted on Android tablet (primary target device)
- Key areas: all features end-to-end, loop miles override, runner
  substitution in Runner Stats, night pace adjustment accuracy
- **Pre-UAT:** Add test 14.8 (runner substitution persists through
  Settings save) before running checklist
- **Pre-UAT:** Fix CR-01 (runner name injection) and CR-02 (CSV restore)
  before UAT if possible — both are latent bugs that could surface during
  testing

### Step 27 — Race Day Kiosk Setup Verification
- End-to-end setup walkthrough on Android tablet
- Screen Pinning verification (still pending — not blocking)
- File System Access API permission persistence on Android

---

## Known Issues / Watch List

- **CR-01 — Runner name apostrophe bug** — A runner name containing an
  apostrophe (e.g. O'Brien) will break the runner substitution dropdown.
  Fix documented in Race_Tracker_Code_Review_Actions_v1.md. Fix before
  race day.
- **CR-02 — CSV restore comma bug** — A team or event name containing a
  comma will restore corrupted. Fix documented in
  Race_Tracker_Code_Review_Actions_v1.md. Fix before race day.
- **CR-03 — Backup file accumulation** — Timestamped backup files
  accumulate indefinitely with no cleanup. Fix documented in
  Race_Tracker_Code_Review_Actions_v1.md. Fix before race day.
- **Tim's Leg 8 pace (8.20/mi)** — suspiciously fast for a trail race.
  Needs verification against actual race records before using 2025 data
  as a forecasting baseline.
- **Android Screen Pinning** — not yet fully verified on the ECOPAD tablet.
  Not blocking — app functions correctly without it.
- **File System Access API on Android** — permission persistence after
  full Chrome close needs verification on the ECOPAD tablet.
- **Night Pace Adjustment default (5%)** — working value pending
  validation against historical race data before next race.

---

## Post-MVP Items (noted, not scheduled)

- Instruction document for end users
- In-app help / info buttons
- iOS support
- Individual runner stats already built — no further items here

---

*Build Log version: 4.0 — Updated April 18, 2026*
*Covers: Steps 1–25 complete. Steps 26–27 remaining.*
*April 18 additions: runner substitution bug fix (commit bfdea7f),
code review summary, CR-01/CR-02/CR-03 added to Known Issues.*
*Companion to: Race Tracker App Plan v7, Race Tracker Data Model v3,*
*Race Tracker Decisions Log v3, Race Tracker Code Review Actions v1*
