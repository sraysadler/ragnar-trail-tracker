# Race Tracker — Do Not Do List

Decisions that were explicitly considered and rejected during planning.
This list exists to prevent well-intentioned changes from reintroducing
approaches that were already evaluated and ruled out.

---

## Architecture & Technology

**Do not use a framework (React, Vue, Angular, etc.)**
The app must be a single self-contained HTML file with vanilla HTML, CSS,
and JavaScript only. No build tools, no bundlers, no package managers,
no node_modules. If it can't be opened by double-clicking the file, it's
too complex.

**Do not use a backend or server of any kind.**
The app is fully offline. There is no API, no database, no cloud sync.
All data lives in localStorage and the local file system.

**Do not use external CDN dependencies.**
No libraries loaded from external URLs. The app must work with zero
network connection. Everything needed must be self-contained in the file.

**Do not deploy to Vercel, Netlify, or any hosting platform.**
This is not a web app that lives at a URL. It is a local file loaded
directly into Chrome. Hosting and deployment pipelines are not relevant.

**Do not build a native app or submit to any app store.**
A single HTML file is the correct format. App store submission, APKs,
and native iOS/Android development are out of scope entirely.

**Do not split into multiple files.**
CSS, JavaScript, and HTML all live in one file. Separate .css and .js
files create deployment and loading complexity that defeats the purpose
of the single-file architecture.

---

## Platform & Device

**Do not build for iOS as part of MVP.**
iOS support is a future consideration only. The File System Access API
is not available on iOS regardless of browser. Do not attempt to work
around this during the initial build.

**Do not assume internet connectivity.**
The app must be fully functional with zero network signal. Do not add
any feature that depends on a live connection, even optionally.

**Do not use Airtable, Google Sheets, Google Forms, or any
third-party data platform as part of this app.**
The Google Sheet built for the 2025 race is a separate artifact and
reference point — it is not part of this application.

---

## User Interface

**Do not use the system keyboard for time entry.**
All time input goes through the custom numeric keypad. No text fields
that invoke the device keyboard for time values.

**Do not allow free-text time entry.**
Times are entered only through the custom keypad. This prevents
formatting errors and inconsistent input during a race.

**Do not add an undo button.**
The ability to tap a row and re-edit an already-entered time via the
Row Detail Modal is the correction mechanism. A separate undo button
is redundant and adds UI clutter.

**Do not add manual pace override fields for individual legs.**
There is no field to manually set an estimated duration or pace for
a specific leg. The forecasting model handles all estimates automatically.

**Do not add alerts, push notifications, or alarm-style prompts.**
The status bar is a passive read-only display. It does not interrupt
the user. The only prompts in the app are confirmations for destructive
actions (clear time, reset config) and the iOS backup reminder.

**Do not use bold/italic formatting as the sole indicator of
estimated vs. actual times.**
The distinction is communicated through both styling (bold white for
actual, italic #999 gray for estimated) and the Status field. Relying
on formatting alone is fragile and was explicitly rejected during design.

**Do not show a "restore race" prompt on every app launch.**
localStorage restore is silent and automatic. A prompt only appears
when localStorage is empty but a file backup exists. Normal relaunches
should feel seamless, not alarming.

---

## Data Model & Config

**Do not use two runner columns (Planned Runner + Actual Runner).**
A single editable Runner column per leg is the correct design.
The original plan to keep planned and actual separate was considered
and rejected in favor of simplicity for a phone-first kiosk tool.

**Do not hardcode team name, event name, runner names, or start time.**
All race-specific data is entered through the Config panel. The app
ships with defaults only. Nothing about Party Pace or Ragnar Trail
Atlanta should be baked into the code as non-editable values.

**Do not require per-runner pace input in Config.**
The forecasting model learns runner pace from actual race data.
No setup field asks for estimated pace per runner. The only pace
input is a single team average used as the initial baseline.

**Do not make the leg count or runner count configurable.**
The app is built for the standard Ragnar Trail format: 8 runners,
3 loops each, 24 legs. These values are fixed in the code.
Teams short a member handle this via runner substitution, not
by changing the leg count.

**Do not make loop order configurable.**
Loop order is always Green → Yellow → Red, repeating. This is fixed
in the Ragnar Trail format and fixed in the app.

---

## Backup & Safety

**Do not rely solely on localStorage as the backup strategy.**
localStorage is the primary safety net but not the only one.
The File System Access API automatic backup and manual export
button are both required. Three layers, not one.

**Do not make the automatic file backup optional or hidden.**
On Android + Chrome, the File System Access API backup runs
automatically after every Actual Finish entry. It is not a
setting the user can turn off.

---

*Document version: 1.0 — April 15, 2026*
*Companion to: Race Tracker App Plan v5, Race Tracker Wireframes v1*
