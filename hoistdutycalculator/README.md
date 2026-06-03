# Hoist Duty Calculator

Works out the **required hoist mechanism duty group** from the load spectrum and
how hard the hoist is worked — under both editions of ISO 4301-1, side by side:

- **FEM 9.511 / ISO 4301-1:1986** — groups `1Dm…5m` / `M1…M8` (what hoist
  manufacturers quote; the primary result)
- **ISO 4301-1:2016** — current cycle-based group classes `A0…A11`

Single standalone HTML file — no build, no server.

## Use

Open `hoist-duty-calculator.html` in any modern browser.

Enter:

| Input | Meaning |
|---|---|
| Mean hoisting height `X_av` | average lift distance per cycle (m) |
| Mean hoisting speed `v_av` | average hoist speed (m/min) |
| Cycles per hour | lifts per hour while the hoist is working |
| Working hours per day | hours the crane is available per day |
| Working days per year | typically ~250 |
| Design life | expected service life (years) |
| Load spectrum | either pick a class (Light/Medium/Heavy/Very heavy) **or** enter a load distribution and let it compute the factor `K` |

You get the FEM group (e.g. `3m`), the ISO mechanism group (`M6`), and the
current-edition group class (`A`), along with the derived total duration of use,
total working cycles and load spectrum factor.

## How it works

From the kinematic inputs it derives the mean cycle time `t_av = 2·X_av/v_av`
(ISO 4301-1:2016 Annex B), the **total duration of use** in motion-hours and the
**total number of working cycles** over the design life. These give the
utilization class; combined with the load spectrum factor they fix the duty group:

- 1986: `T-class × L-class → M` (FEM `1Dm…5m`)
- 2016: `U-class × Qp-class → A` (`A0…A11`)

All classification tables are taken from the cited standard clauses — see
[`CLAUDE.md`](CLAUDE.md) for the exact values and the "do not change" invariants.

## Important

Classification only. It does **not** size the hoist motor (that's FEM 9.683 —
permissible starts/hour) or address component-level duty. Always check the result
against the controlling standard before placing an order.

## Background

Replaces an older in-house `FEM Calc.xls`. The spreadsheet's logic was largely
sound, but it classified on *average daily operating hours*; the standard
classifies on **total duration of use over the whole design life**. This tool
uses the standard's axis (which also means it can return the lightest group,
1Dm/M1, that the old sheet's formula could never produce).
