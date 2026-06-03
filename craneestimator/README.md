# craneestimator

The start of the **"enter the crane once" configurator**. **Phase 1:** a rough
**estimate** for a **single-girder overhead crane** — spec → structure selection
& weight → installed power → **tender cost** (from an editable rate card) →
outline spec.

## Files

- **`crane-estimator.html`** — interactive React app. Open directly in a browser;
  no build, no server.

## What it does

1. **Spec** — SWL, span, HOL, runway, speeds, duty, supply (the shared crane spec).
2. **Structure** — sizes the main girder with real beam mechanics (bending +
   deflection to `span/limit`), picks the lightest UB that passes, and estimates
   total structural steel weight.
3. **Power** — estimates hoist and installed kW.
4. **Cost** — rolls up a rough tender total from an **editable rate card**
   (fabricated steel, hoist, drives/panel, power supply, wheels, sundries,
   engineering, install, overhead, margin), with a **£/tonne cross-check** against
   your current rule-of-thumb and `£/tonne SWL` / `£/kg` sanity metrics.
5. **Outline spec** for the quote, with Print/PDF.

## Important

- **Indicative only.** Rates are seeded with industry-typical figures (no project
  data yet) — expect **±20–30%** until tuned to your shop. Replace the rate card
  with your numbers; the £/tonne cross-check is there to calibrate against your
  current method.
- The section library and weight factors are **indicative** — firm the structure
  in the [Crane Loading Calculator](../craneloadingscalc/) and the drives in the
  [Crane Drive Sizer](../cranedrivesizer/). It is **not a fixed quotation**.

## Roadmap

Phase 2 adds double-girder, gantry and jib/monorail; the same spec then expands
into the detailed-design tools once a job is won.
