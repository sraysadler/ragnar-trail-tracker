# Race Tracker — Decisions Log

A record of key decisions made during planning and build, including the
reasoning behind each. When a judgment call arises during the build,
check here before making assumptions.

---

## Architecture

**Single HTML file**
Chosen over a multi-file web app, PWA, or native app. The race happens
at a campsite with no reliable internet. A single file requires no
installation, no server, no build process, and no network connection.
It can be loaded onto the device in multiple ways and works immediately
in Chrome. Simplicity and reliability outweigh any developer convenience
that a framework or build tool would provide.

**Vanilla JavaScript — no frameworks**
React, Vue, and similar frameworks require a build step and introduce
external dependencies. Both are incompatible with the single-file,
offline-first requirement. The app logic is well-defined enough that
a framework adds no meaningful benefit.

**Android + Chrome as primary target**
Chrome on Android supports the File System Access API, which allows
automatic background CSV writes without user interaction after the
initial one-time permission grant. iOS does not support this API
regardless of browser, due to Apple's WebKit requirement. Android
was chosen as the primary target specifically because of this
backup reliability advantage.

**localStorage as primary data store**
All race data is written to localStorage after every entry. This
survives browser restarts and accidental tab closures. It was chosen
over in-memory-only storage because losing 20+ legs of data to an
accidental browser close during a 28-hour race is unacceptable.

**IndexedDB for File System Access API folder handle**
The File System Access API folder handle cannot be stored in
localStorage — it is not JSON-serializable. IndexedDB is used
specifically for this purpose. All other app data remains in
localStorage. This separation was discovered during the build
and is a technical requirement, not a design preference.

**Three-layer backup strategy**
localStorage alone is insufficient because it can be cleared. The
File System Access API automatic backup adds a second layer that
writes to the device file system independently of the browser.
The manual export button adds a third layer the user controls.
All three are required — not optional.

---

## Race Format

**24 rows, one per leg — not 8 rows, one per runner**
The original instinct was an 8-row table with one row per runner.
This was rejected because timing logic depends on each individual
handoff, not on each runner as a unit. A 24-leg sequential model
is the correct representation of how the race actually works and
makes cascading start time calculations clean and unambiguous.

**Leg count and runner count are fixed in code**
The app is built specifically for the Ragnar Trail format: 8 runners,
3 loops each, 24 legs. Making these configurable would add complexity
without meaningful benefit. Teams short a runner handle the situation
through the runner substitution mechanism, not by changing leg count.

**Loop order is fixed: Green → Yellow → Red**
This is the Ragnar Trail race format. It does not vary. Hardcoding
it is correct.

**LEG_DEFINITIONS is a static constant — never derived**
Loop color, loop initial, and row background color for all 24 legs
are defined as a hardcoded JavaScript constant. They are never
computed from config values. This avoids unnecessary logic and
processing for values that never change.

---

## Config & Setup

**Single "Estimated Team Average Pace" field, not separate baseline inputs**
Earlier drafts showed separate fields for average loop time and average
loop miles, from which baseline pace was derived. This was simplified
to a single pace-per-mile field because it is more intuitive for users
to enter a number they understand ("our team runs about 12:00 miles")
than to reconstruct it from loop duration and distance averages.

**Loop distances are configurable, not hardcoded**
Ragnar Trail loop distances are approximate and vary year to year.
The actual distances are not confirmed until race day. Default values
(Green 4.0, Yellow 5.0, Red 6.0) are starting points only.

**Night pace adjustment is configurable as a percentage**
Initial data from the 2025 race suggested runners were approximately
30 seconds per mile slower during night loops (roughly 8pm–7am).
This is a config field with a working default of 5%. For MVP, only
percentage input is supported. The `nightAdjustmentType` field is
retained in the data structure to allow future MM:SS per mile support
without a schema change.

**Night hours window is configurable**
8:00 PM – 7:00 AM is the default, but conditions vary by race location
and time of year. Two separate tappable time fields in Config allow
this window to be adjusted.

**All config fields use custom keypads — no system keyboard for numeric
and time fields**
Runner name fields use the system keyboard (text input is appropriate
there). All other config fields — start time, loop distances, avg pace,
night adjustment, night hours — open custom keypads appropriate to
their data type. This was scoped as a separate build step after the
race table keypad was confirmed working.

**All config values included in every CSV backup**
If a restore from backup is ever needed, the full config state must
be recoverable alongside the race data. Separating them would create
a situation where race data restores correctly but config does not.

---

## User Interface

**Dark mode throughout**
The race runs for approximately 28 hours, including overnight. Runners
approaching the kiosk at 2am need a screen that does not blind them.
Dark mode also conserves battery on OLED/AMOLED screens. Pure black
(#000000) background was chosen as the base.

**Loop row colors: #003300 / #444400 / #330000**
Dark-tinted versions of green, yellow, and red. Yellow was lightened
from the original planned value of #333300 to #444400 during the build
after it proved too dark to distinguish from the green rows at a glance.
Green and red remained at their original values.

**Estimated times: italic, #999 gray**
Distinguishes estimated from actual without making estimates hard
to read. Pure white italic was tried and was too similar to confirmed
bold white. Pure dark gray (#555) was tried and was too dim. #999
was the agreed middle ground.

**Actual times: bold, white**
Bold white is the strongest visual signal available in the dark
mode palette. Confirmed times should be immediately obvious at
a glance, especially when scrolling through 20+ rows.

**Projected team finish gets its own highlighted bar**
This is the number the team cares most about at any given moment.
Giving it a dedicated full-width green bar above the three-column
status row ensures it is never missed or confused with other status
information.

**Status bar shows: Current loop (leg# + runner), Next runner,
Next start time**
These three pieces of information answer the question every team
member has when they walk up to the kiosk: "Who is out right now,
who is next, and when do they start?" The projected finish answers
the overarching team question. Together these four data points cover
all immediate race management needs.

**Single Settings button in footer only**
Having Settings in both header and footer created visual clutter.
The current time was moved into the header where Settings used to
be, since knowing the current time is more useful at a glance than
a second path to settings.

**Current time displayed in header**
Runners need to know the current time to understand how a leg is
progressing. Displaying it in the header means it is always visible
without requiring any interaction.

**"Last saved" as permanent footer indicator — no flash message**
The original plan called for a brief "Race data restored" flash message
on app launch when localStorage data was found. During the build this
was replaced with a permanent "Last saved: [time]" indicator in the
footer. This approach is simpler, more informative, and always visible
without requiring a flash message mechanism. The indicator updates on
every save and serves as continuous reassurance that data is current.

---

## Data Entry

**Custom numeric keypad for all time entry**
The system keyboard was explicitly ruled out. On a touchscreen
kiosk used by tired runners, a custom keypad with large buttons
eliminates formatting errors, inconsistent input, and the visual
disruption of the system keyboard appearing. The keypad is also
faster for the specific task of entering a 4-digit time.

**Calculator-style right-to-left digit entry**
Digits fill from the right and shift left as new digits are entered.
This is the most natural pattern for entering a time value and
matches the mental model of typing a number on a calculator.

**Intermediate display states during entry are expected and correct**
While typing (e.g. `--:65` while entering `6:54`), the display shows
partially-formed values. This is correct calculator-style behavior —
the digits resolve to a valid time as entry completes. These
intermediate states are not validation errors.

**Both 12-hour and 24-hour input accepted, always displayed as
12-hour**
Runners may naturally think in either format. Accepting both
eliminates input errors. The display always shows 12-hour format
for consistency and readability on the kiosk.

**AM/PM auto-detected from digit input**
If the entered hour exceeds 12, AM/PM flips automatically. The
toggle button allows manual override.

**Keypad pre-populated with existing field value**
If a time has already been entered, the keypad opens showing that
time. If the field is empty, it defaults to the current device time.

**Clear mechanism replaces separate confirmation prompt**
The original plan called for a separate confirmation dialog when
clearing an entered time. During the build this was replaced with
a more elegant keypad-native approach: when all digits are backspaced
from an existing entry, the Confirm button changes to a red "Clear"
button. Tapping Clear removes the time. Beginning to type new digits
reverts Clear back to Confirm. This is less disruptive than a dialog
and makes the intention of clearing explicit through the UI state.

---

## Row Detail Modal

**Modal implemented as a floating card, not a full-screen panel**
The original plan specified the modal as a full-screen panel replacing
the main view. During the build the first implementation was full-screen
and the user requested it be resized to match the keypad and Settings
panel pattern — a centered card overlaid on a dimmed background.
This approach keeps the race table visible behind the modal, providing
context, and feels lighter and less disruptive for a detail view.

**Modal opened by tapping anywhere on a row except the Actual
Finish cell**
Tapping the Actual Finish cell directly opens the time keypad —
the most common action — without the extra step of opening the
modal first. Tapping elsewhere on the row opens the modal for
detail viewing and secondary edits.

**Modal title shows leg number and loop color, not runner name**
"Leg 6 — Red" is the correct title. The runner name is shown
as an editable field within the modal. Putting the runner name
in the title would make it feel static when it is actually editable.

**Modal field order**
The final field order in the modal differs slightly from the original
spec. After review during the build, Est Duration was moved before
Actual Duration so that estimated and actual values for the same
concept (duration) are grouped with their context:

Runner → Loop Miles → Start → Est Finish → Actual Finish →
Status → Est Duration → Actual Duration → Predicted Pace → Actual Pace

**Runner substitution uses a custom dark-styled dropdown**
The native browser `<select>` element was replaced with a custom
dropdown that matches the app's dark UI. The native dropdown appeared
visually inconsistent with the rest of the app — light background,
system styling. The custom dropdown uses dark backgrounds, blue
highlight for the selected runner, checkmark indicator, and a chevron
that flips when open.

**Actual Finish saves immediately when keypad confirms**
Actual Finish changes made via the keypad inside the modal save
immediately when the keypad Confirm is tapped — they do not wait
for the modal Save button. The modal Save button is primarily for
runner substitution changes. This distinction was intentional and
is important for understanding the modal's save behavior.

**Modal shows estimated finish pre-populated in the Actual
Finish tappable field**
When no actual finish has been entered, the Actual Finish field
shows the current estimated finish in italic as a starting point.
This reduces the number of digits a runner needs to type if the
actual time is close to the estimate.

---

## Forecasting

**Pace per mile as the core prediction unit, not raw duration**
Loop distances vary between Green, Yellow, and Red. Using pace
per mile allows the model to adapt cleanly when distances change
mid-race. Raw duration averages would produce incorrect estimates
if loop distances are updated after the race starts.

**Simplified forecasting model — runner history only**
The original planned model included a team loop-color average as
one of its weighted inputs (e.g. 60% runner average + 40% team
Green average). During planning this was simplified to use only
the runner's own pace history, falling back to the config baseline
when no history exists.

Reasons for simplification:
- With only 24 legs across 8 runners, the team loop-color sample
  size is too small to be meaningfully predictive
- Runner-specific history is more predictive than a small group
  average for this dataset
- Simpler logic is easier to verify and debug during a live race

Final model:
- 0 prior legs → config baseline pace → Estimated (Baseline)
- 1 prior leg → runner's actual pace from that leg → Estimated (Live)
- 2+ prior legs → equal-weight average of runner's actual paces
  → Estimated (Live)

**Three status values: Actual / Estimated (Baseline) /
Estimated (Live)**
Two values (Actual / Estimated) were considered but rejected.
The three-value system communicates whether an estimate is still
using the original assumptions or has begun learning from real
race data. This distinction is meaningful during a race.

**Night pace adjustment applied to estimated duration, not pace**
The adjustment is a percentage applied to the estimated duration
after multiplying pace × loop miles. It is not applied to the
pace value itself. This keeps actual pace calculations clean
and unaffected by the night adjustment.

**Night pace adjustment applied automatically based on estimated
start time**
If a leg's estimated start time falls within the configured night
window, the pace adjustment is added automatically. The runner
entering the time does not need to think about whether it is a
night leg — the app handles it.

---

## Config Panel Keypad Integration

**Config tappable fields wired to custom keypads as a separate
build step**
The config panel fields were styled as tappable from the start but
the custom keypad wiring was deferred until after the race table
keypad was confirmed working. This was a deliberate sequencing
decision — validate the keypad mechanism on the simpler race table
use case first, then extend it to the more varied config field types.

**Different keypad variants for different config field types**
- Time fields (start time, night hours): standard time keypad with AM/PM
- Distance fields (loop miles): numeric keypad with decimal point
  replacing AM/PM button
- Pace field (avg pace): numeric keypad without AM/PM, with /mi suffix
  shown in the field display
- Percentage field (night adjustment): numeric keypad with decimal point

**Runner name fields use system keyboard**
Text input for runner names is appropriate — names are free-form strings
that cannot be entered with a numeric keypad. The system keyboard is
acceptable here because it is used in a calm, pre-race setup context
rather than during live race data entry.

---

## Reusability

**App is generic — all team/race data entered via Config**
The 2025 race was tracked in a Google Sheet built specifically for
Party Pace at Ragnar Trail Atlanta. The app generalizes this so
any Ragnar Trail team can use it by entering their own details
at setup. Nothing team-specific is hardcoded.

**Designed for reuse across years, not rebuilt each year**
The Config panel allows updating runner names, loop distances,
start time, and pace settings for a new race year. The app itself
does not need to be rebuilt.

---

*Document version: 2.0 — Updated April 16, 2026*
*Companion to: Race Tracker App Plan v6, Race Tracker Wireframes v1,
Race Tracker Do Not Do v1, Race Tracker Data Model v2,
Race Tracker Build Log v2*
