# Build Task: Step 1 — HTML/CSS Dark Mode Layout and Touch Interface

## Objective
Create the structural skeleton of the app as a single self-contained HTML file.
This step is layout and styling only — no JavaScript logic, no real data, no
functionality. The goal is to get the visual shell looking correct so it can
be evaluated before any logic is added.

---

## File
Create: `ragnar-trail-tracker.html` in the root of the repository.

All CSS goes inside a `<style>` block in the `<head>`. No separate stylesheet.
No external CSS frameworks or CDN links. Everything self-contained.

---

## Four Zones (Portrait Layout)

The screen is divided into four zones. Three are fixed (do not scroll).
One scrolls.

### Zone 1 — Header (fixed, top)
- Left side: Team name (large, white, bold) and event name (smaller, muted)
- Right side: "Current time:" label (small, muted) above the time value
  (slightly larger, light weight, not bold)
- Background: `#1a1a1a`
- Bottom border: `1px solid #333`

### Zone 2 — Status Bar (fixed, below header)
Two rows:
- Row 1 (full width): Projected team finish — green highlighted bar
  - Background: `#1a2e1a`, border-bottom: `1px solid #2d5a2d`
  - Left: "Projected team finish" label in small uppercase green (`#5a9e5a`)
  - Right: Time value in larger green (`#7dcc7d`, 500 weight)
- Row 2: Three equal columns — Current loop | Next runner | Next start
  - Background: `#0d0d0d`, border-bottom: `1px solid #333`
  - Each column: small muted label (`#555`) above a slightly larger value (`#ddd`)

### Zone 3 — Race Table (scrollable)
- Column headers are fixed at the top of the scroll area — they do not scroll
- Header row background: `#1a1a1a`, border-bottom: `1px solid #444`
- 24 data rows, one per leg
- Each row has a bottom border: `1px solid #1e1e1e`
- Rows are tappable (add enough row height for comfortable finger tapping —
  minimum 44px)

**Six visible columns:**
| Column | Width | Notes |
|--------|-------|-------|
| # | Narrow/fixed | Leg number |
| Runner | Medium/fixed | Runner name |
| Lp | Narrow/fixed | Loop color initial (G/Y/R) |
| Start | Equal share of remaining width | Time |
| Est fin | Equal share of remaining width | Time |
| Actual | Equal share of remaining width | Time — tappable target |

The three time columns (Start, Est fin, Actual) share remaining space equally.

**Row colors by loop:**
- Green legs: `#003300`
- Yellow legs: `#333300`
- Red legs: `#330000`

**Time display styles:**
- Actual confirmed times: bold, white (`#fff`)
- Estimated times: italic, `#999`
- Unstarted Actual cells: dashed border, italic, `#555` — indicates tappable

**Active leg** (currently running): subtle highlight — `outline: 1px solid #555`

### Zone 4 — Footer (fixed, bottom)
- Background: `#1a1a1a`, border-top: `1px solid #333`
- Three items across: Export button | "Last saved: [time]" | Settings button
- Buttons: small, bordered, muted — consistent style

---

## Placeholder Data for Layout Evaluation

Use the Party Pace 2025 race data below so the table renders at full 24-leg
length and the row colors, time styles, and layout can be properly evaluated.

Show the race mid-progress: Legs 1–5 have actual finish times (bold white),
Leg 6 is the active leg (no actual finish yet, dashed tap target), Legs 7–24
are estimated (italic, #999).

| Leg | Runner | Loop | Start | Est Fin | Actual |
|-----|--------|------|-------|---------|--------|
| 1 | Wendy | G | 9:00 AM | 9:48 AM | 9:50 AM |
| 2 | Kristine | Y | 9:50 AM | 10:50 AM | 11:08 AM |
| 3 | Scott | R | 11:08 AM | 12:20 PM | 12:20 PM |
| 4 | Jeff | G | 12:20 PM | 1:08 PM | 1:13 PM |
| 5 | Jason | Y | 1:13 PM | 2:13 PM | 2:21 PM |
| 6 | Ray | R | 2:21 PM | 3:35 PM | — |
| 7 | Jenny | G | 3:35 PM | 4:21 PM | — |
| 8 | Tim | Y | 4:21 PM | 5:21 PM | — |
| 9 | Wendy | R | 5:02 PM | 6:14 PM | — |
| 10 | Kristine | G | 6:10 PM | 6:58 PM | — |
| 11 | Scott | Y | 6:55 PM | 7:55 PM | — |
| 12 | Jeff | R | 7:53 PM | 9:05 PM | — |
| 13 | Jason | G | 9:06 PM | 9:56 PM | — |
| 14 | Ray | Y | 9:56 PM | 10:59 PM | — |
| 15 | Jenny | R | 11:11 PM | 12:27 AM | — |
| 16 | Tim | G | 12:26 AM | 1:16 AM | — |
| 17 | Wendy | Y | 1:06 AM | 2:09 AM | — |
| 18 | Kristine | R | 2:20 AM | 3:36 AM | — |
| 19 | Scott | G | 3:33 AM | 4:23 AM | — |
| 20 | Jeff | Y | 4:20 AM | 5:23 AM | — |
| 21 | Jason | R | 5:25 AM | 6:41 AM | — |
| 22 | Ray | G | 6:55 AM | 7:45 AM | — |
| 23 | Jenny | Y | 7:49 AM | 8:49 AM | — |
| 24 | Tim | R | 8:56 AM | 10:08 AM | — |

Status bar placeholder values:
- Projected team finish: 10:08 AM
- Current loop: 6 – Ray
- Next runner: Jenny
- Next start: 3:35 PM
- Current time: 2:24 PM

---

## Key Constraints (from Do Not Do list)

- Single HTML file only — no separate CSS or JS files
- No CSS frameworks (no Bootstrap, Tailwind, etc.)
- No external CDN links of any kind
- No JavaScript in this step — layout and styling only
- Do not use `position: fixed` in a way that breaks scroll behavior —
  use a flexbox or grid outer layout to achieve fixed header/footer
  with a scrollable middle zone

---

## Deliverable

A single `ragnar-trail-tracker.html` file that:
- Renders the four-zone layout correctly in Chrome on a ~1280x800 portrait screen
- Shows all 24 legs with correct row colors
- Distinguishes visually between actual (bold white), estimated (italic #999),
  and unstarted (dashed tap target) time cells
- Has frozen column headers that stay visible while the table scrolls
- Looks correct without any JavaScript running

Commit to the main branch of ragnar-trail-tracker on GitHub when complete.
