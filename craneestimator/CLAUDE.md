# CLAUDE.md — craneestimator

Tool-specific guidance. Read this **and** the repo-root `CLAUDE.md`.

## What this tool is

The start of the **"enter the crane once" configurator** — Phase 1: an **estimate**
for a **single-girder overhead (top-running)** crane. From a minimal spec it
selects a girder, estimates structure weight and installed power, and rolls up a
**rough tender cost** from an editable **rate card**, plus an outline spec.

Staged by design: the same `spec` object is meant to **expand into the
detailed-design tools** (Crane Loading Calculator, Crane Drive Sizer, electrical)
once a job is won. Phase 2 adds **double-girder, gantry, jib/monorail** (stubbed
in the type selector).

No build step. React 18 + Babel + Tailwind from CDN. House style as siblings.

## Authority / honesty (important)

- **Structure sizing uses real beam mechanics** (simply-supported girder, point
  live load `P=(SWL+hook+crab)·g·φ` at midspan, self-weight UDL; bending
  `σ=M/Wx ≤ fy/γM` and deflection `δ=PL³/48EI ≤ L/limit`), picking the **lightest
  UB** that passes. But the **`UB` section library is INDICATIVE** (mass/Ix/Wx
  from memory of the blue book) and the **crab / end-carriage / misc weight
  factors are rough** — for a firm design use the Crane Loading Calculator. The
  UI says so.
- **Every cost is a rate in the editable `RATES` "rate card", SEEDED with
  industry-typical GBP values — there is no project data behind them yet.** Expect
  **±20–30%** until tuned against real jobs. Never present the number as a fixed
  quote. Replace rates with the shop's figures as they arrive.
- `g = 9.81 m/s²`.

## Cost model

`total = (Σ line items) × (1+overhead%) × (1+margin%)`. Line items: fabricated
steel (material £/kg + fab hrs/tonne × labour), paint, hoist (base + £/t SWL ×
duty), motors/VFD/panel (base + £/kW on estimated installed kW), conductor bar
(£/m × runway), wheels/bogies (base + £/t crane), sundries, engineering hours,
install days. **`£/tonne cross-check`** (`tRate`, basis = per-tonne SWL or
per-tonne crane steel) compares the firm's rule-of-thumb against the build-up —
the gap is the calibration signal. **The firm currently estimates on £/tonne** —
get their actual figure + basis and set `tRate`/`tBasis`.

## Calibration loop

The point of "parametric from scratch": seed typical rates, then tune from real
outcomes. The `£/tonne cross-check` and the `£/tonne SWL` / `£/kg crane` sanity
metrics are the levers. When real job prices arrive, adjust the rate card (and,
later, log each estimate so the curves self-improve).

## Scope / non-goals (Phase 1)

Single-girder overhead only; estimate depth only. No detailed load combinations,
fatigue, runway/civil reactions, or firm component selection — those are the
detailed-design tools the spec will feed. Not a fixed quotation.

## Roadmap

- Phase 2: double-girder (box), gantry (legs), jib/monorail structure modules.
- Phase 3: wire the shared `spec` into the detailed-design tools for the
  "expand once won" stage.
- Later: vendor price lists / per-job logging to calibrate the parametric curves.
