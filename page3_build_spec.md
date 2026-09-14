# Page 3 Build Spec: The Gas / The Brakes

Force Report: CMJ Profile, Athlete Report. Page 3 of the multi-page CMJ report, following Page 1 (cover/hero metrics) and Page 2 (Metric History).

Ship this spec alongside two other files:
- `lead_cmj_norms.json`: the team average reference data (see section 4)
- `page3_force_velocity_mockup.html`: visual reference for layout and typography only, not a source of truth for colors or data values

## 1. Purpose

Two paired quadrant charts that bucket the athlete on force output vs. speed of movement, split into the two phases of the jump:

- **The Gas**: propulsive phase. Force-velocity dominance, is the athlete's output driven more by force, more by speed, both, or neither.
- **The Brakes**: braking phase. How hard and how fast the athlete decelerates and reverses out of the countermovement.

These replace the Force-vs-mRSI plan discussed earlier in this build. mRSI does not appear on this page.

## 2. Required CSV Fields

Confirmed present in Hawkin's standard CMJ export (verified against real athlete exports):

```
Avg. Relative Propulsive Force   (% bodyweight, NOT N/kg, despite the field name pattern)
Avg. Propulsive Velocity         (m/s)
Avg. Relative Braking Force      (% bodyweight)
Avg. Braking Velocity            (m/s, exported as a negative value)
```

No relative/normalized velocity field exists in the export, and none should be synthesized. Velocity doesn't need bodyweight normalization the way force does, since force scales with mass and velocity doesn't. Force uses the Relative (%BW) column: raw N absolute force is a worse comparison basis across a mixed-bodyweight roster.

If any of these four columns is missing from an uploaded CSV, fail gracefully: hide the affected chart and show a one-line "Braking data not available in this export" / "Propulsive data not available in this export" message rather than blocking the whole page.

## 3. Which Test Value to Plot

Use the same aggregation convention as Page 1 (Last-5 Avg), not a single most-recent trial, for consistency across the report and to reduce single-test noise. If Page 1's existing aggregation logic is already implemented, reuse that function against these four fields instead of writing new aggregation logic.

Filter to standard CMJ trials only before aggregating: Segment values starting with "Countermovement Jump:" and a numeric suffix. Exclude "Countermovement Jump-Arm Swing" and any combo segments (for example "Countermovement Jump-Plyo Push Up") from this page's calculations, they use a different movement pattern and will skew the force/velocity relationship if mixed in.

Where an athlete has multiple trials logged on the same test day, use the best trial of that day (highest Jump Height), then pull that same trial's force and velocity values, matching the "same-day jumps: best attempt used" convention already in place elsewhere in the report.

## 4. Team Average / Reference Group Logic: resolved

The "team average" crosshair on both charts comes from a maintained static reference file, not a live recalculation off whatever CSV happens to be loaded. This keeps a coach testing two or three athletes on a Tuesday from shifting the whole team's quadrant boundaries.

**Data source:** `lead_cmj_norms.json`, included alongside this spec. Sex-split (`female` / `male`), each with `mean`, `median`, `sd`, `min`, `max`, and `n` for all four Page 3 fields plus Jump Height, Peak Relative Propulsive Power, and mRSI. Use the `mean` value for crosshair placement on both charts, matched to the athlete's sex.

**Methodology baked into the file** (also present as the file's own `methodology` field): standard CMJ trials only, arm-swing and combo segments excluded; same-day trials reduced to the best attempt by Jump Height; each athlete's value is their average across up to their last 5 test days; the roster stat is computed across those per-athlete values.

**Refresh cadence:** this file should be regenerated periodically from a fresh Hawkin roster export, suggest monthly, or whenever a meaningful number of new tests have landed, rather than treated as permanent. Flag it in the codebase as a data file that goes stale, not a hardcoded constant.

**Edge case, athlete's sex isn't available or doesn't match either reference set:** don't fabricate a crosshair. Render the athlete's point with no average lines, and show: "Team average not available for this athlete's roster group."

## 5. Axis Ranges

Do not hardcode fixed axis ranges. Compute min/max per metric from the norms file's `min`/`max` fields (per sex), with roughly 10% padding on each end. A baseball roster and a basketball roster will not occupy the same force/velocity ranges, and a fixed scale from one sport will misrepresent the other.

## 6. Quadrant Boundaries

The crosshair (team average on both axes) is what divides the four quadrants, not the geometric center of the axis range. This matches Hawkin's own quadrant tool. If the norms file is refreshed, the quadrant boundaries move with it.

## 7. Chart Titles & Labels

```
Chart 1 title: The Gas
Chart 1 subtitle: Propulsive phase, Avg. Relative Propulsive Force vs. Avg. Propulsive Velocity
Chart 1 X axis: Avg. Relative Propulsive Force (% BW)
Chart 1 Y axis: Avg. Propulsive Velocity (m/s)
Chart 1 quadrant labels: Velocity-dominant (top-left) / Well-rounded (top-right) / Underdeveloped (bottom-left) / Force-dominant (bottom-right)

Chart 2 title: The Brakes
Chart 2 subtitle: Eccentric phase, Avg. Relative Braking Force vs. Avg. Braking Speed
Chart 2 X axis: Avg. Relative Braking Force (% BW)
Chart 2 Y axis: Avg. Braking Speed (m/s)
Chart 2 quadrant labels: Fast, low force (top-left) / Fast and forceful (top-right) / Slow and light (bottom-left) / Slow and forceful (bottom-right)
```

Note the sign flip on Chart 2: `Avg. Braking Velocity` is negative in the raw CSV (center of mass moving downward). Display it as an absolute value labeled "Braking Speed." Do not show the negative sign or the field's raw name on the parent-facing report.

## 8. Legend

Two markers, consistent across both charts:

- Athlete: solid dot
- Team average: horizontal dash, not a dot. Visually ties back to the dashed crosshair lines rather than introducing an unrelated shape.

## 9. Display Formatting

- Force: whole number, no decimal, for example `223% BW`
- Velocity / Speed: two decimals, for example `1.59 m/s`
- No em dashes anywhere in generated copy or labels
- All copy gender-neutral ("the athlete," "they/their")

## 10. Coach's Notes Copy (if-then, by quadrant)

Populate based on which quadrant the athlete's point falls into on each chart. These pair directly with the quadrant labels in section 7.

**The Gas**

```
Well-rounded:
No single quality is holding the jump back. The athlete is producing solid force and getting it into motion quickly, the engine and the throttle are both working. Programming stays balanced: continue strength-speed work across the spectrum rather than biasing toward one end.

Force-dominant:
The raw output is there, but it is not converting into speed. This is a strong but slow profile: an athlete who can move real weight but whose jump looks heavy. Programming should shift toward the velocity end of the strength-speed continuum: ballistic and speed-strength work, lighter loads moved with intent, contrast pairs where a heavy set is immediately followed by an unloaded jump.

Velocity-dominant:
The athlete is quick, but there is a ceiling on how quick they can stay without more raw force underneath it. This profile looks explosive right now but will plateau if the strength base does not catch up. Programming should lean into the hypertrophy-strength continuum: build tissue capacity and maximal strength first.

Underdeveloped:
Neither quality is where it needs to be yet. This is not really a deficiency in the diagnostic sense, it is usually a training-age story: a young or early-stage athlete who has not built a foundation in either direction. This is a GPP block: general strength, general work capacity, movement competency, before specializing.
```

**The Brakes**

```
Fast and forceful:
The athlete is absorbing the descent hard and fast, loading the tendon-muscle system aggressively and reversing direction with authority. This is the signature of a well-tuned stretch-shortening cycle: minimal energy leak on the way down. Programming can lean into reactive and plyometric work with confidence.

Slow and forceful:
The strength to brake hard is there, but the athlete is taking their time doing it: a longer, more controlled descent rather than a sharp catch and reverse. Worth cueing a faster, more aggressive transition at the bottom of the jump before assuming this needs a strength fix.

Fast, low force:
The athlete is moving through the brake quickly, but without much force behind it: less a controlled catch, more a fast fall. High eccentric velocity without the force to match it can mean the tissue is taking the deceleration passively rather than actively controlling it. Eccentric strength work, tempo lowering, controlled landings, deceleration-specific training, is the priority here.

Slow and light:
Neither strong nor fast through the brake, the athlete is not loading the eccentric phase aggressively at all. A longer, softer brake like this can mean energy is leaking before the concentric drive even starts. Eccentric-emphasis strength work and reactive landing drills are the starting point.
```

## 11. Styling

Reuse existing app CSS variables/theme tokens (including Lumin Mode dark-theme handling) rather than hardcoding new colors. Page 3 should inherit whatever mode the rest of the report is in. `page3_force_velocity_mockup.html` is a visual reference for layout and typography only, not a source of truth for color values, and its "Sample Athlete" data point is illustrative, not a real roster member.

## 12. Open Decision to Confirm Before Building

Whether the "Fast, low force" braking language should ship as-is or be softened pending a season of real outcome data. It's flagged in the design discussion as the least-validated quadrant of the eight, a reasonable injury-risk hypothesis rather than an established one.
