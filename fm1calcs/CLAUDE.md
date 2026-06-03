# CLAUDE.md

Guidance for Claude Code when working in the **fm1calcs** tool. Read this
together with the repo-root `CLAUDE.md`.

## What this tool is

A live, single-file re-build of the firm's **"Form FM1"** design-practice
spreadsheet (`FM1 calcs.xls`) — proof-of-competence calculations for lifting
structures to **BS 2853:1957** (inc. amendments 1–4).

- **`fm1-workbook.html`** — tabbed React app. No build step, no server. Open
  directly in a browser; React 18 + Babel standalone + Tailwind load from CDN
  (same convention as `craneloadingscalc/`).
- **`FM1 calcs.xls`** — the original 24-sheet workbook (the source of truth for
  every formula, lookup table and section property). Kept in-tree as the
  provenance reference.

The `.xls` is legacy BIFF; read it with Python `xlrd==2.0.2` (`py -3.12`).
Note `xlrd` exposes only **cached cell values**, not formula strings — the
formulas are reconstructed from the algebraic labels printed in adjacent cells
(e.g. `B14="W1.L3"`, `B15="48.E.s"`) and then verified numerically.

## Architecture

One `<script type="text/babel">` block. The beam forms are **config-driven**: a
single `calc(cfg, s)` engine is parameterised by a per-sheet `cfg` entry in the
`SHEETS` array. Adding a beam form = adding a config entry, not new calc code.

| Symbol | Role |
|---|---|
| `SHEETS` | per-sheet config (model, duty %, load mappings, defaults, formulas, notes) |
| `calc(cfg, s)` | the engine — dispatches on `cfg.model` (`central` / `cantilever` / `gantry`) |
| `pbcLookup(lry, dt)` | bilinear interpolation into Table 1, clamped + off-table flag |
| `selfTest()` / `EXPECT` | regression test: each sheet's worked example vs the original stored cell values |
| `UB` / `UBDATA` | Universal Beam section register (from "Section misc") |
| `T1LRY` / `T1DT` / `T1V` | Table 1 allowable-stress matrix (metric, t/cm²) |
| `BeamSheet` | generic input/output/verdict UI for any `cfg` |

Implemented sheets: Crane (1), Gantry (2), Runway (3), Cantilever (6), Simply
Supported (7). The rest are listed as disabled `FUTURE` tabs.

## Calculation rules — DO NOT CHANGE without re-verifying against the .xls

These tie the tool to the firm's historical signed calculations. `selfTest()`
guards them — keep all sheets ✓ green.

- **Units throughout: tonnes/cm.** Loads → t, lengths → cm, E = 2100 t/cm²,
  stresses → t/cm². Convert at the input boundary only.
- **Allowable bending Pbc** comes from **Table 1** (BS15 steel) indexed by
  `l/ry` and `D/T`; effective length = full span (`l/ry = Lcm / ry`). Beyond
  the tabulated range (l/ry 85–300, D/T 10–50) it is **clamped and flagged** —
  Pbc is an editable override because the original is a manual blue cell where
  the engineer applies judgement on compression-flange restraint.
- **Per-cell load mapping is NOT uniform across sheets** — reproduce exactly:

  | Quantity | Load used | Exceptions |
  |---|---|---|
  | Required Ixx (sizing) | rounded **design** load `W_des` | — |
  | Bending Pbc | **actual** total incl. duty % | Simply Supported uses `W_des` |
  | Flange load `1.4·C·W / K2·T²` | **actual** total | Cantilever uses `W_des` |
  | Deflection | **actual** total | — |

- **Beam models**: central point load → `W·L³/48EI`, `M = W·L/4`; cantilever tip
  → `W·L³/3EI`; gantry two-wheel (`W₁ = W/2` at offset `a`) →
  `W₁·a²·(3L−4a)/6EI`. Gantry bending uses 10% W; gantry flange uses full W.

## Known anomalies in the original form (surfaced, not silently propagated)

- **Cantilever bending** uses the crane-beam `W·L/4·Zxx` factor. A true
  cantilever moment is `W·L`, so the correct stress is **4× higher**. The tool
  shows both (`calPbcCorrect`) with an amber warning; faithful value retained
  for audit parity. Flag for the responsible engineer — do not "fix" silently.
- **Simply Supported** bending references the design load, not the actual total
  (likely a cell-reference slip). Reproduced as-is.

These are deliberate. If a future change "corrects" them, it must be a
documented engineering decision in the PR, not an incidental edit.

## Section properties (Blue Book provenance)

`UBDATA` is the firm's "Section misc" register. Cross-checked against SCI P363
(Blue Book): for every section the tabulated `Ix` sits ~1–2% above a no-fillet
geometry calc, which is the expected root-fillet contribution — i.e. the values
agree with published Blue Book properties (spot checks: 610x229x125 `Ix` 98579
vs 98610; 305x305x198 `Zx` 2995 exact).

A few entries are **superseded serial sizes** kept for historical jobs and
flagged via `LEGACY` in the UI (452x152x60/67 → 457x152; 452x191x67/82/89/98 →
457x191; 533x210x90 → 533x210x92). Do not delete them (they tie to old calcs);
if you refresh the register to current sizes, keep the legacy ones available and
keep the `LEGACY` map in sync.

## Engineering caveat

BS 2853:1957 is a **withdrawn** standard, retained as the firm's documented
design practice. The tool is a design-office aid; every result needs a
competent-engineer check (the form's "Ch'k by" line).
