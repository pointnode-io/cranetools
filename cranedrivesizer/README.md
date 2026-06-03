# cranedrivesizer

A single-file tool that **sizes the drive** for the three crane mechanisms —
**hoist**, **long-travel** and **cross-travel** — from the physical duty.

## Files

- **`crane-drive-sizer.html`** — interactive React app. Open directly in a
  browser; no build, no server (React 18 + Babel + Tailwind from CDN).

## What it does

Pick a mechanism tab and enter the duty. For each, it returns:

| Output | Hoist | Travel (LT / CT) |
|---|---|---|
| Required motor power (steady + peak) | ✓ | ✓ |
| Motor torque (steady + acceleration) | ✓ | ✓ |
| Gearbox ratio `i = n_motor / n_out` | ✓ | ✓ |
| Minimum brake torque | ✓ | ✓ |
| **Next standard motor** (IEC ladder) + breakdown-torque margin | ✓ | ✓ |
| Drives-per-mechanism load sharing | ✓ | ✓ |
| Drum speed & lead-line rope pull | ✓ | — |
| **Reeving** efficiency (derived from sheaves) + ratio | ✓ | — |
| **Rope safety** `MBL/S ≥ Z_p` and **drum D/d ≥ min** PASS/FAIL | ✓ | — |
| Hoisting **dynamic factor φ₂** (EN 13001-2) | ✓ | — |
| Optional **counterweight** (with overhauling/brake handling) | ✓ | — |
| Resisting forces (rolling / gradient / wind) | — | ✓ |
| **Anti-skid (drive) check** — tractive demand vs wheel/rail adhesion | — | ✓ |
| **VFD acceleration** — achievable accel = min(skid, torque) limited | ✓ | ✓ |
| Regen energy/power (decel & lowering → brake-resistor sizing) | ✓ | ✓ |
| Required deceleration & stopping distance | — | ✓ |
| **Indicative duty/energy** — RMS power, %ED, kWh & running cost | ✓ | ✓ |

The **anti-skid check** is the headline travel result: it confirms the driven
wheels can transmit the acceleration force without slipping (and flags if the
brake would lock the wheels). Inputs are grouped into collapsible sections so the
core sizing stays front-and-centre and the advanced options (reeving/rope, VFD,
duty cycle) expand on demand.

> **Motor pick is acceleration-aware.** Travel motors are sized on the *greater*
> of the continuous demand and the peak-torque demand ÷ usable overload — sizing
> on steady running alone would pick a frame too small to accelerate the mass.

## How it fits the toolkit

This is the complement to the [Hoist Duty Calculator](../hoistdutycalculator/),
which classifies the mechanism duty group but stops short of sizing the motor
(it names **FEM 9.683** as the source). This tool picks up from there.

## Engineering basis

Two kinds of value, clearly separated in the app:

- **Kinematics & dynamics** — exact, edition-independent mechanics
  (`P = F·v/η`, `T = 9549·P/n`, `i = n_motor/n_out`, reflected inertia,
  `F_adh = μ·N`). These are not tunable.
- **Empirical coefficients** — adhesion μ, rolling-resistance terms, in-service
  wind, brake safety factor — are typical **defaults exposed as editable
  inputs**, badged *“default — confirm”*. The governing figures depend on the
  project, the wheel/rail or guide, and the controlling standard
  (FEM 9.683 / EN 13135 / EN 13001-2).

> **Scope:** this returns the *demanded* power, torque, ratio and *minimum*
> brake torque — it does **not** pick a catalogue motor. Thermal selection
> (duty %ED + permissible starts/hour, FEM 9.683) is a separate check that can
> govern. Verify every flagged coefficient and the final selection against the
> controlling standard before order.

## Usage

Open `crane-drive-sizer.html` in a browser. Fill the project stamp and use
**Print / PDF** for a record copy.
