# fm1calcs

A live, single-file re-build of the firm's **Form FM1** design-practice
spreadsheet — proof-of-competence calculations for lifting structures to
**BS 2853:1957** (inc. amendments 1–4).

## Files

- **`fm1-workbook.html`** — interactive React workbook. Open directly in a
  browser; no build, no server (React 18 + Babel + Tailwind from CDN).
- **`FM1 calcs.xls`** — the original 24-sheet workbook, kept as the provenance
  reference that every formula and lookup table is derived and verified against.

## What it does

A tabbed workbook, one tab per FM1 sheet. Each tab takes the same "blue cell"
inputs as the paper form (SWL, added load, span, deflection limit, section) and
returns the proof-of-competence checks with live **PASS/FAIL** verdicts:

- Required **Ixx** and section adequacy
- Bending stress **Pbc** vs the allowable from **Table 1** (BS15 steel,
  auto looked-up by `l/ry` × `D/T`, with an editable override)
- **Flange load** vs the 2.2 t/cm² limit
- **Deflection** vs the preferred limit

Sheets implemented: **Crane Beam (1), Gantry Beam (2), Runway Beam (3),
Cantilever (6), Simply Supported (7)**. Other FM1 sheets (compound runway, jib
arms/columns, davit, C-hook, column, A-frame leg, lifting beam) appear as
disabled tabs — next to be added.

## Verification

Every formula is reconstructed from the original sheet's printed algebra and
checked against its stored cell values. A **self-test banner** at the top of the
app re-runs all five worked examples on load and confirms the live results match
the spreadsheet (Required Ixx, bending Pbc, flange load, deflection — 20 checks).

## Faithful reproduction

The original paper form is not internally consistent about which load feeds
which check (e.g. the Simply Supported bending cell uses the rounded design load;
the Cantilever flange cell does too). These mappings are reproduced **exactly**
so the tool ties out to the firm's historical signed calculations. Known
anomalies — notably the Cantilever bending factor — are surfaced with warnings
rather than silently changed. See [`CLAUDE.md`](CLAUDE.md) for the full rules.

> BS 2853:1957 is a withdrawn standard, retained as documented design practice.
> This tool is a design-office aid; every result needs a competent-engineer check.
