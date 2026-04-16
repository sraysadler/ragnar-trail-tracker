# Build Task: Step 6 — Forecasting Logic

## Objective
Replace the flat baseline pace calculation with a weighted forecasting model
that learns from each runner's actual pace history as the race progresses.
Also implement the night pace adjustment for legs that start within the
configured night hours window.

This step modifies `calculateLegData()` and updates the status logic.
No visual changes — the table already renders correctly. Only the
estimated times and status values will change.

---

## Simplified Forecasting Model

The team loop-color average has been removed from the formula.
The model uses only the runner's own pace history and the config
baseline as fallback.

### Predicted Pace Rules

**Runner has completed 0 prior legs:**
- Predicted pace = config `avgPace`
- Status = `'Estimated (Baseline)'`

**Runner has completed 1 prior leg:**
- Predicted pace = that runner's actual pace from their one completed leg
- Status = `'Estimated (Live)'`

**Runner has completed 2 or more prior legs:**
- Predicted pace = simple average of all that runner's actual paces
  from all completed legs (equal weight, no recency bias)
- Status = `'Estimated (Live)'`

**If actual finish exists for this leg:**
- Status = `'Actual'`
- No predicted pace needed — actual data is used directly

---

## Runner Pace History Calculation

Before calculating any leg's estimated duration, build a runner pace
lookup from all completed legs:

```javascript
function buildRunnerPaceHistory(raceLegs, computedLegs) {
  // Returns an object keyed by runner name
  // Each entry contains array of actual paces from completed legs
  // Example:
  // {
  //   'Wendy': [14.25, 13.80],   // minutes per mile from 2 completed legs
  //   'Ray': [12.50],             // minutes per mile from 1 completed leg
  //   'Jenny': []                 // no completed legs yet
  // }

  const history = {};

  computedLegs.forEach(leg => {
    if (leg.actualFinish && leg.actualPace !== null) {
      if (!history[leg.runner]) history[leg.runner] = [];
      history[leg.runner].push(leg.actualPace);
    }
  });

  return history;
}
```

### Getting predicted pace for a runner

```javascript
function getPredictedPace(runnerName, history, configAvgPace) {
  const paces = history[runnerName] || [];

  if (paces.length === 0) {
    return { pace: configAvgPace, isBaseline: true };
  }

  const avgPace = paces.reduce((sum, p) => sum + p, 0) / paces.length;
  return { pace: avgPace, isBaseline: false };
}
```

---

## Night Pace Adjustment

Applied to any leg whose `startTime` falls within the configured
night hours window.

### Night window logic
The night window can cross midnight (e.g. 8:00 PM to 7:00 AM).
Handle this correctly:

```javascript
function isNightLeg(startTimeMinutes, nightStartMinutes, nightEndMinutes) {
  if (nightStartMinutes > nightEndMinutes) {
    // Window crosses midnight
    return startTimeMinutes >= nightStartMinutes ||
           startTimeMinutes < nightEndMinutes;
  } else {
    return startTimeMinutes >= nightStartMinutes &&
           startTimeMinutes < nightEndMinutes;
  }
}
```

### Applying the adjustment
Night adjustment is stored in config as a percentage (`nightAdjustment: 5`
means 5%). Apply it to estimated duration only (not to actual durations):

```javascript
if (isNight) {
  estimatedDuration = estimatedDuration * (1 + config.nightAdjustment / 100);
}
```

The adjustment applies to the estimated duration AFTER multiplying
pace × loop miles, not to the pace itself. This keeps actual pace
calculations clean and unaffected.

---

## Updated calculateLegData() Flow

The calculation chain now follows this sequence for each leg:

```
1. Build runner pace history from all previously computed legs
   (only legs already processed in this pass — order 1→24)

2. For this leg:
   a. Determine startTime (Leg 1: config start, others: prev finishTime)
   b. Get loopMiles from config based on loop color
   c. Get predicted pace for this runner from history
   d. Calculate base estimatedDuration = predictedPace × loopMiles
   e. Check if this is a night leg based on startTime
   f. If night leg: estimatedDuration *= (1 + nightAdjustment / 100)
   g. Calculate estFinishTime = startTime + estimatedDuration
   h. Set finishTime = actualFinish if set, else estFinishTime
   i. If actualFinish set:
      - Calculate actualDuration = actualFinish - startTime
      - Calculate actualPace = actualDuration / loopMiles
      - Add actualPace to runner history for subsequent legs
   j. Determine status:
      - actualFinish set → 'Actual'
      - no actualFinish, isBaseline → 'Estimated (Baseline)'
      - no actualFinish, not isBaseline → 'Estimated (Live)'

3. Store computed leg object with all fields
4. Move to next leg
```

**Critical:** Runner pace history must be built incrementally as legs
are processed in order. Leg 9 (Wendy's second leg) should use Wendy's
actual pace from Leg 1 when calculating its estimate. Process legs
strictly 1 → 24 and add to history only after each leg is computed.

---

## Status Logic Update

Status now correctly reflects whether the runner has prior history:

```javascript
function getStatus(leg, isBaseline) {
  if (leg.actualFinish) return 'Actual';
  if (isBaseline) return 'Estimated (Baseline)';
  return 'Estimated (Live)';
}
```

Note: A runner on their first leg with no prior history always gets
`Estimated (Baseline)` even if other runners have completed legs.
This is correct and intentional — their estimate is still using
the config baseline, not their own data.

---

## Config Values Used in This Step

```javascript
config.avgPace          // minutes per mile (e.g. 12.0)
config.nightAdjustment  // percentage (e.g. 5)
config.nightStart       // string 'hh:mm AM/PM' (e.g. '8:00 PM')
config.nightEnd         // string 'hh:mm AM/PM' (e.g. '7:00 AM')
config.greenMiles       // number (e.g. 4.0)
config.yellowMiles      // number (e.g. 5.0)
config.redMiles         // number (e.g. 6.0)
```

---

## Verification Steps

Test the following before committing:

1. **Baseline only (no actual times entered):**
   All legs show `Estimated (Baseline)` status.
   Estimated times based purely on config avgPace × loopMiles.

2. **After entering Leg 1 actual finish:**
   Leg 1 → `Actual`
   Legs 2–8 (other runners, no history) → `Estimated (Baseline)`
   Leg 9 (Wendy's second leg) → `Estimated (Live)` with estimated
   time based on Wendy's actual pace from Leg 1, not baseline.

3. **After entering several actual finish times:**
   Runners with completed legs show `Estimated (Live)` for their
   future legs.
   Runners with no completed legs still show `Estimated (Baseline)`.

4. **Night pace adjustment:**
   Enter actual times for Legs 1–12 using the sample data.
   Legs 13 onward (starting around 9:06 PM) should show slightly
   longer estimated durations than day legs of the same distance
   due to the 5% night adjustment.
   Verify Leg 13 (Green, 4 miles) estimated duration is ~5% longer
   than Leg 1 (Green, 4 miles) estimated duration would be at the
   same pace.

5. **Night window crossing midnight:**
   Legs that start after midnight (e.g. Leg 16 starting ~12:26 AM)
   should still receive the night adjustment since they fall within
   the 8:00 PM – 7:00 AM window.
   Legs starting after 7:00 AM should NOT receive the adjustment.

6. **Runner with 2 completed legs:**
   Enter actual times for all of Wendy's first two legs (Legs 1 and 9).
   Wendy's third leg (Leg 17) estimated pace should be the average
   of her Leg 1 and Leg 9 actual paces.

7. **Config change mid-race:**
   Change avgPace in Settings and save.
   Runners with no prior history should update their estimates.
   Runners with actual pace history should NOT change — their
   estimates come from their own data, not config.

8. **Refresh page:**
   All estimates recalculate correctly from localStorage data.
   Night adjustments still apply correctly after reload.

---

## Sample Data for Testing

Use the 2025 race actual finish times for thorough verification:

| Leg | Runner | Actual Finish |
|-----|--------|--------------|
| 1  | Wendy    | 9:50 AM  |
| 2  | Kristine | 11:08 AM |
| 3  | Scott    | 12:20 PM |
| 4  | Jeff     | 1:13 PM  |
| 5  | Jason    | 2:21 PM  |
| 6  | Ray      | 3:35 PM  |
| 7  | Jenny    | 4:21 PM  |
| 8  | Tim      | 5:02 PM  |
| 9  | Wendy    | 6:10 PM  |
| 10 | Kristine | 6:55 PM  |
| 11 | Scott    | 7:53 PM  |
| 12 | Jeff     | 9:06 PM  |

After entering these 12 legs, verify:
- All remaining legs show `Estimated (Live)` (all runners have
  at least one completed leg)
- Night adjustment is visible on legs starting after 8:00 PM
- Each runner's estimated pace for future legs reflects their
  actual pace history, not the config baseline

---

## Key Constraints

- Runner pace history built incrementally, strictly leg 1 → 24
- Night adjustment applied to estimated duration only, not actual
- No team loop-color average — runner history only
- Config baseline used only when runner has zero completed legs
- Single HTML file, no external dependencies
- Full re-render on every change — no targeted cell updates

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Weighted forecasting model replacing flat baseline estimates
- Night pace adjustment applied to estimated durations
- Status values correctly reflecting Baseline vs Live
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
