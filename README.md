[README_CT_Cement_Plug.md](https://github.com/user-attachments/files/34159938/README_CT_Cement_Plug.md)
# Coiled Tubing Cement Plug Simulator (V1)

**File:** `CT_Cement_Plug_V1.html`
**Author:** S. Siddhu | Central FEU
**Version:** V1.3 · 2026-10-29
**Disclaimer:** For training and review purposes only — not a job design tool. Results come from simplified models and user inputs; verify against the service company's job design and field data before operational or regulatory use.
**Version note:** V1 rev. 2 replaces the two-view (full well + magnified inset) layout with a single scale-break column and adds the coil reel schematic.
**Type:** Single-file HTML / CSS / JavaScript (no external libraries, no network calls)
**Purpose:** Training and review tool for setting a balanced cement plug with coiled tubing (CT), such as an abandonment plug on a CIBP. The nozzle is kept a user-set distance (the cement head) below the top of cement while the CT is pulled. Fluid positions, CT depth, pull speed and pressures are all computed from pump rate and time.

> This is a teaching and review aid, not a job design tool. See *Assumptions & Limitations*.

---

## 1. How to Run

1. Open `CT_Cement_Plug_V1.html` in any current browser.
2. Enter inputs in the left panel; results update as you type.
3. Click **▶ RUN JOB** to play the whole job, or click a numbered step to play just that phase. **❚❚ PAUSE / ▶ RESUME** holds the clock. **↺ RESET** returns the CT to surface.
4. **Animation speed** sets how much job time passes per real second (×60 = 1 minute of job time per second).
5. **☀ Light view / ☾ Dark view** switches themes; the choice is remembered in that browser.

---

### Volume Units (header toggle)
**Volume units: bbl | ft³**, in the header bar.
- The model always calculates in barrels; the toggle only changes what is displayed and entered.
- Switching units converts the volume inputs in place (1 bbl = 5.614583 ft³).
- Results show the chosen unit first, with the other unit in brackets, for example `304.3 bbl (1,708.4 ft³)` or `1,708.4 ft³ (304.3 bbl)`. Capacities show bbl/ft and ft³/ft the same way.
- Gauges, warnings, status text and readouts use the chosen unit.
- Pump rate stays in bpm.
- The choice is remembered in that browser and shared between the Primary Cementing and CT tools.
- In the CT tool, the pump schedule increment and volume column also use the chosen unit, so setting ft³ with a 5 ft³ increment gives rows every 5 ft³. The matched-pull-speed note shows the rate in ft³/min when ft³ is selected.

---

## 2. Inputs

### Well & Plug
| Field | Units | Default | Notes |
|---|---|---|---|
| Casing preset / OD / weight | in, lb/ft | 7" 26.00 | ID = √(OD² − W/2.667) unless overridden |
| Casing ID override | in | blank | Use a published or measured ID |
| Plug base / CIBP | ft | 5,000 | Cement stacks up from here |
| Target plug top | ft | 4,500 | Used for required volume and pass/short check |
| Well fluid density | ppg | 9.0 | Fluid in the hole above the plug |

### Coiled Tubing
| Field | Units | Default | Notes |
|---|---|---|---|
| CT preset / OD / wall | in | 1-3/4" × 0.134" | CT ID = OD − 2 × wall |
| CT length on reel | ft | 12,000 | **Total** string length; all of it must be filled before fluid reaches the nozzle |
| Surface lines | bbl | 2.0 | Pump to reel inlet |
| Max pump pressure | psi | 5,000 | Warning limit; shown as a green mark on the pressure gauge |

### Fluids
| Field | Units | Default |
|---|---|---|
| Spacer ahead | bbl / ppg | 5 / 8.33 |
| Cement | bbl / ppg | 22 / 15.8 |
| Cement plastic viscosity / yield point | cP / lb/100 ft² | 60 / 20 |
| Spacer behind | bbl / ppg | 2 / 8.33 |
| Displacement | ppg | 8.6 |

The CT is assumed full of displacement fluid at the start.

### Operations
| Field | Units | Default | Notes |
|---|---|---|---|
| Pump rate | bpm | 1.5 | Constant while pumping |
| Cement head above nozzle | ft | 100 | Distance the nozzle stays below the cement top while laying the plug |
| RIH / POOH speed | ft/min | 100 | Trip speed |
| Pull-out-of-cement speed | ft/min | 30 | Pumps off, nozzle pulled clear of the plug |
| Pull above final TOC | ft | 200 | Where the pull-out step stops before POOH |

---

## 3. Job Sequence

| Step | What happens |
|---|---|
| 1. RIH | CT runs to the plug base at trip speed. Pumps off. |
| 2. Pump & build head | CT stationary at the plug base. Spacer and cement travel through surface lines and the full CT length on the reel. Spacer exits first, then cement fills up around the nozzle until the cement top is the set head above the nozzle. |
| 3. Lay plug | CT is pulled at the **matched speed** while pumping, so the head stays constant. Pumping stops when the last cement leaves the nozzle (spacer behind is then at the nozzle). |
| 4. Pull out of cement | Pumps off. Nozzle pulled up through the cement and clear of the plug. The cement top drops slightly as the CT volume leaves the cement. |
| 5. POOH | CT pulled to surface at trip speed. |

---

## 4. Calculation Method

### 4.1 Capacities
| Quantity | Formula |
|---|---|
| Wellbore capacity C_w (bbl/ft) | Casing ID² / 1029.4 |
| CT internal capacity C_ct | CT ID² / 1029.4 |
| CT displacement C_od | CT OD² / 1029.4 (steel plus contents) |
| Annulus around CT C_ann | C_w − C_od |
| Volume to nozzle | Surface lines + CT length × C_ct (constant: CT moving in or out of the hole does not change total string volume) |

### 4.2 Fluid tracking
Fluids are tracked by cumulative pumped volume, with plug flow (no mixing). A parcel that entered when the cumulative pumped volume was *u* is at path position *V − u*. Positions along the string are converted to:
- surface lines,
- CT on the reel (shown in the bar across the top of the schematic),
- CT in the hole, by depth.

Fluid that has left the nozzle stacks up from the CIBP. Wellbore capacity is C_w below the nozzle and C_ann above it.

### 4.3 Building the head (Step 2)
With the nozzle at the plug base, the cement needed to put the cement top *H* feet above the nozzle is:

**V₀ = H × C_ann**

Default: 100 ft × 0.03516 bbl/ft = **3.52 bbl**.

### 4.4 Matched pull speed (Step 3)
While the nozzle is submerged by a constant head *H*, the cement in the wellbore is:

V_cmt = (Z_base − Z_top) × C_w − H × C_od

The second term is constant, so pumping at rate *q* raises the cement top at q / C_w. To keep the head constant, the nozzle must rise at the same rate:

**CT pull speed = pump rate ÷ wellbore capacity** (ft/min = bpm ÷ bbl/ft)

Default: 1.50 bpm ÷ 0.03813 bbl/ft = **39.3 ft/min**. In the model, CT depth during this step is computed from this relationship, so the fill in the animation and the CT movement stay locked together. The head gauge holds at the set value (verified constant to 0.001 ft through the whole step), and the speed gauge shows the matched speed with a green reference mark.

**Practical note:** the matched speed depends on the actual rate. If the rate drops (pump problems or a pressure limit), the pull speed must drop with it, or the nozzle will be pulled out of the cement.

### 4.5 Plug top
| Quantity | Formula | Default |
|---|---|---|
| TOC at end of pumping (CT still in) | Z_base − (C + H × C_od) / C_w | 4,415 ft |
| Final TOC (CT out) | Z_base − C / C_w | 4,423 ft |
| TOC drop when CT exits | H × C_od / C_w | 7.8 ft |
| Required cement to target | (Z_base − target) × C_w | 19.1 bbl |
| Excess over required | (C ÷ required − 1) × 100 | +15.4% |

### 4.6 Pressures
| Quantity | Method |
|---|---|
| Annulus pressure at nozzle | 0.052 × Σ density × height of wellbore fluids above the nozzle |
| CT hydrostatic at nozzle | 0.052 × Σ density × height of fluids in the CT in the hole (reel portion is horizontal, no head) |
| CT friction | Bingham-plastic pipe flow over the full CT length (reel + hole) for each fluid segment; laminar and turbulent (Fanning) both evaluated and the larger used |
| CT pump pressure | max(0, annulus at nozzle − CT hydrostatic at nozzle + CT friction) while pumping |
| Free-fall | Flagged when annulus at nozzle exceeds CT hydrostatic plus friction |
| Pressure on CIBP | 0.052 × Σ density × height of the full wellbore column |
| Head ΔP | 0.052 × (cement − well fluid density) × H |

Default-case results: CT friction is about 1,519 psi with the string full of water-like fluid and 6,672 psi if it were full of cement. The peak pump pressure during the job is **4,828 psi**, while cement is in the CT.

**Friction caveat:** the friction values are estimates. They use a simple Bingham-plastic model with the PV/YP you enter, treat spacers and displacement as water (PV 1 cP, YP 0), and assume straight smooth pipe. Real CT friction is also affected by:
- reel curvature,
- friction reducers,
- slurry rheology at temperature.

Treat the pump-pressure gauge as indicative, and compare it against actual job data before relying on the numbers. Annulus friction is ignored; the annulus around CT in casing is large.

---

## 5. Display

- **Top strip:** volume to nozzle, cement to build head, matched pull speed, required cement, excess, final TOC, estimated job time. Hover for formulas.
- **Warning line:** head not achievable, head larger than plug, plug short of target, pressure limit exceeded, free-fall.
- **Coil reel:** a reel drum that unspools during RIH and spools during pulling.
  - The wrap diameter shrinks or grows with the footage on the reel, and the hub spokes turn with the footage paid out.
  - The wraps are colored by the fluid inside them. Inner wraps are the pump/swivel end and outer wraps are the end heading to the well, so you can watch spacer and cement travel through the reel before they reach the well.
  - The coil runs off the reel, over the gooseneck and into the injector.
  - The readout under the reel gives direction, speed, and footage on the reel and in the hole.
- **Wellbore (single view):** one continuous column with the CT, fluid layers, nozzle, cement-head bracket, target top (dashed green), forecast final TOC and CIBP.
  - When the plug is deep relative to its length, a **scale break** (zigzag) compresses the upper hole into the top quarter of the column. The plug zone below is expanded so the 100 ft head is easy to see.
  - Depths are labeled on each side of the break. Everything moves on one picture at one rate.
- **Gauges:**
  - CT pump pressure, with a mark at the max pressure limit.
  - Cement head above nozzle, with a mark at the set head.
  - CT speed, with a mark at the matched speed; direction is shown in the title.
  - Volume pumped.
- **Readout strip:** job time, nozzle depth, TOC, cement out, pressures at the nozzle, CT friction, CIBP pressure, state.

---

### Pump Schedule (right-hand panel)
A pump schedule generated from the same model as the animation. Every value comes from the same calculation that drives the gauges.

| Column | Meaning |
|---|---|
| Bbl | Cumulative volume pumped from the start of the spacer |
| Min | Pumping time at the set rate |
| Pump → Nozzle | Fluid entering the CT at the pump → fluid exiting at the nozzle (color-coded) |
| CT ft | Nozzle depth |
| ft/min | CT pull speed (0 while stationary; matched speed while laying the plug) |
| TOC | Cement top in the wellbore (— before cement exits) |
| Head | Cement height above the nozzle |
| psi | Surface pump pressure, including the CT friction estimate |

- **Increment (bbl):** the user sets the row spacing (for example 0.5, 1, 1.5, 2.5 bbl). The table is capped at about 1,500 rows.
- **Event rows** are highlighted:
  - start of each fluid at the pump;
  - spacer at nozzle and cement at nozzle;
  - head built (start pulling);
  - last cement out (pumps off);
  - pull out of cement;
  - POOH.
- **Current position:** a **▶** marker and highlight follow the job as it plays, and the table scrolls to keep the current row in view.
- **☰ Hide / Show schedule** (top left of the schematic) gives the drawing the full width.
- On screens narrower than 1,500 px the side panels tighten automatically, so the layout fits a 1366 × 768 display.

**Accuracy of gauge and schedule values:**
- **Calculated directly from geometry, volumes and densities (exact for the stated assumptions):** volumes, depths, pull speed, cement head, TOC and hydrostatic pressures.
- **Estimated:** pump pressure. It adds the CT friction estimate described in 4.6, so it is indicative.

### CT Tag & Buckling Check (left panel)
Checks two static tags (pumps off) in a vertical cased well:
- **CIBP / previous plug**, at the plug base depth.
- **New plug top**, at the final TOC of the plug just placed, for the tag after waiting on cement.

**Inputs:**

| Field | Default | Notes |
|---|---|---|
| Slack-off, CIBP / prev. plug | 3,000 lb | Weight-indicator drop when tagging |
| Slack-off, new plug top | 3,000 lb | Can differ from the CIBP tag |
| Friction factor, low / nominal / high | 0.15 / 0.25 / 0.35 | CT on casing in fluid. Must satisfy low ≤ nominal ≤ high. These are placeholder values; calibrate or take from the service company's tubing force analysis |
| CT yield strength | 80,000 psi | Grade-dependent; verify against string data |
| Design limit | 80% of yield | Stress limit for the check |

**Table output** (one column per tag), in two blocks: *Slack-off as entered* and *If all tubing weight is slacked off*.
- **Tag depth.**
- **ATW (all tubing weight):** buoyed hanging weight of the CT at the tag depth. This is the most that can be slacked off.
- **Slack-off.**
- **Force on plug** at nominal friction, the range from high to low friction, and the % of slack-off that reaches the plug.
- **Lockup ceiling** at nominal friction, with its range.
- **Buckled length and neutral point.**
- **Max stress:** % of yield, evaluated at low friction, which gives the highest force.
- **Status:** OK, friction-limited, slack-off greater than ATW, or over stress.

**ATW block:** force on the plug, friction range, % reaching the plug and max stress, assuming the operator sets down the full buoyed string weight. In the default case:
- Slacking off all 9,805 lb onto the CIBP puts about **2,782 lb** on the plug (2,355–3,568 lb across the friction range). Only 28% of the ATW reaches the plug, because the force is already close to the lockup ceiling.
- Stress reaches 26% of yield.
- On the chart, hollow red points on the red ATW lines mark this case.

**How the friction values are used:**

| Result | Friction used | Why |
|---|---|---|
| Buckling onset (sinusoidal / helical) | none | Onset depends only on stiffness, buoyed weight and clearance |
| Force on plug, lockup ceiling (main value) | nominal | Best estimate |
| Force on plug, lockup ceiling (range) | high → low | High friction gives the least force on the plug; low friction gives the most |
| Max stress | low | Most force reaches the plug, so this is the worst case for stress |
| Buckled length / neutral point | none | Set by slack-off ÷ buoyed weight |

Use the **high-friction** value when the question is whether enough force reaches the plug. Use the **low-friction** value when the question is whether the coil is overstressed.

**Default-case results** (1-3/4" × 0.134" CT in 7" 26# casing, 3,000 lb slack-off):

| | CIBP @ 5,000 ft | New plug top @ 4,423 ft |
|---|---|---|
| ATW (buoyed, 1.96 lb/ft) | 9,805 lb | 8,673 lb |
| Force on plug, μ 0.25 | 2,207 lb | 2,207 lb |
| Range, μ 0.35 → 0.15 | 2,014 – 2,456 lb | 2,014 – 2,456 lb |
| Lockup ceiling, μ 0.25 (range) | 2,787 lb (2,356 – 3,598) | same |
| Max stress at low μ | 18% of yield | 18% of yield |

**Force on the plug does not depend on depth** in a vertical well. The transfer curve depends only on CT size, casing ID, fluids and friction factor. Depth sets the ATW, which is the most that can be slacked off. The two tags differ only when their slack-offs differ or one slack-off exceeds the ATW at that depth.

**Method:**

| Quantity | Formula | Basis |
|---|---|---|
| Buoyed CT weight w | steel + internal fluid − displaced fluid (CT full of displacement, annulus well fluid) | |
| ATW at depth Z | w × Z | |
| Sinusoidal / helical onset (vertical) | 2.55 (EI w²)^⅓ / 5.55 (EI w²)^⅓ | Wu & Juvkam-Wold |
| Helical wall contact force | r F² / (4 EI) | Lubinski / Mitchell |
| Load transfer, buckled vertical string with friction | F_plug = F_L · tanh(ΔW / F_L) | He & Kyllingstad |
| Lockup ceiling | F_L = √(4 EI w / (μ r)) | Maximum force that can ever reach the plug |
| Buckled length | ΔW / w | |
| Max stress (outer fiber) | F / A_s + D r F / (4 I) | Axial + helical bending |

> **Verification status:** these formulas were written from model knowledge, not transcribed from the original papers. Verify each against the source (equation number and page) before using results in a formal review.

**Envelope chart:**
- The blue line is force on the plug at nominal friction. The shaded band covers high to low friction.
- The dashed grey line is the no-friction 1:1 case.
- The amber line is the lockup ceiling at nominal friction.
- The red dotted line is the stress limit.
- The red dashed vertical lines are the ATW at each tag depth.
- The circle (CIBP) and square (new plug) are the operating points, with bars showing the friction range.

**Limits of the check:**
- Vertical well only. No deviation, dogleg or tortuosity.
- Pumps off. Pumping pressure adds effective compression and is not included.
- The whole compressed section is treated as helical. That is conservative for delivery, because the short sinusoidal zone is ignored.
- The new-plug tag assumes the CT is again full of displacement fluid and the annulus is well fluid.
- Stripper/injector friction, residual bend, ovality and fatigue are not included.
- The friction factor is the controlling uncertainty. That is why the band is shown.

### End of CT (Coiled Tubing section)
A three-way selector. The choice is remembered in that browser.

| Option | Drawing | Hydraulics |
|---|---|---|
| **Open-ended** (default) | Square-cut open end, straight-down discharge | No end restriction |
| **Open + CT straightener** | Straightener body with rollers just above the open end | Same as open-ended. The straightener's length and OD are not included in volumes or friction |
| **Jetting nozzle** | Pointed nozzle with side jets | Adds a nozzle pressure drop to the pump pressure for whichever fluid is exiting |

**Nozzle pressure drop:** ΔP = ρ Q² / (12,032 C_d² A²), with ρ in ppg, Q in gpm, A = TFA in in², and C_d = 0.95. This is the standard bit-nozzle equation.
- Default TFA is 0.15 in². With 15.8 ppg cement at 1.5 bpm, ΔP ≈ **257 psi**, and peak pump pressure rises from 4,828 to about 4,963 psi.
- The value is shown under the selector and in the Pressures box.

Labels read "CT end" for the open-ended options and "nozzle" for the jetting nozzle. This applies to the depth labels, gauges, readout, schedule events and warnings.

### Scale Break Toggle
**⇕ Scale break: ON/OFF** (top left of the schematic):
- **ON:** compresses the upper hole and expands the plug zone, so the head and plug are easy to see.
- **OFF:** draws the whole well at one true vertical scale.

The choice is remembered in that browser.

### Training Scenario — Pulling Faster Than the Fill Rate
Demonstrates how a bad plug forms when the CT is pulled faster than cement fills the hole.

| Input | Default | Meaning |
|---|---|---|
| Training: pull speed | 80 ft/min | CT speed applied from the moment the button is clicked |
| Contamination | 1.0 bbl well fluid / bbl cement | Mixing assumed for cement strung out above the cement top; 0 = strung but unmixed |

**How to use:**
1. Run the job.
2. While pumping (step 2 or 3), click **⚠ TRAINING: Pull Faster Than Fill Rate**. The CT switches to the training speed at that moment and the animation plays to the end of pumping.
3. Click **✖ Clear training pull** to return to the normal job. **↺ RESET** also clears it.

**What is calculated exactly** (same geometry as the rest of the model):
- **Rate the head closes:** pull speed minus the rate the cement top rises. With the nozzle in cement, the top rises at (q − v·C_od) / C_ann, because withdrawing CT steel lowers the cement top.
- **When and where the nozzle leaves the cement**, and the volume pumped at that point. This is solved numerically. A red dashed line and the schedule row "Nozzle out of cement" mark it.
- **Volume of cement pumped after that point.** This is the strung cement.
- **Competent TOC** = plug base − (cement out before the nozzle left) ÷ wellbore capacity.

**What is illustrative:**
- Where strung cement ends up. Cement leaving the nozzle above the cement top falls through lighter well fluid; plug flow cannot predict the mixing.
- The model stacks it as a hatched "strung / contaminated" zone directly above the competent cement. Its height is strung volume × (1 + contamination ratio) ÷ wellbore capacity, and its density is the volume-weighted mix.
- It is **not credited** as plug.

**Default example** (training pull of 80 ft/min started at 41.7 bbl, matched speed 39.3 ft/min):
- The head closes at about 44 ft/min, so the 100 ft head is lost in 2.3 min.
- The nozzle leaves the cement at **4,673 ft** after **45.1 bbl** pumped.
- **9.5 bbl** of cement is strung.
- The competent TOC is **4,673 ft**, 173 ft short of the 4,500 ft target. The contaminated zone extends to 4,173 ft at 1:1 mixing.

**Displays during the scenario:**
- The head gauge turns red and reads "NOZZLE ABOVE CEMENT!".
- The speed gauge turns red.
- The status line reads "TRAINING — PULLING 80 FT/MIN > FILL RATE 39.3 FT/MIN · NOZZLE x FT ABOVE CEMENT — STRINGING".
- The pump schedule adds "TRAINING: pull started" and "Nozzle out of cement — stringing begins" rows.
- The warning line and the Placement box report the competent TOC against the target.

If the training speed is not faster than the matched speed, a warning says no stringing would occur.

## 6. Assumptions & Limitations

- Vertical well; plug set inside casing on a CIBP.
- Plug flow: no mixing, contamination, or slumping of cement into lighter well fluid.
- Cement assumed to stay on the CIBP. Density-driven swapping with lighter fluid below the nozzle is not modeled.
- Constant pump rate; no compressibility, pump efficiency or line stretch.
- CT starts full of displacement fluid; RIH is done with pumps off.
- Spacer behind is left in the CT when pumping stops. Under-displacement and pump-while-pulling-out practices are not modeled.
- Friction estimate only (see 4.6).
- CT presets are common OD/wall combinations. Verify against the CT string data sheet before use.
