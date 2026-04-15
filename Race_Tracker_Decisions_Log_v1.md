# Race Tracker — Decisions Log

A record of key decisions made during planning, including the reasoning
behind each. When a judgment call arises during the build, check here
before making assumptions.

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

**Night pace adjustment is configurable**
Initial data from the 2025 race suggested runners were approximately
30 seconds per mile slower during night loops (roughly 8pm–7am).
Rather than hardcoding a number that may vary by year and team,
this is a config field with a working default of 5%. The field
accepts either MM:SS per mile or a percentage. The default should
be validated against historical race data before the next race.

**Night hours window is configurable**
8:00 PM – 7:00 AM is the default, but conditions vary by race location
and time of year. Two separate tappable time fields in Config allow
this window to be adjusted.

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

**Loop row colors: #003300 / #333300 / #330000**
Dark-tinted versions of green, yellow, and red chosen to be visible
against the black background without being bright enough to cause
eye strain at night. These are starting values and can be adjusted
during build if they prove too subtle or too bright on the actual device.

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

**Both 12-hour and 24-hour input accepted, always displayed as
12-hour**
Runners may naturally think in either format. Accepting both
eliminates input errors. The display always shows 12-hour format
(hh:mm AM/PM) for consistency and readability on the kiosk.

**AM/PM auto-detected from digit input**
If the entered hour exceeds 12, AM/PM flips automatically. The
toggle button allows manual override. This means a runner can
type 2315 and get 11:15 PM without touching the AM/PM button.

**Keypad pre-populated with existing field value**
If a time has already been entered, the keypad opens showing that
time. If the field is empty, it defaults to the current device time.
The display clears and rebuilds as soon as the first digit is tapped.
This reduces the number of taps needed to make a small correction.

**Confirmation required before saving time entry**
A dedicated Confirm button prevents accidental saves. Given that an
incorrect time cascades through all downstream estimates immediately,
the one extra tap is worth the protection.

**Clearing an entered time requires confirmation**
"Clear finish time for Leg [N]? It will revert to estimated." appears
before any actual finish time is deleted. Accidental deletion of a
confirmed time mid-race could cause confusion.

---

## Row Detail Modal

**Modal opened by tapping anywhere on a row except the Actual
Finish cell**
Tapping the Actual Finish cell directly opens the time keypad —
the most common action — without the extra step of opening the
modal first. Tapping elsewhere on the row opens the modal for
detail viewing and secondary edits.

**Modal title shows leg number and loop color, not runner name**
"Leg 6 — Red" is the correct title. The runner name is shown
as an editable field within the modal. Putting the runner name
in the title would make it feel static when it is actually
editable.

**Runner substitution handled via modal dropdown**
The Runner field in the Row Detail Modal shows a dropdown of
all 8 team members. This is the correct place for substitutions
because it is a deliberate action that should not happen
accidentally. It is not directly editable from the main table.

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

**Weighted blending model with three tiers**
The forecasting model was chosen over a simpler flat average because
the team has runners with meaningfully different paces. A flat team
average would consistently over- or underestimate fast and slow runners.
The weighted model self-corrects after a runner's first leg without
requiring any per-runner input at setup.

Tier weights:
- 0 prior legs: 70% team loop-color average + 30% config baseline
  (or 100% baseline if no team data exists)
- 1 prior leg: 60% runner average + 40% team loop-color average
  (or 40% baseline if no loop-color data)
- 2+ prior legs: 75% runner average + 25% team loop-color average
  (or 25% baseline if no loop-color data)

**Three status values: Actual / Estimated (Baseline) /
Estimated (Live)**
Two values (Actual / Estimated) were considered but rejected.
The three-value system communicates whether an estimate is still
using the original assumptions or has begun learning from real
race data. This distinction is meaningful during a race and
worth the small added complexity.

**Night pace adjustment applied automatically based on estimated
start time**
If a leg's estimated start time falls within the configured night
window, the pace adjustment is added automatically. The runner
entering the time does not need to think about whether it is a
night leg — the app handles it.

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

*Document version: 1.0 — April 15, 2026*
*Companion to: Race Tracker App Plan v5, Race Tracker Wireframes v1,
Race Tracker Do Not Do v1*
