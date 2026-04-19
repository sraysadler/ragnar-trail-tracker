# Ragnar Trail Tracker — UAT Checklist

A test plan for verifying all app functionality before race day.
Complete all sections on the target Android tablet in Chrome.
For each test, mark Pass, Fail, or Note.

Tester name: _______________
Device: _______________
Date: _______________
App URL: https://sraysadler.github.io/ragnar-trail-tracker/ragnar-trail-tracker.html

---

## Section 1 — First Load / Race Setup

**Before starting:** Clear all browser data in Chrome (Settings → Privacy
and security → Clear browsing data → Cookies and site data + Cached images).

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 1.1 | Load the app for the first time | "Race Setup" panel opens automatically | |
| 1.2 | Panel title | Shows "Race Setup" not "Settings" | |
| 1.3 | Runner name fields | All blank | |
| 1.4 | Team name and Event name | Both blank | |
| 1.5 | Default values | Start time 8:00 AM, Green 4 mi, Yellow 5 mi, Red 6 mi, Avg pace 12:00/mi, Night adjustment 5%, Night hours 8:00 PM – 7:00 AM | |
| 1.6 | Backup setup prompt | A banner appears offering to set up automatic backup | |
| 1.7 | Tap "Set Up" on backup prompt | Folder picker opens | |
| 1.8 | Select a backup folder | Banner dismisses, confirmation appears | |

---

## Section 2 — Config / Race Setup

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 2.1 | Enter Team Name | System keyboard opens, text input works | |
| 2.2 | Enter Event Name | System keyboard opens, text input works | |
| 2.3 | Tap Start Time field | Time keypad opens | |
| 2.4 | Enter start time 9:00 AM | Displays correctly in field | |
| 2.5 | Enter all 8 runner names | All names accepted and displayed | |
| 2.6 | Tap Green Miles field | Numeric keypad opens | |
| 2.7 | Enter Green Miles — type 4, 2 | Displays "4.2 mi" (implied decimal) | |
| 2.8 | Tap Avg Pace field | Pace keypad opens | |
| 2.9 | Enter pace — type 1, 2, 0, 0 | Displays "12:00 /mi" | |
| 2.10 | Tap Night Adjustment field | Numeric keypad opens | |
| 2.11 | Enter 5, 0 | Displays "5.0%" (implied decimal) | |
| 2.12 | Tap Night Hours From field | Time keypad opens | |
| 2.13 | Enter 8:00 PM | Displays correctly | |
| 2.14 | Tap Night Hours To field | Time keypad opens | |
| 2.15 | Enter 7:00 AM | Displays correctly | |
| 2.16 | Tap Save | Settings closes, header shows team name and event name | |
| 2.17 | Reopen Settings | Title now shows "Settings" not "Race Setup" | |
| 2.18 | Runner names in race table | All 24 legs show correct runner names in rotation | |
| 2.19 | Change a runner name in Settings, tap Save | All legs for that runner slot update to the new name in the race table | |

---

## Section 3 — Race Table Display

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 3.1 | Row colors | Green legs green, yellow legs yellow/amber, red legs red | |
| 3.2 | Estimated times | Italic, gray (#999) | |
| 3.3 | Start time Leg 1 | Matches race start time from config | |
| 3.4 | Cascading estimates | Each leg's start = previous leg's estimated finish | |
| 3.5 | Night legs | Legs starting between 8 PM and 7 AM show slightly longer estimated durations than equivalent day legs at same pace | |
| 3.6 | Column headers | #, Runner, LP, Start, Est Fin, Actual — readable and sized appropriately | |
| 3.7 | Column headers frozen | Scroll table down — headers stay visible at top | |
| 3.8 | Leg 1 active outline | White outline visible on first row | |

---

## Section 4 — Time Entry Keypad (from race table)

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 4.1 | Tap Actual Finish cell for Leg 1 | Time keypad opens directly (no modal) | |
| 4.2 | Keypad header | Shows "Leg 1 – [Runner name]" | |
| 4.3 | Default value | Shows current device time | |
| 4.4 | Enter 9, 5, 0 | Displays 9:50 AM (or PM depending on AM/PM state) | |
| 4.5 | AM/PM toggle | Tapping AM/PM button switches between AM and PM | |
| 4.6 | 24-hour input | Type 2, 1, 3, 0 → displays 9:30 PM | |
| 4.7 | Backspace | Removes last digit, shifts display right | |
| 4.8 | Confirm 9:50 AM | Leg 1 shows bold "9:50 AM" in Actual column | |
| 4.9 | Cascade update | Leg 2 start updates to 9:50 AM | |
| 4.10 | Status update | Leg 1 status becomes Actual, Leg 2 becomes Estimated (Live) | |
| 4.11 | Cancel | Returns to table, no change made | |
| 4.12 | Active leg outline | Moves to Leg 2 after Leg 1 actual is entered | |

---

## Section 5 — Status Bar

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 5.1 | Projected team finish | Shows estimated finish time of Leg 24 | |
| 5.2 | Current loop | Shows "1 – [Runner 1]" before any actual times entered | |
| 5.3 | Next runner | Shows Runner 2 name | |
| 5.4 | Next start | Shows Leg 2 estimated start time | |
| 5.5 | After entering Leg 1 actual | Current loop updates to "2 – [Runner 2]" | |
| 5.6 | Current time | Shows correct device time, updates every minute | |
| 5.7 | Last saved indicator | Shows time of last save after entering actual | |
| 5.8 | Green dot | Appears immediately left of "Last saved" text after backup writes | |

---

## Section 6 — Row Detail Modal

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 6.1 | Tap anywhere on a row (not Actual cell) | Row Detail Modal opens as floating card | |
| 6.2 | Modal title | Shows "Leg [N] — [Loop Color]" | |
| 6.3 | All fields visible | Runner, Loop Miles, Start, Est Finish, Actual Finish, Status, Est Duration, Actual Duration, Predicted Pace, Actual Pace | |
| 6.4 | Loop Miles field | Shows dotted border indicating it is interactable | |
| 6.5 | Status color | Actual = green, Estimated (Baseline) = gray, Estimated (Live) = blue | |
| 6.6 | Actual Finish blue box | Shows Est Finish in italic when no actual entered | |
| 6.7 | Tap Actual Finish blue box | Keypad opens on top of modal | |
| 6.8 | Keypad Cancel from modal | Returns to modal, not table | |
| 6.9 | Keypad Confirm from modal | Modal updates with new time, actual duration and pace calculated | |
| 6.10 | Runner dropdown | Tap runner field — list of all 8 runners appears | |
| 6.11 | Change runner | Select different runner, tap Save changes — table updates | |
| 6.12 | Cancel with unsaved runner change | "Close without saving changes?" prompt appears | |
| 6.13 | Close via X button | Modal closes, returns to table | |
| 6.14 | Tap dimmed background | Modal closes | |

---

## Section 7 — Forecasting Logic

Enter actual finish times for Legs 1–8 using the 2025 sample data:

| Leg | Runner | Actual Finish |
|-----|--------|--------------|
| 1 | Wendy | 9:50 AM |
| 2 | Kristine | 11:08 AM |
| 3 | Scott | 12:20 PM |
| 4 | Jeff | 1:13 PM |
| 5 | Jason | 2:21 PM |
| 6 | Ray | 3:35 PM |
| 7 | Jenny | 4:21 PM |
| 8 | Tim | 5:02 PM |

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 7.1 | All runners now have 1 completed leg | All remaining legs show Estimated (Live) status | |
| 7.2 | Open modal for Leg 9 (Wendy, Red) | Predicted pace reflects Wendy's actual pace from Leg 1, not baseline | |
| 7.3 | Open modal for Leg 10 (Kristine) | Predicted pace reflects Kristine's actual pace from Leg 2 | |
| 7.4 | Night legs | Legs starting after 8 PM show longer estimated durations than equivalent day legs at same runner pace | |
| 7.5 | Enter Leg 9 (Wendy) actual | Leg 17 predicted pace updates to average of Wendy's normalized Leg 1 and Leg 9 paces | |

---

## Section 8 — Clearing a Finish Time

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 8.1 | Tap Actual cell for a completed leg | Keypad opens showing entered time | |
| 8.2 | Backspace all digits | Display shows "--:-- AM", Confirm changes to red "Clear" | |
| 8.3 | Tap Clear | Leg reverts to italic estimated time | |
| 8.4 | Downstream cascade | Next leg's start time adjusts, estimates recascade | |
| 8.5 | Begin typing new digits | "Clear" reverts back to green "Confirm" | |

---

## Section 9 — Race Complete State

Enter all 24 actual finish times (use 2025 sample data from
docs/Race_Tracker_Sample_Data_2025.csv).

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 9.1 | After Leg 24 entered | Status bar changes to Race Complete state | |
| 9.2 | Projected finish bar | Label changes to "FINAL TEAM FINISH" | |
| 9.3 | Current loop | Shows "Complete" | |
| 9.4 | Next runner / Next start | Both show "—" | |
| 9.5 | All rows bold | All 24 Actual cells show bold white times | |
| 9.6 | Clear Leg 24 actual | Race Complete state reverts to In Progress automatically | |

---

## Section 10 — Settings: Reset, Restore, and Export

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 10.1 | Tap Reset | Confirmation dialog appears | |
| 10.2 | Confirm Reset | All config fields revert to defaults. Race table updates. | |
| 10.3 | Tap Restore | Confirmation dialog appears with most recent backup filename | |
| 10.4 | Confirm Restore | Settings closes, footer shows "Restored from backup [filename]", table re-renders | |
| 10.5 | Backup Status section | Shows folder name as tappable blue link and last backup timestamp | |
| 10.6 | Tap folder name link | Folder picker opens to change backup folder | |
| 10.7 | Export CSV button | Visible in Backup Status section of Settings | |
| 10.8 | Tap Export CSV | CSV file downloads, "Exported successfully" appears briefly in footer | |
| 10.9 | Exported CSV content | Three sections: Race Summary, Race Data, Config. Avg Pace formatted correctly (not NaN). | |

---

## Section 11 — Data Persistence

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 11.1 | Enter several actual finish times | Note the values | |
| 11.2 | Close Chrome completely | — | |
| 11.3 | Reopen Chrome and navigate to app | All entered times restored, estimates recalculated | |
| 11.4 | Last saved time | Shows time of last save before closing | |

---

## Section 12 — Automatic File Backup

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 12.1 | Backup folder configured | Green dot visible in footer next to Last saved | |
| 12.2 | Enter an actual finish time | New CSV file appears in backup folder | |
| 12.3 | Filename format | ragnar_tracker_YYYYMMDD_HHMM.csv | |
| 12.4 | Enter another time 1 minute later | Second CSV file appears — first file unchanged | |
| 12.5 | Close Chrome completely and reopen | Permission persists, next entry still writes backup | |
| 12.6 | Settings backup status | Shows correct last backup timestamp and folder name | |

---

## Section 13 — Runner Stats Modal

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 13.1 | Tap "Runner Stats" in footer | Runner Stats modal opens | |
| 13.2 | Footer button style | "Runner Stats" shows in blue, distinct from "Settings" | |
| 13.3 | All 8 runners visible | Scroll through modal — all 8 runner sections present | |
| 13.4 | Runner with completed legs | Summary line: regular weight. e.g. "2 of 3 complete · avg 12:30/mi" | |
| 13.5 | Runner with no completed legs | Summary line: italic. e.g. "0 of 3 complete · avg 12:00/mi" | |
| 13.6 | Completed leg rows | Bold white data for all columns | |
| 13.7 | Unstarted leg rows | Italic gray data for all columns | |
| 13.8 | Color bars | Left edge of each leg row shows correct muted loop color | |
| 13.9 | Leg numbers correct | Each runner shows their correct leg numbers (e.g. Runner 1: legs 1, 9, 17) | |
| 13.10 | Duration format | Shows h:mm only (e.g. "0:50", "1:18") — no seconds | |
| 13.11 | Pace format | Shows MM:SS/mi (e.g. "12:30/mi") | |
| 13.12 | Team Totals section | Shows at bottom: legs complete, total run time, team avg pace | |
| 13.13 | Team Totals — no data | Legs complete shows 0, time and pace show "—" | |
| 13.14 | Close button | ✕ Close returns to main view | |

---

## Section 14 — Runner Substitution

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 14.1 | Open modal for a leg with default runner | Runner dropdown shows current runner | |
| 14.2 | Change runner to different name | Dropdown closes, new name shown | |
| 14.3 | Tap Save changes | Table row updates with new runner name | |
| 14.4 | Forecasting update | Future legs for substituting runner use their combined pace history | |
| 14.5 | Runner Stats — substituted legs | All legs assigned to substituting runner appear under their section | |
| 14.6 | Runner Stats — original runner | Only shows legs still assigned to original runner | |
| 14.7 | Runner Stats avg pace | Reflects all completed legs for current runner assignment | |
| 14.8 | Substitution persists through Settings save | Substitute a runner on any leg. Open Settings, tap Save without making any changes. Verify the substituted runner name still appears on that leg in the table and in Runner Stats — not the original default runner. | |
| 14.9 | Unchanged runner slot — substitution preserved | Substitute Runner 2 onto a leg. Change Runner 1's name in Settings, tap Save. Verify Runner 1's legs update to the new name AND Runner 2's substitution on that leg is still in place. | |
| 14.10 | Changed runner slot — substitution overridden | Substitute Runner 2 onto one of Runner 1's legs. Change Runner 1's name in Settings, tap Save. Verify that substituted leg now shows the new Runner 1 name (substitution correctly overridden by the name change). | |

---

## Section 15 — Loop Miles Override

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 15.1 | Loop Miles field in modal | Always shows dotted border | |
| 15.2 | Press and hold on incomplete leg | Nothing happens — override not available | |
| 15.3 | Press and hold 3 seconds on completed leg | Numeric keypad opens | |
| 15.4 | Keypad pre-populated | Shows current effective miles (config default if no override) | |
| 15.5 | Enter new value (e.g. type 7, 0 for 7.0 mi) | Keypad shows 7.0 | |
| 15.6 | Confirm override | Modal shows 7.0 mi in green | |
| 15.7 | Actual pace recalculated | Pace now based on 7.0 miles — lower than before | |
| 15.8 | Future leg estimates update | Next leg for this runner uses corrected pace in history | |
| 15.9 | Other runners unaffected | Other legs still show config default miles | |
| 15.10 | Runner Stats asterisk | Pace for overridden leg shows asterisk: e.g. "12:30/mi*" | |
| 15.11 | CSV export | Overridden leg shows 7.0 in Loop Miles column | |
| 15.12 | Persistence | Refresh browser — override still shows in green | |
| 15.13 | Reset override | Press and hold again, enter config default. Miles reverts to white. | |

---

## Section 16 — Edge Cases

| # | Test | Expected Result | Result |
|---|------|-----------------|--------|
| 16.1 | Midnight crossing | Legs crossing midnight display correct AM times (e.g. 1:06 AM, not 25:06) | |
| 16.2 | Night window boundary | Leg starting at 7:59 PM — no night adjustment. Leg at 8:01 PM — adjustment applied. | |
| 16.3 | After 7:00 AM | Legs starting after 7:00 AM — no night adjustment | |
| 16.4 | Long team name | Header clips or wraps without breaking layout | |
| 16.5 | All legs estimated | No actual times entered — all 24 rows show italic estimated times | |
| 16.6 | Multiple runner substitutions | One runner covering 5 legs — all 5 appear under their section in Runner Stats | |

---

## Notes / Issues Found

Use this section to record any issues discovered during testing:

| # | Description | Severity (High/Med/Low) | Status |
|---|-------------|------------------------|--------|
| | | | |
| | | | |
| | | | |

---

## Sign-off

All tests passed: Yes / No

If No, list outstanding issues:

Tester signature: _______________
Date: _______________

---

*UAT Checklist version 4.0 — April 18, 2026*
*Changes from v3: Added test 2.19 (runner name change propagates to table);
added tests 14.9 and 14.10 (runner name propagation interaction with
substitutions — unchanged slots preserve substitutions, changed slots
override them).*
*Ragnar Trail Tracker — github.com/sraysadler/ragnar-trail-tracker*
