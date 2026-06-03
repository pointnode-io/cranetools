# CLAUDE.md — cranedrivesizer

Tool-specific guidance. Read this **and** the repo-root `CLAUDE.md`.

## What this tool is

A single-file HTML/React app (`crane-drive-sizer.html`) that sizes the **drive**
for the three crane mechanisms — **hoist**, **long-travel** and **cross-travel**
— returning, per mechanism:

- required **motor power** (steady and peak-at-end-of-acceleration)
- **motor torque** (steady and steady + acceleration)
- **gearbox ratio** `i = n_motor / n_out`
- minimum **brake torque**
- a **"next standard motor" pick** from the IEC kW ladder, with a breakdown-
  torque margin check
- for travel: the **anti-skid (drive) check** — tractive demand vs wheel/rail
  adhesion — which is the headline result of the two travel tabs.

### Added feature batches (reeving / VFD / realism / duty-energy)

- **Reeving & rope (hoist):** reeving ratio + diverting sheaves, efficiency
  *derived* from sheave count & per-sheave η via `η=(1−η_s^n)/(n(1−η_s))·η_s^nDiv`
  (or manual), lead-line pull, **rope safety** `MBL/S ≥ Z_p` and **drum D/d ≥
  minimum** PASS/FAIL.
- **VFD acceleration:** accel input by *time* or *max rate*; achievable accel =
  `min(skid-limited, torque-limited)` with the limiting factor named; VFD
  overload factor; **regen** energy/power on travel decel and steady hoist
  lowering (→ feeds the brake-resistor tool).
- **Realism:** drives-per-mechanism load sharing; IEC next-standard-motor pick;
  hoist **φ₂** dynamic factor (EN 13001-2 form); optional **counterweight** with
  the overhauling/brake nuance handled.
- **Duty & energy (indicative):** trapezoidal-move RMS torque/power, **%ED**,
  implied starts/hour, and kWh + running cost per year.

**Motor pick is acceleration-aware.** Travel drives are sized on the *greater*
of the continuous demand and `peak_torque_demand / usable_overload` — sizing on
steady running alone picks an absurdly small frame that cannot accelerate the
mass. `peakFactor = vfd ? vfdOverload : 1.6`. Do not regress this to a
steady-power-only pick.

It is the deliberate complement to `hoistdutycalculator/`, which classifies the
mechanism duty group but explicitly **does not** size the motor. This tool picks
up there. It does **not** replace `hoistdutycalculator` — duty class is an input
to drive *selection*, not computed here.

No build step. React 18 + Babel-standalone + Tailwind, all from CDN. Open the
HTML directly in a browser. House style matches the sibling tools (`ACCENT`/
`STEEL` palette, `Badge`/`Field`/`Num` atoms, project stamp + Print/PDF, cited
`ENGINEERING DATA` block, reference `<details>`, Levate footer).

## Architecture

- Three tabs from one `App`: `hoist`, `lt` (long-travel), `ct` (cross-travel).
- **Long-travel and cross-travel share one `TravelTab` component and one
  `computeTravel(t)` engine** — they differ only in defaults, labels and the
  mass that's entered. Keep them sharing the engine; do not fork it.
- Each mechanism has its own `useState` object so values persist across tabs.
- Pure functions `computeHoist(h)` and `computeTravel(t)` hold all the maths and
  return every intermediate for display; the components only render.

## Engineering — the authority split (IMPORTANT)

Two kinds of value live in the `ENGINEERING DATA` block, and they are **not**
equal in authority. Preserve this distinction in any edit:

1. **Kinematic / dynamic relations — exact, edition-independent mechanics.**
   Do not "tune" these:
   - `P = F·v / η` (translation) and `P = m·g·v / η` (hoist)
   - `T[N·m] = 9549.3 · P[kW] / n[rpm]` ( = `P/ω`)
   - `i = n_motor / n_out`, `n_out = v / (π·D)`
   - reflected inertia of a moving mass `J_eq = m·(v/ω)²`
   - `T_acc = (J_mot + J_eq/η)·α`, `α = ω/t_a`
   - hoist lead-line pull `S = m·g / (n_falls · η_reev)`
   - anti-skid limit `F_adh = μ · N_driven`, demand `F_drive = F_resist + m·a`

2. **Empirical / tabulated coefficients — typical DEFAULTS, exposed as editable
   inputs**, in the `DEF` object. These are project/standard/edition-dependent
   and must be confirmed; the UI badges them `default — confirm` (blue
   `Badge tone="def"`). Never present a result as normative when it rests on a
   default:
   - adhesion `μ ≈ 0.15` (≈0.12 wet … 0.20 dry steel)
   - rolling lever `f = 0.5 mm`, bearing `μ = 0.02`, flange factor `c = 1.5`
   - in-service wind `p = 250 N/m²`
   - hoist brake safety factor `1.6 × static load torque`
   - sheave `η_s = 0.98`; mechanism `η = 0.90`
   - rope coefficient `Z_p = 4.0`; drum `D/d min = 20`
   - `φ₂,min = 1.1`, `β₂ = 0.34`; breakdown `2.7×`; VFD overload `1.5×`

   `g = 9.80665 m/s²` is the only fixed physical constant.

   **Why Z_p / φ₂ / D-d are inputs, not baked tables.** Published values for the
   rope coefficient of utilisation, the φ₂ coefficients and the drum/sheave D/d
   minima **disagree across standards and editions** (ISO 4308-1 vs EN 14492-2 vs
   FEM; 1986 vs current ISO 4301-1 bands) — this was checked during the build and
   sources genuinely conflict (e.g. Z_p M1–M8 quoted as 2.3–5.0 by one source,
   3.15–9.0 by others). Rather than assert one set as authoritative — which the
   repo's "verify against the actual standard text" rule forbids — the tool gives
   the **formula** and exposes the coefficient. If you ever bake a per-duty-group
   default table, it MUST be verified against the controlling primary text first,
   and stay editable.

### Running resistance
FEM running-resistance form, with a fallback specific-resistance input:

```
F_roll = (W / D_wheel) · c_flange · (μ_bearing · d_journal + 2 · f_roll)
       — or —  F_roll = w_spec · W / 1000   (w_spec in N per kN)
```

### Brake torque
Referred to the **motor shaft** (where the holding brake usually sits). For a
brake on the drum/wheel shaft, multiply by the gear ratio `i`. Hoist brake =
`SF × static load torque` computed lossless (conservative).

## Scope / non-goals

- **Demand, not selection.** Returns the *demanded* power/torque/ratio and the
  *minimum* brake torque. It does **not** pick a catalogue motor: thermal
  selection — duty type S1–S5 / %ED and permissible **starts per hour** to
  **FEM 9.683** — is a separate check that can govern on short-move, high-cycle
  duties. `%ED` and `starts/hour` are captured on the hoist tab only as a
  reminder for that downstream check; they are not yet used in a calculation.
  If FEM 9.683 motor-power derating is ever added, verify against primary text —
  the hoist-duty tool's notes flag that the only research source for 9.683 was a
  secondary advisory document.
- No VFD/regen sizing, no rope/drum strength (ISO 4308 MBL) yet — candidate
  follow-ups. Rope output here is the **lead-line pull** for reference only.
- Anti-skid uses **even** driven-wheel load distribution; it does not model
  CoG shift, dynamic load transfer or per-wheel skew loads.

## Future: the configurator wizard

Inputs use a deliberately consistent convention (mechanism masses, speeds,
wheel/drum geometry, efficiencies, motor speed) so a future "enter the crane
once" wizard can feed a single crane-spec object into this and the other tools
without retrofitting input models. When wiring that up, derive the travel masses
and speeds from the shared spec — don't duplicate inputs here.

## Provenance

Built to complement `hoistdutycalculator` (which names FEM 9.683 as the motor-
sizing source). The dynamic relations are general mechanics; coefficient
defaults are typical crane values pending confirmation against the controlling
standard (FEM 9.683 / EN 13135 / EN 13001-2) and the project.
