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
| Drum speed & lead-line rope pull | ✓ | — |
| Resisting forces (rolling / gradient / wind) | — | ✓ |
| **Anti-skid (drive) check** — tractive demand vs wheel/rail adhesion | — | ✓ |
| Required deceleration & stopping distance | — | ✓ |

The **anti-skid check** is the headline travel result: it confirms the driven
wheels can transmit the acceleration force without slipping (and flags if the
brake would lock the wheels).

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
