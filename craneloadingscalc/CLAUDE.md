# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Two standalone HTML files forming a gantry/overhead crane engineering toolkit:

- **`crane-calculator.html`** — Interactive proof-of-competence calculator (EN 15011 / EN 13001)
- **`crane-standards-deep-dive.html`** — Standards reference document with collapsible cards and scroll-spy nav

No build step, no package manager, no server. Open either file directly in a browser. All dependencies (React 18, Babel standalone, Tailwind CSS) are loaded from CDN.

Target repository: **https://github.com/pointnode-io/cranetools**

## Calculator architecture

The calculator is a single-file React app written in JSX compiled at runtime by Babel. The entire application lives inside one `<script type="text/babel">` block.

### Tab structure

Four tabs — the "Crane Beam Design" tab was merged into the first tab to keep all physical inputs together:

| Tab | Contents |
|---|---|
| **Geometry & Design** | Crane geometry, single/double girder config, beam section, material, fatigue, underslung trolley |
| **φ Factors** | Dynamic factors φ₁–φ₇ |
| **Horizontal & Skew** | Accelerations, buffer force, skew inputs |
| **Combinations** | EN 13001-2 load combinations A/B/C |

### State model

Eight independent input groups managed with `useState`:

| State | Shape | Notes |
|---|---|---|
| `geo` | `{L, a, e1, e2, Lb, beamCount}` | `beamCount`: 1 = single girder, 2 = double girder |
| `mass` | `{ec, trolley, hoist}` | Bridge mass is **auto-calculated** — not a manual input |
| `fac` | `{phi1, phi2, phi4, phi5, phi7}` | EN 13001-2 dynamic factors |
| `hz` | `{aLong, aCross, buffer, alpha, lambda, fOver}` | Horizontal/skew inputs |
| `secType` / `mainSec` / `pfcSec` / `box` | section definition | see section library |
| `mat` | `{fy, gammaM}` | Material properties |
| `fat` | `{dsc, sm, m, gammaMf}` | Fatigue parameters |
| `trolley` | `{nw, g, as}` | Underslung trolley (single girder only) — wheels, gauge (mm), contact length (mm) |

### Bridge mass calculation

`massPerMetre(secType, mainSec, pfcSec, sp)` derives kg/m automatically:
- **UB/UC/PFC**: parses the last token of the section name (e.g. `"457x191x74"` → 74 kg/m); for `+PFC` types, sums both
- **Box beam**: `sp.A × 1e-4 × 7850` (area in cm² → m² × steel density)

`bridgeMassTotal = bridgeMassPerM × L × geo.beamCount` replaces any manual bridge mass input. This feeds `Wb` (vertical self-weight) and `hLongWheel` (longitudinal inertia) in `useMemo`.

### Section library

`UB[]`, `UC[]`, `PFC[]` constant arrays storing `{n, h, b, tw, tf}` (mm). Three pure functions compute design-unit properties from geometry:
- `iProps({h,b,tw,tf})` → `{Ix, Wx, Iy, Wy, Aweb, A}` for any I-section
- `combinedProps(main, pfc)` → combined UB/UC + ∩-cap via parallel axis theorem
- `boxProps({Bt,Tt,Bb,Tb,Hw,Tw,s})` → welded box section properties
- `bottomFlangeDims(secType, mainSec, box)` → `{tf, tw, hw, b}` of the main section's bottom flange (used for the underslung trolley check)

`computeSection(secType, mainSec, pfcSec, box)` dispatches to the correct function. All outputs are in cm-based units (cm³, cm⁴, cm²).

### Computation — useMemo sequence

1. `bridgeMassTotal = bridgeMassPerM × L × beamCount`
2. Per-wheel dead and hoist loads via lever rule (`f1`, `f2` fractions)
3. Skew force (EN 15011 Annex D formula or manual override)
4. Horizontal inertia forces (F = m·a — no gravity factor)
5. Design wheel loads per combination (A/B/C) applying γG, γL, φ factors
6. Moving-load moment envelope: `M = P(Lb − a/2)² / (2Lb)` — Müller-Breslau for 2 equal travelling loads
7. Max shear: `V = P(2 − a/Lb)` (leading wheel at support)
8. Stress checks: bending σ and shear τ checked **separately**
9. Deflection: `δ = Pc(3L² − 4c²) / (24EI)`, limit Lb/600 per EN 1993-6
10. **Bottom flange bending** (single girder only — see below)

`govU = Math.max(uStatic, uFat, uDefl, beamCount===1 ? uFlangeCombo : 0)`

### Bottom flange bending check (single girder, EN 1993-6 §5.7)

Only computed and displayed when `geo.beamCount === 1`. The underslung hoist trolley runs on the bottom flange; wheel loads create transverse bending.

| Step | Formula |
|---|---|
| Design wheel force | `Fw = (γL·Q·φ₂ + γG·m_t·φ₁) / nw` (Combination A) |
| Moment arm (web face → wheel) | `cf = g/2 − tw/2` |
| Dispersion length (30° through web) | `l_eff = as + 1.155 × hw` |
| Transverse flange bending stress | `σ_trans = 6·Fw·cf / (l_eff·tf²)` |
| Biaxial interaction at flange root | `σ_eq = √(σ_x² + σ_trans² − σ_x·σ_trans)` |
| Utilisation | `u = σ_eq / f_Rd` |

Both `σ_x` (longitudinal, tensile at bottom fibre) and `σ_trans` (tensile at flange root from cantilever) are additive at the same critical point. Warnings fire if wheel gauge exceeds the flange width or falls inside the web.

### Unit conventions

Forces in N, lengths in m throughout `useMemo`. The identity `N·m / cm³ = N/mm²` (×1000 each side cancels) means section moduli in cm³ and moments in N·m produce stresses directly in N/mm² without conversion.

### CraneLayoutSVG component

A static plan-view schematic at the top of the Geometry & Design tab. Not driven by live inputs — it is a fixed-coordinate illustration labelling all geometry variables (L, a, Lb, e₁, e₂) and mass callout items (Bridge mass, Trolley mass, SWL). Lb is shown spanning between two adjacent column supports; 4 columns are shown per runway to reflect a realistic building layout.

## Standards context

The engineering domain is **EN 15011:2020** (bridge and gantry cranes, EU/UK CE marking route) and its normative references, primarily **EN 13001** parts 1–3. The calculator implements EN 13001-1 (classification), EN 13001-2 (loads and combinations A/B/C), and EN 13001-3-1 (steel structure limit states). Deflection uses **EN 1993-6** (crane runway beams). Bottom flange bending uses **EN 1993-6 §5.7**. The standards deep-dive documents the full normative reference hierarchy.

## Editing guidelines

- All `EN`-badged field tooltips are engineering claims — only change them if you can verify against the actual standard text.
- The three load combinations (A regular, B occasional, C exceptional) and their default γ values come from EN 13001-2 Table 1; treat as authoritative defaults.
- `pMin` (minimum wheel load) uses trolley at maximum approach to the opposite rail: fraction = `e₂/L`. Both trolley and hoist mass must be included.
- The moving-load moment formula is intentional and correct — it gives the envelope maximum for a travelling crane, not the fixed-position symmetric-load result.
- Do not introduce a von Mises combined stress check mixing bending (extreme fibre) and shear (neutral axis) — these are separate limit states in EN 13001-3-1.
- Section property functions (`iProps`, `combinedProps`, `boxProps`) ignore root radii — intentional (conservative, avoids needing additional tabulated data).
- For the PFC ∩ cap: `h_pfc` (original PFC depth) becomes the cap width; `b_pfc` (original flange width) becomes the vertical leg height. A fit warning fires when `h_pfc − 2·tf_pfc < main flange width`.
- Box beam web inset `s` is measured from the outer edge of the top flange to the outer face of the web. Web centroid from section axis = `Bt/2 − s − Tw/2`.
- Bridge mass is **never a manual input** — always derived from `massPerMetre()`. Do not add a manual bridge mass field.
- The bottom flange check (`uFlangeCombo`) uses the biaxial von Mises form without the shear term — this is correct because bending-root shear is negligible at the web-flange junction.
