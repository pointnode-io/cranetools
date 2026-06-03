# CLAUDE.md — hoistdutycalculator

Tool-specific guidance. Read this **and** the repo-root `CLAUDE.md`.

## What this tool is

A single-file HTML/React app (`hoist-duty-calculator.html`) that determines the
**required hoist mechanism duty group** from engineering inputs (load spectrum +
utilization). It reports results under **two editions of the same standard,
side by side**:

- **FEM 9.511 / ISO 4301-1:1986** — the hours-based scheme. Mechanism groups
  `1Dm…5m` (FEM) ↔ `M1…M8` (ISO). This is what hoist manufacturers (Demag,
  Konecranes, Verlinde, GIS) still quote and what EN 14492 references, so it is
  the **primary** output.
- **ISO 4301-1:2016** — the current cycle-based scheme. Group classes `A0…A11`
  (with sub-classes `A01/A02/A03`), from utilization classes `U0…U9` (total
  working cycles) and load classes `Qp0…Qp5`.

No build step. React 18 + Babel-standalone + Tailwind, all from CDN. Open the
HTML directly in a browser.

## How a result is produced

All from one set of physical inputs (mean hoist height `X_av`, mean speed
`v_av`, cycles/hour, working hours/day, working days/year, design life years):

1. **Mean cycle time** `t_av = 2·X_av / v_av` — ISO 4301-1:2016 Annex B Eq (B.1)
   (the ×2 covers raise + lower).
2. **Operating time/day (in motion)** = `t_av × cycles/hour × hours/day`.
3. **Total duration of use** (h) = motion-h/day × days/year × years → **T-class**.
4. **Total working cycles** = cycles/day × days/year × years → **U-class**.
5. **Load spectrum factor** `K = Σ (wᵢ/Σw)·(Pᵢ/P_max)³` (cube of the load ratio,
   m = 3) — either computed from a load distribution or set from a picked class.
6. Look up the group: 1986 → `M`; 2016 → `A`.

## Engineering invariants — DO NOT change without checking the standard

These are normative. Every constant in the `ENGINEERING DATA` block of the HTML
is from a cited table; changing one silently changes the duty result.

- **T-class bounds** (1986 Table 4, total motion hours):
  `200, 400, 800, 1600, 3200, 6300, 12500, 25000, 50000, 100000`.
- **Km load-class bounds** (1986 Table 5): `L1 ≤ 0.125, L2 ≤ 0.25, L3 ≤ 0.50, L4 ≤ 1.00`.
- **1986 group selection**: `M = clamp(T_index + L_class − 2, 1, 8)`
  (`T_index` 0–9, `L_class` 1–4). This reproduces ISO 4301-1:1986 **Table 6**
  cell-for-cell — verified against the published L1/L4 rows. If you ever replace
  the formula with an explicit matrix, it must match this.
- **FEM↔ISO ladder**: `M1=1Dm, M2=1Cm, M3=1Bm, M4=1Am, M5=2m, M6=3m, M7=4m, M8=5m`
  (one-to-one; research-verified, beware OCR-garbled web sources rendering 1Bm as “M23”).
- **U-class bounds** (2016 Table 2, total cycles):
  `1.6e4, 3.15e4, 6.3e4, 1.25e5, 2.5e5, 5e5, 1e6, 2e6, 4e6, 8e6`.
- **Qp Kp-bounds** (2016 Table 3): `0.0313, 0.0625, 0.125, 0.25, 0.50, 1.00`.
- **2016 group selection**: `A_index = U_index + Qp_index − 3`, mapped to labels
  `A03…A11`. Verified against ISO 4301-1:2016 **Table 4** (corner cells A03 and A11,
  plus interior cells).
- **K formula**: cube exponent **m = 3** (life ∝ 1/load³). Never change the exponent.
- **Cf / Table 5** and **Tf / Table B.1** constants are present for reference and
  the M-class→A-class bridge; values are from ISO 4301-1:2016.

Subtlety worth preserving in any rework: the 1986 `Km` is strictly *time*-weighted
and the 2016 `Kp` is *cycle*-weighted. The tool uses one weight input for both
(identical Σ·(P/Pmax)³ form); the binning into classes differs by edition. Keep
that note visible to users.

## Scope / non-goals

- Classification only. It does **not** size the hoist motor (FEM 9.683,
  permissible starts/hour) nor compute component-level `Ac` duty. If asked to add
  motor sizing, treat FEM 9.683 as the source and verify against primary text
  (the only research source for it was a secondary HMI advisory document).
- The primary-source extract used to verify the 2016 tables is in
  `iso_extract.txt` (ISO 4301-1:2016 text). `fm1calcs/` is a *separate* engineer's
  tool — do not edit it from here.

## Provenance

Built from a deep-research pass (FEM 9.511 / ISO 4301-1 classification) reconciled
against an older in-house `FEM Calc.xls`. The old sheet's FEM↔ISO map, operating-
time formula, load multiplier and design-life table all checked out; its one
weakness — classifying on *daily* hours instead of the standard's *total duration
of use* — is corrected here (which is also why this tool can return 1Dm/M1, which
the old sheet's formula never could).
