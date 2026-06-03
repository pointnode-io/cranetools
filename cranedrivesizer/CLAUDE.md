# CLAUDE.md — cranedrivesizer

Tool-specific guidance. Read this **and** the repo-root `CLAUDE.md`.

## What this tool is

A single-file HTML/React app (`crane-drive-sizer.html`): **one tool, pick the
motion** (tabs **Hoist · Long-travel · Cross-travel**) and size the **motor,
gearbox and brake**. Shared project stamp, speed-unit toggle (m/min ↔ m/s),
Print/PDF and reference section across all three tabs.

This is the consolidated drive sizer — the former standalone `hoistdrivesizer`
was merged in as the **Hoist** tab. It complements `hoistdutycalculator/`
(mechanism duty *group*), which it does not replace.

No build step. React 18 + Babel-standalone + Tailwind from CDN. House style:
`ACCENT`/`STEEL`, `Badge`/`Field`/`Num`/`Row`/`Pill`/`Toggle`/`Group`/`SpeedField`.

`g = 9.81 m/s²` (single constant; the hoist serial-hoist method uses 9.81 — the
travel difference vs 9.80665 is negligible).

## Architecture

- One `App` with tabs `hoist` / `lt` / `ct`; each has its own `useState` object.
- **Long-travel and cross-travel share `TravelTab` + `computeTravel(t)`** — same
  engine, different defaults/labels/mass. Do not fork it.
- **Hoist uses `HoistTab` + `computeHoist(h)`** (the deep serial-hoist model).
- Pure compute functions return every intermediate; components only render.

## PER-DRIVE (important)

`nDrives` on each tab. When > 1, the load is split **equally** and the
motor/gearbox/brake figures are reported **PER DRIVE**, with the **combined
(× N)** total shown alongside. The `DriveScope` banner states this on each tab.
Brake `req_2x = brakeFactor × per-motor rated torque`; the proof/static brake
requirements use the per-drive static load torque. Keep the per-drive split.

## Motor pick — won't undersize, won't pad

- **Hoist:** required power = **governing max of duty (static or, in V/f, total)
  and thermal RMS** (RMS includes the accel peak for the first `t_acc` — not
  understated), per drive, then `pickMotor` rounds **up** to the next IEC frame
  (`÷k` when 87 Hz). No safety margin is added beyond the true demand; the
  utilisation/PASS-FAIL tests the entered motor against the requirement.
- **Travel is acceleration-governed:** size on the *greater* of continuous demand
  and `peak_torque / usable_overload` (`peakFactor = vfd ? vfdOverload : 1.6`) —
  steady-power-only picks a frame too small to accelerate. Do not regress this.

## Hoist specifics (serial-hoist method)

- Load `m = mL + mH` (payload + hook/block). Counterweight `mG` **subtracts
  statically** (`m − mG`) and **adds dynamically** (`m + mG`).
- Drum torque `= (m∓mG)·g·d / (η·2·s)`; motor torque divides by `i_gear` and `η`.
- **Gear-ratio guidance:** the recommended ratio is the **speed-match**
  `i_speed = (base speed) / (drum rpm)` (use the extended 87 Hz base speed when
  enabled) — pick this so the motor sits at base speed at the hoisting speed.
  `i_opt = √[(J_load/s² + J_drum)/J_motor]` (inertia match) is **secondary**.
- **Duty type:** `S1` continuous → `%ED` is N/A (always passes), thermal-RMS
  governs; `ED` intermittent → checks computed `%ED ≤ rated %ED`. (SEW geared
  motors are typically S1 — use S1.)
- **Stall margin** `M_K / M_peak ≥ stallFactor` (1.3); **thermal util** `M_rms /
  M_rated ≤ 100%`.
- **87 Hz technique (delta):** `k = V_supply/V_delta` (≈√3); extends base speed
  and continuous power **× k** at **constant torque** (torque NEVER scaled by k);
  inverter must carry `√3 × I_star`. Suitability checks (speed, inverter current)
  + declared (dual-voltage, cooling).

## Brake

- **Hoist holding brake:** recommended torque = **worst case** of `brakeFactor ×
  rated motor torque` (default **2.0**), the **125% proof-load** hold
  (`proofFactor × static load torque`), and **exceeds the static load torque** —
  each shown PASS/FAIL against the entered brake. Static load torque is gross,
  **lossless** (gearbox friction only helps hold), per drive; counterweight not
  credited. Must be **fail-safe** (spring-applied, released by power, applied on
  power loss) — stated in the note.
  - Background factors (verify against the controlling standard; editable):
    EN 14492-2 ≥ 1.5× on the *load* torque; FEM 1.001 / DIN 15435 ≈ 1.6–1.8×;
    hot/molten metal ≈ 1.75×; lifting persons ≈ 2.5×. The tool's default basis is
    a factor (2.0) on the *rated motor torque* combined with the proof/static
    floors — keep these as editable inputs, do not hard-assert.
- **Travel brake is different — no holding factor.** Sized for **deceleration /
  stopping distance** and must **not exceed adhesion** (else it skids); handled by
  the travel braking + anti-skid checks, not a `×` factor.

## Engineering — the authority split (preserve)

1. **Exact mechanics** (computed, amber-badged): `P=F·v/η`, `P=m·g·v/η`,
   `T=9549.3·P/n=P/ω`, `i=n_motor/n_out`, `J_eq=m·(v/ω)²`, `F_adh=μ·N`, the
   serial-hoist drum-torque forms, the 87 Hz `k` factor.
2. **Empirical / tabulated coefficients** — `DEF` object + per-tab inputs, all
   **editable "confirm" defaults** (blue badge): adhesion `μ=0.15`, rolling
   `f=0.5`, bearing `μ=0.02`, flange `c=1.5`, wind `p=250`, mech `η=0.90`, VFD
   overload `1.5`, breakdown `2.7`, service factor `fs`, brake `2.0`, proof `1.25`,
   overload `1.10`, stall `1.3`, %ED. Published figures differ by
   standard/edition — the tool gives the **formula** and you enter the governing
   value. Never present a default-based result as normative.

### Running resistance (travel)
`F_roll = (W/D_wheel)·c_flange·(μ_bearing·d_journal + 2·f_roll)`, or a direct
specific resistance `w` (N per kN).

## Scope / non-goals

A sizing & first-pass verification aid — not a catalogue selector and not a
compliance certificate. Mechanism duty group → `hoistdutycalculator/`. Rope/drum
strength, drum-shell stress and full EN 14492/EN 13001 PASS/FAIL compliance are
out of scope here. Verify the final motor/gearbox/brake/inverter against the
supplier's data and the controlling standard before order.

## Future: the configurator wizard

Inputs use a consistent convention so a future "enter the crane once" wizard can
feed one crane-spec object into every tab. Derive travel masses/speeds from the
shared spec rather than duplicating inputs.
