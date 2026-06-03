# hoistdrivesizer

A single-file tool that sizes the **motor, gearbox, holding brake and inverter**
for a **hoist / winch** drive using the serial-hoist (Siemens SINAMICS) method,
and checks the design withstands the **125% proof load** and meets the **brake
and gearbox service factors**.

## Files

- **`hoist-drive-sizer.html`** — interactive React app. Open directly in a
  browser; no build, no server (React 18 + Babel + Tailwind from CDN).

## What it does

From the load, geometry, motion and motor/gearbox/brake data it returns:

- **Power & torque** — static, dynamic and total hoisting power; regen power on
  lowering; drum torque; static / dynamic / **peak** / **RMS** motor torque.
- **Gearbox** — optimum (inertia-matched) ratio `i_opt`, motor speed at the
  hoisting speed, **service factor `fB`** and rated-output-torque check.
- **Motor** — recommended power, **stall-torque margin**, thermal utilisation
  (RMS) and duty %ED, each with PASS/FAIL.
- **Holding brake & 125% proof load** — recommended brake torque as the worst
  case of *brake factor × motor rated* (Siemens serial-hoist ≈ 2.0), *holds the
  proof load* (≈ 1.25 × static load torque) and *exceeds the static load torque*,
  with PASS/FAIL on each.
- **Overload / proof** — rated, operational-overload (110%) and proof (125%) loads.
- **87 Hz technique (delta)** — optional. Extends the constant-torque speed range
  and continuous power by `k = V_supply/V_delta` (≈ √3) — **torque is unchanged,
  only speed & power increase**. Reports the 87 Hz base frequency, extended base
  speed, power capability (× k), and the **required inverter current (√3 × rated)**.
  Lets a smaller motor frame meet the duty (at the cost of a larger drive); warns
  if the operating point goes into field weakening.

## It will not undersize

Every recommendation errs to the safe side:

- **Motor power** is the **governing maximum of the duty and thermal (RMS)
  demands**, snapped **up** to the next IEC standard frame. The RMS includes the
  acceleration peak, so high-cycle / high-accel duties are covered.
- **Brake torque** is the **worst case** of the three requirements above.
- **Stall-margin** and **service-factor** checks guard the peak-torque and
  gearbox cases. Recommendations round up, not down.

## Engineering basis

Serial-hoist / Siemens SINAMICS sizing equations, implemented verbatim.
`g = 9.81 m/s²`. The load in the formulas is the **total** `mL + mH`
(payload + hook/block); the counterweight `mG` subtracts statically and adds
dynamically. Physics is computed; the **factors** (brake 2.0, proof 1.25,
overload 1.10, service factor, %ED, stall 1.3) are **editable inputs** with
typical defaults — confirm against **EN 14492-2 / EN 13001 / EN 13135**.

> **Scope:** a sizing & first-pass verification aid. It does not classify duty
> (use the [Hoist Duty Calculator](../hoistdutycalculator/)), check rope/drum
> strength, or assert full standards compliance. For multi-mechanism sizing
> (hoist + long/cross travel, anti-skid, wheel/rail) see
> [`cranedrivesizer/`](../cranedrivesizer/). Verify the final motor / gearbox /
> brake / inverter against the supplier's selection data before order.

## Usage

Open `hoist-drive-sizer.html` in a browser. Fill the project stamp and use
**Print / PDF** for a record copy.
