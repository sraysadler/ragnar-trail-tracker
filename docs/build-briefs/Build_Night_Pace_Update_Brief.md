# Build Task: Night Pace Adjustment Logic Update

## Objective
Update the forecasting logic to apply the Night Adjustment as a pace
modifier rather than a duration modifier. This produces more accurate
estimates by treating night running as a pace penalty, and by normalizing
historical pace data so day and night loops can be compared fairly.

---

## Current Behavior (to be replaced)

Currently the night adjustment is applied to the estimated duration
after the fact:

```
Estimated Duration = Predicted Pace × Loop Miles × (1 + nightAdjustment/100)
```

This is being replaced with a pace-based approach.

---

## New Behavior

### Core principle
The Night Adjustment % represents how much slower a runner goes at
night compared to their natural day pace. All pace history is stored
and compared in terms of normalized day pace. When estimating a night
leg, the night penalty is applied to the predicted pace. When estimating
a day leg, pace history is used as-is after normalization.

### Step 1 — Normalize historical pace data

When collecting a runner's pace history for use in predictions, any
pace from a night leg must be normalized to its day-equivalent:

```javascript
function normalizePace(actualPace, isNightLeg, nightAdjustment) {
  if (isNightLeg) {
    return actualPace / (1 + nightAdjustment / 100);
  }
  return actualPace;
}
```

Example: Runner ran a night leg at 12:36/mi with 5% night adjustment.
Normalized day pace = 12:36 / 1.05 = 12:00/mi.

This normalized value is what gets stored in and used for pace history
averaging — not the raw night pace.

### Step 2 — Predict pace for an upcoming leg

After calculating the runner's normalized average pace from their
history, apply the night adjustment if the upcoming leg is a night leg:

```javascript
function getPredictedPace(normalizedAvgPace, isNightLeg, nightAdjustment) {
  if (isNightLeg) {
    return normalizedAvgPace * (1 + nightAdjustment / 100);
  }
  return normalizedAvgPace;
}
```

### Step 3 — Estimated duration

```
Estimated Duration = Predicted Pace × Loop Miles
```

No additional multiplier needed — the night adjustment is already
baked into the predicted pace.

---

## Updated calculateLegData() Flow

For each leg processed in order 1→24:

```
1. Determine if this leg is a night leg (startTime within night window)

2. If actual finish exists for this leg:
   a. Calculate actual pace = actualDuration / loopMiles
   b. Normalize the actual pace:
      normalizedPace = isNightLeg
        ? actualPace / (1 + nightAdjustment/100)
        : actualPace
   c. Add normalizedPace to runner's pace history
      (history always stores normalized/day-equivalent paces)

3. If no actual finish (need to estimate):
   a. Get runner's normalized pace history (array of day-equivalent paces)
   b. Calculate normalizedAvgPace:
      - 0 entries: use config avgPace (already a day pace)
      - 1+ entries: average of all normalized paces
   c. Apply night adjustment if this is a night leg:
      predictedPace = isNightLeg
        ? normalizedAvgPace * (1 + nightAdjustment/100)
        : normalizedAvgPace
   d. Estimated Duration = predictedPace × loopMiles
```

---

## Example Walkthrough

Config: avgPace = 12:00/mi, nightAdjustment = 5%, night window 8PM–7AM
Runner: Wendy — Legs 1 (Green, day), 9 (Red, night), 17 (Yellow, day)

**After Leg 1 actual entered (day loop, 50 min, 4 mi):**
- Actual pace = 50/4 = 12.50/mi
- Is night leg? No → normalized pace = 12.50/mi
- Wendy's pace history: [12.50]

**Estimating Leg 9 (Red, 6 mi, night loop):**
- Normalized avg pace = 12.50/mi
- Is night leg? Yes → predicted pace = 12.50 × 1.05 = 13.125/mi
- Estimated duration = 13.125 × 6 = 78.75 min ≈ 1:18:45

**After Leg 9 actual entered (night loop, 79 min, 6 mi):**
- Actual pace = 79/6 = 13.167/mi
- Is night leg? Yes → normalized pace = 13.167 / 1.05 = 12.54/mi
- Wendy's pace history: [12.50, 12.54]

**Estimating Leg 17 (Yellow, 5 mi, day loop):**
- Normalized avg pace = (12.50 + 12.54) / 2 = 12.52/mi
- Is night leg? No → predicted pace = 12.52/mi
- Estimated duration = 12.52 × 5 = 62.6 min ≈ 1:02:36

**If Leg 17 were a night loop instead:**
- Normalized avg pace = 12.52/mi
- Is night leg? Yes → predicted pace = 12.52 × 1.05 = 13.146/mi
- Estimated duration = 13.146 × 5 = 65.73 min ≈ 1:05:44

---

## Impact on Baseline Estimates

When a runner has no pace history, the config avgPace is used as the
normalized day pace baseline. Night adjustment still applies:

- Day leg baseline: avgPace × loopMiles
- Night leg baseline: avgPace × (1 + nightAdjustment/100) × loopMiles

This matches the previous behavior for baseline estimates, so the
change is only visible once actual pace data exists.

---

## Key Change Summary

| | Before | After |
|--|--------|-------|
| Where adjustment applied | To duration at end | To pace before duration calc |
| Pace history stored | Raw actual pace | Normalized day-equivalent pace |
| Night leg estimate | duration × 1.05 | pace × 1.05, then × miles |
| Day leg after night history | Used raw night pace in average | Normalizes night pace before averaging |
| Result for estimates | Slightly less accurate | More accurate — night/day history comparable |

---

## Verification Steps

Use config: avgPace = 12:00/mi, nightAdjustment = 5%,
night window 8:00 PM – 7:00 AM.
Runner order: Wendy, Kristine, Scott, Jeff, Jason, Ray, Jenny, Tim.
Start time: 9:00 AM.

**Test 1 — Baseline night estimate (no actual times entered)**
- Find a leg whose estimated start falls after 8:00 PM
- Open its Row Detail Modal
- Predicted pace should be 12:00 × 1.05 = 12.60/mi
- Est Duration should be 12.60 × loop miles

**Test 2 — After entering Leg 1 actual (day loop)**
Enter Leg 1 (Wendy, Green, 4 mi) actual = 9:50 AM
- Wendy actual pace = 50/4 = 12.50/mi (day pace)
- Open modal for Wendy's first night leg
- Predicted pace should be 12.50 × 1.05 = 13.125/mi ≈ 13:07/mi
- Est Duration = 13.125 × loop miles

**Test 3 — After entering a night leg actual**
Continue entering actuals until Wendy's first night leg is reached.
Enter her actual night finish time.
- Calculate her actual night pace (duration / miles)
- Her normalized pace = actual night pace / 1.05
- Open modal for Wendy's next day leg
- Predicted pace should be average of (12.50, normalized night pace)
- Should NOT include the raw night pace in the average

**Test 4 — Verify day leg after mixed history**
With both a day and night leg completed for one runner:
- Predicted pace for their next day leg should be close to their
  day pace (not inflated by the raw night pace)
- Predicted pace for their next night leg should be that same
  average plus 5%

---

## Key Constraints

- Pace history array always stores normalized (day-equivalent) paces
- Raw actual pace is never stored in history — always normalize first
- Night adjustment applied at prediction time, not storage time
- Config avgPace is treated as a day pace (no normalization needed)
- Single HTML file, no external dependencies

---

## Deliverable

Updated `ragnar-trail-tracker.html` with:
- Pace history storing normalized day-equivalent paces
- Night leg predictions using pace × (1 + adjustment)
- Day leg predictions using normalized average pace
- All verification steps passing

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
