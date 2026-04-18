# Build Task: Step 17 — Visual Design Refinement

## Objective
Three targeted visual adjustments to the main view. No functional changes —
layout and styling only.

---

## Change 1 — Footer: Green dot position

The green backup indicator dot should sit immediately to the left of the
"Last saved:" text with a small gap — not floating separately in the center.

**Current behavior:** Dot and "Last saved" text appear as separate elements
with too much space between them.

**Required behavior:** Dot is immediately adjacent to the left of "Last saved:",
reading as a single unit: `● Last saved: 7:19 PM`

The dot and text should be in the same container, aligned center vertically,
with approximately 4–6px gap between the dot and the text. The whole unit
should be centered in the footer between the Export and Settings buttons.

---

## Change 2 — Column headers: improved readability

The column header row (#, Runner, LP, Start, Est Fin, Actual) needs slightly
more visual weight for tablet readability.

**Current:** Small, all-caps, low contrast
**Required:**
- Increase font size slightly — try 10px (up from current ~8px)
- Increase the header row height slightly — try 36px (up from current ~28px)
- Keep the existing `#1a1a1a` background and `#777` color
- Keep all-caps styling — just bigger and with more vertical breathing room

---

## Change 3 — Active leg outline: better contrast on yellow rows

The current `2px solid #888` outline on the active leg is hard to see
against the yellow row (`#444400` background).

**Required:** Change the active leg outline to `2px solid #fff` (white).
This gives clear contrast against all three loop colors — green, yellow,
and red — without being distracting on completed rows.

---

## Verification

1. Footer: dot sits immediately left of "Last saved:" text, small gap,
   visually reads as one unit. Centered between Export and Settings.
2. Column headers: slightly larger and taller, still readable, not
   overwhelming the table rows.
3. Active leg: white outline clearly visible on yellow row. Also check
   it looks acceptable on green and red rows.

---

## Key Constraints

- No functional changes — styling only
- Single HTML file
- Do not change any colors other than the active leg outline

---

## Deliverable

Updated `ragnar-trail-tracker.html` with all three visual adjustments.
Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
