# Quoting Configurator — Feature Plan

> Planning document for the **crane quoting / configurator** feature of the
> (separate) web app. This repo (`cranetools`) is the **engineering reference**:
> its calculators are the verified engines the web app reuses, and the
> `craneestimator/` tool is the working prototype of the quote build-up.
>
> Status: **planning** — schemas and rules below are proposals to confirm.
> Nothing here is built into the web app yet.

## 1. Purpose & scope

- Turn an enquiry into a **fast, consistent, defensible tender price + outline
  spec + client-ready quote document**, repeatable and auditable.
- **Staged:** quick budget estimate → firms up into detailed design once won, on
  the *same* spec object.
- **All crane types:** overhead single-girder, overhead double-girder,
  Goliath/gantry, jib/monorail.

## 2. Architecture

- **Quoting configurator = a proper web app on a Supabase backend** (already in
  development): auth, catalog admin, saved quotes, the configurator UI.
- **Engineering calculators stay standalone** (this repo) and **feed** the app.
- **Single source of truth in Supabase:** component **catalog**, **commercial
  settings** (rates), and saved **quotes**.

### 2.1 Shared engineering modules (key recommendation)
Extract each calculator's `compute…()` logic into **framework-agnostic, pure
JS/TS modules** (no React, no DOM) so **both** the standalone HTML tools **and**
the web app import the *same verified engine* — no re-implementation, no drift.

Proposed engine modules:
- `sizeDriveHoist(spec)` / `sizeDriveTravel(spec)` — from `cranedrivesizer`.
- `sizeStructure(spec)` — girder/section + **fabrication weight** per crane type
  (the price multiplier). Single-girder prototyped in `craneestimator`.
- `classifyDuty(spec)` — from `hoistdutycalculator`.
- `checkStructure(spec, section)` — proof-of-competence, from `craneloadingscalc`
  (detailed/won-job stage).

Each is a pure function: `spec → results`. The HTML tool renders it; the web app
calls it and stores the result on the quote.

### 2.2 Data flow
```
Enquiry ─► Crane Spec ─► engine modules (weight, duty, drives)
        ─► catalog query (bought-in components + cost)
        ─► cost model (steel £/t + bought-out + extras + margin)
        ─► quote document  ─► save quote to Supabase
                                     └► win/loss ─► rate calibration
```

## 3. Data model (proposed)

### 3.1 Crane Spec (shared enquiry object)
```
CraneSpec {
  type            // single-girder | double-girder | gantry | jib | monorail
  swl, hook                       // kg
  span, hol, runwayLength, gauge  // m
  crabArrangement                 // single | twin | tandem
  speeds { hoist, cross, long }   // m/min (+ creep/dual)
  duty { femGroup, loadSpectrum, cyclesOrHours }
  environment { indoorOutdoor, wind, tempRange, hazard }  // hazard: std|hotMetal|personnel
  supply { voltage, phases, frequency, faultLevel }
  controls            // pendant | radio | cabin
  finish              // paint spec | galv
  standards, deflectionLimit
}
```

### 3.2 Catalog (Supabase) — ready-to-go components
One row per orderable component; categories: hoist, wheel block, geared-motor,
brake, VFD, control panel, conductor bar, festoon/energy-chain, buffer, limit
switch, end carriage, pendant/radio, etc.
```
CatalogItem {
  id, category, manufacturer, partNo, description, status   // active|preferred|obsolete
  spec { ... category-specific: swl, femGroup, kW, wheelLoadCap, length, ... }
  cost, currency, leadTimeDays, supplier
  effectiveFrom            // price versioning
}
```

### 3.3 Quote (Supabase)
```
Quote {
  id, ref, customer, date, validUntil, status   // draft|sent|won|lost
  spec: CraneSpec
  engineResults { weightT, dutyGroup, installedKW, driveSelections }
  lines: QuoteLine[]
  commercials { steelRatePerT, overheadPct, marginPct }
  totals { steelwork, boughtOut, labour, overhead, margin, price }
  outcome { wonLost, actualOutturn? }            // calibration feedback
}
QuoteLine {
  category, description, source,   // source: catalog | manual | parametric
  catalogItemId?, qty, unitCost, total, note
}
```

### 3.4 Commercial settings (Supabase)
`£/t of fabrication`, labour rates, install day rate, overhead %, margin %,
quote-doc boilerplate (inclusions, exclusions, lead time, validity, payment
terms, T&Cs), company header. Maintained once.

## 4. Cost model (confirmed)
```
PRICE =
  Steelwork   = £/t_fab × fabrication tonnes          // steelwork ONLY (≈ £3,300/t, flat for now)
+ Bought-out  = Σ component lines                      // catalog / manual / parametric
+ Labour      = engineering hours + install/commission days
+ Delivery
× (1 + overhead%)                                      // → cost
× (1 + margin%)                                        // → price
```
Two views: **internal cost breakdown** + **client price** (single figure or
summarised on the quote). Old material+labour+paint build-up kept only to
*derive/sanity-check* the £/t, not as the engine.

## 5. Component selection logic
Per bought-in line, **source precedence**:
1. **Catalog** — auto-match from Supabase by spec (e.g. hoist where `swl ≥ req`
   AND `femGroup ≥ req`; prefer `status=preferred`, else cheapest). User can swap.
2. **Manual (special)** — override the price, or add a free-text special item
   (description + price + optional supplier/part no.). Saved with the quote;
   **promotable** into the catalog later.
3. **Parametric** — ballpark fallback (£ vs SWL/kW/length) when no catalog match.

Manual override is **always** available on any line, even when the catalog matched.

## 6. Weight models per crane type (the price multiplier)
Because price = £/t × **weight**, the weight estimate dominates accuracy.
- **Single-girder** — prototyped: simply-supported girder, point live load at
  midspan + self-weight, bending + deflection, lightest UB. (Section library is
  indicative; firm in `craneloadingscalc`.)
- **Double-girder** — TBD (box girders, two girders + end carriages).
- **Gantry** — TBD (girder + legs + sill beams; leg height from spec).
- **Jib / monorail** — TBD (different structure entirely).
Validate each against a few real cranes before trusting the price.

## 7. Quote document (client-ready)
Header (company, customer, ref, date, **validity**) · outline spec · **scope of
supply (inclusions)** · **exclusions** · lead time · price (+ optional
spares/options) · commercial terms · T&Cs. Print/PDF. Internal cost breakdown
not shown to client.

## 8. Calibration loop
Every quote saved with spec + lines + price + **win/loss** (+ actual outturn if
known). Periodically tune `£/t`, margin and parametric curves from outcomes. The
`£/tonne` cross-check and `£/tonne-SWL` / `£/kg` metrics are the levers.

## 9. Open items / TBD
- Confirm Crane Spec fields per type.
- Catalog schema attributes per category + matching rules.
- Quote scope: single crane vs project (multiple cranes / options / spares).
- Margin: single % vs varies by type/risk.
- Quote revisions/versioning; currency/VAT; roles/permissions.
- Weight models for double-girder / gantry / jib.
- Quote-doc boilerplate (inclusions/exclusions/lead time/validity/terms).

## 10. Decisions log
- Quoting is a primary purpose; staged estimate → detailed.
- All crane types in scope.
- Cost = steel £/t-of-fab (steelwork only) + bought-out lines + extras + margin.
- £/t of fabrication ≈ £3,300/t (flat for now; may vary by section type/finish).
- Bought-in from Supabase catalog; per-line source Catalog/Manual/Parametric;
  manual price for specials, saved with quote, promotable to catalog.
- Output = full client-ready quote document.
- Quoting configurator = web app on Supabase (auth, catalog, saved quotes);
  calculators standalone and feeding it; quotes persisted → calibration loop.
- Engine logic to be extracted into reusable pure modules shared by both.
