# cranedrivesizer

**One tool — pick the motion** (Hoist / Long-travel / Cross-travel) and size the
**motor, gearbox and brake** for a crane drive. Single self-contained HTML file.

## Files

- **`crane-drive-sizer.html`** — interactive React app. Open directly in a
  browser; no build, no server (React 18 + Babel + Tailwind from CDN).

## Tabs

### Hoist (deep serial-hoist sizer)
Static / dynamic / peak / **RMS** torque & power, drum torque, regen on lowering;
**speed-match** gear ratio (recommended) and inertia-match `i_opt`; motor **stall
margin** and **thermal utilisation**; **S1-continuous or %ED** duty; the holding
**brake** (factor on rated motor torque, **125% proof-load** hold and static-load
checks); **VFD** acceleration and the **87 Hz technique** (delta) with its
suitability checks; and the geared-motor / gearbox **service-factor** outputs.

### Long-travel / Cross-travel
Running resistance (bearing+rolling or specific), the **anti-skid drive check**
(tractive demand vs wheel/rail adhesion, with achievable accel = min of skid- and
torque-limited), VFD acceleration with **regen**, braking / stopping distance, and
a geared-motor spec (n₂, ratio, output torque, fₛ, wheel radial load).

## Key behaviours

- **Won't undersize, won't pad.** Required power is the true demand (governing
  larger of duty and thermal-RMS); the recommendation is the **smallest IEC frame
  that meets it**, with the entered motor checked PASS/FAIL against the requirement.
- **Multiple drives → per drive.** With N drives the load is split equally;
  motor / gearbox / brake figures are shown **per drive** with the **combined
  (× N)** total. A banner states this on each tab.
- **Hoist brake** = holding-brake factor (default 2.0 × rated motor torque) and
  must also pass the 125% proof-load and static-load checks (worst case wins).
  **Travel brake** = sized for deceleration / stopping distance and must not
  exceed adhesion (no holding factor).

## Engineering basis

Exact mechanics are computed; **coefficients** (adhesion, rolling resistance,
wind, brake / proof / overload / service factors, %ED, 87 Hz `k`) are **editable
"confirm" defaults** — published figures differ by standard/edition. `g = 9.81
m/s²`. Mechanism **duty group** → use the [Hoist Duty Calculator](../hoistdutycalculator/).

> **Scope:** a sizing & first-pass verification aid, not a catalogue selector or
> compliance certificate. Verify the final motor / gearbox / brake / inverter
> against the supplier's data and the controlling standard
> (EN 14492-2 / EN 13001 / EN 13135) before order.

## Usage

Open `crane-drive-sizer.html` in a browser. Fill the project stamp and use
**Print / PDF** for a record copy.
