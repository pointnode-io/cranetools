# CLAUDE.md

Guidance for Claude Code (claude.ai/code) across the **cranetools** repository.

This file covers repo-wide conventions. Each tool also has its own `CLAUDE.md`
with tool-specific architecture — Claude Code reads this root file **and** the
tool's file when you work inside a tool directory.

## What this repo is

A collection of standalone crane-engineering design tools. Each tool lives in
its own top-level directory and is self-contained.

| Tool | Directory | Summary |
|---|---|---|
| Crane loading calculator | `craneloadingscalc/` | EN 15011 / EN 13001 proof-of-competence calculator + standards reference (single-file HTML/React apps, no build) |
| Hoist duty calculator | `hoistdutycalculator/` | Mechanism duty group from load spectrum + utilization — FEM 9.511 / ISO 4301-1:1986 and ISO 4301-1:2016, side by side |
| FM1 calcs | `fm1calcs/` | Live rebuild of the firm's Form FM1 design-practice spreadsheet — proof-of-competence to BS 2853:1957 |
| Crane drive sizer | `cranedrivesizer/` | One tool, pick the motion: motor · gearbox · brake sizing for hoist / long-travel / cross-travel. Hoist tab is the deep serial-hoist sizer (static/dynamic/peak/RMS torque, optimum & speed-match gear ratio, stall margin, 125% proof-load, brake & service factors, S1/%ED duty, 87 Hz, per-drive). Travel tabs add the anti-skid drive check. Recommendations round up (won't undersize) |
| Crane estimator | `craneestimator/` | Phase 1 of the "enter the crane once" configurator — single-girder overhead: spec → girder selection & steel weight → installed kW → rough tender cost from an editable rate card (+ £/tonne cross-check). Indicative; expands into the detailed-design tools. Phase 2 adds double-girder / gantry / jib |

When you add a new tool, create a new top-level directory and give it its own
`CLAUDE.md` (and `README.md`).

## Engineering domain (shared context)

These tools target **gantry and overhead crane design to EU/UK standards**. The
normative hierarchy applies across the whole repo:

- **EN 15011** — bridge and gantry cranes (the CE-marking product standard)
- **EN 13001** parts 1–3 — classification, loads & combinations, steel structure limit states
- **EN 1993-6** — crane runway beams (deflection, bottom-flange bending)

Engineering claims (formulas, factors, tooltips badged with a standard) are
authoritative — only change them if you can verify against the actual standard
text. Each tool's `CLAUDE.md` lists its specific calculation rules and the
"do not change" invariants.

## Conventions

- **No build step by default.** Tools are plain HTML/JS opened directly in a
  browser, with dependencies from CDN. Don't introduce a bundler/package manager
  for a tool unless that tool genuinely needs one.
- **One concern per commit**, with a clear message. Branch off `main` for
  non-trivial work and open a PR rather than committing to `main` directly.
- Keep each tool **self-contained** — no cross-tool imports. Shared knowledge
  lives in documentation (this file), not in shared code, until there's a real
  need.

## Collaborating

This repo is worked on by more than one engineer. To keep things smooth:

- Pull before you start; prefer short-lived feature branches + PRs.
- Don't commit local Claude settings — `.claude/settings.local.json` is
  gitignored. Shared Claude config (if any) can go in `.claude/settings.json`.
- If you change an engineering assumption, say so in the PR description and cite
  the standard clause — your reviewer needs to be able to check it.
