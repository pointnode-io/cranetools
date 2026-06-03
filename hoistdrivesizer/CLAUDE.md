# CLAUDE.md — hoistdrivesizer

Tool-specific guidance. Read this **and** the repo-root `CLAUDE.md`.

## What this tool is

A single-file HTML/React app (`hoist-drive-sizer.html`) that sizes the **motor,
gearbox, holding brake and inverter** for a **hoist / winch** drive using the
serial-hoist (Siemens SINAMICS "Serienhebezeuge") method, and verifies the
design withstands the **125% proof load** and meets the **brake & gearbox
service factors**.

It is a focused *sizing* tool — not a full compliance suite. Mechanism **duty
classification** (ISO 4301-1 / FEM) lives in `hoistdutycalculator/`; broader
multi-mechanism sizing (long/cross travel + hoist, anti-skid, wheel/rail) lives
in `cranedrivesizer/`. This tool is the deep hoist/winch drive-train sizer.

No build step. React 18 + Babel-standalone + Tailwind from CDN. House style
matches the siblings (`ACCENT`/`STEEL`, `Field`/`Num`/`Badge`/`Row`/`Group`,
project stamp + Print/PDF).

## Engineering — verbatim Siemens serial-hoist equations

The numbered relations are implemented **as specified by the user** (Siemens
method); do not "improve" them. `g = 9.81 m/s²` (as specified — note this differs
from the 9.80665 used in `cranedrivesizer`; keep 9.81 here to match the source).

Key relations (exact mechanics, amber-badged):
- Static / dynamic power: `P = (m∓mG)·g·v / η`
- Static / dynamic drum torque: `M = (m∓mG)·g·d / (η·2·s)` (+ the `a` form)
- Drum speed `n = s·v/(π·d)`; motor speed `n·i_gear`
- Motor torque `M = (m∓mG)·g·d / (η_mech·η_gear·2·s·i_gear)` + `J_motor·α`,
  `α = 2π·n_motor[rev/s]/t_acc`
- Optimum (inertia-matched) ratio `i_opt = √[(J_load/s² + J_drum)/J_motor]`,
  `J_load = (m+mG)·(d/2)²`
- RMS motor torque over hoist/lower/rest (standstill cooling `kc`)
- Service factor `fB = M_gearbox,rated / (i_gear·M_motor,rated)`

**Masses:** the load in the formulas is the **total** `m = mL + mH` (payload +
hook/block). Counterweight `mG` **subtracts statically** (`m − mG`, it helps
hold) and **adds dynamically** (`m + mG`, it must be accelerated). This is the
deliberate reading of the source formulas (which wrote only `mL`); `mH` was an
otherwise-unused input, so it is folded into the load. Flagged in the UI.

## What it checks (PASS/FAIL)

- **Motor:** the **recommended power is the governing maximum of the duty basis**
  (vector → static / V/f → total, ÷ η_gear) **and the thermal RMS basis**, then
  **snapped UP to the next IEC frame** (`pickMotorUp`). The RMS deliberately
  **includes the acceleration peak** for the first `t_acc` of the hoist phase so
  it is not understated. Also: **stall margin** `M_K/M_peak ≥ 1.3` (editable),
  thermal utilisation `M_rms/M_rated ≤ 100%`, duty `%ED ≤ rated %ED`.
  **Do not regress the recommendation to static-power-only — that undersizes
  high-cycle / high-acceleration duties.** Every recommendation rounds up.
- **Gearbox:** service factor `fB ≥ fB_min`; rated output torque ≥ peak drum torque.
- **Brake (the emphasis):** the recommended brake torque is the worst case of
  **brake factor × motor rated** (Siemens serial-hoist ≈ **2.0×**), **proof-load
  hold** (`proofFactor × static load torque`, default **1.25×**), and **exceeds
  the static load torque**. Brake torque is referred to the motor shaft and the
  load torque is computed **lossless / gross** (gearbox friction only helps hold;
  counterweight not credited) — the conservative side. Fail-safe (spring-applied)
  is stated as a requirement in the note.
- **Overload / proof:** rated, operational overload (default 110%) and proof
  (default 125%) loads reported.

## Authority of values

- Physics/relations: exact, computed.
- **Factors are editable inputs** with typical defaults — `brakeFactor 2.0`,
  `proofFactor 1.25`, `overloadFactor 1.10`, `fB_min 1.0`, `stallFactor 1.3`,
  `motor_ED 40`, `kCool 0.33`. Published figures differ by standard/edition
  (EN 14492-2 / EN 13001 / EN 13135), so they are confirmed by the user, not
  asserted as normative.

## Scope / non-goals

A sizing & first-pass verification aid. It does **not** classify duty (use the
Hoist Duty Calculator), check rope/drum strength, or assert standards compliance.
Verify the final motor / gearbox / brake / inverter against the supplier's
selection data and the controlling standard before order / CE marking.

## History

Built from a user spec of the Siemens serial-hoist equations, then deliberately
**scoped back** from a broader EN 14492/EN 13001 compliance suite to a focused
motor/gearbox/brake/inverter sizer that nonetheless covers the 125% proof load
and the brake/service-factor checks. The wider compliance items (safety-device
checklist, drum-shell stress, duty classification, standards matrix) were left
out of the core by request — they can return as a separate compliance tool if
needed.
