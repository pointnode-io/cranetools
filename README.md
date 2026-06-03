# cranetools

A collection of standalone crane-engineering design tools for gantry and
overhead cranes, built to EU/UK standards (EN 15011 / EN 13001 / EN 1993-6).

Each tool is self-contained in its own top-level directory.

## Tools

| Tool | Directory | Summary |
|---|---|---|
| Crane loading calculator | [`craneloadingscalc/`](craneloadingscalc/) | EN 15011 / EN 13001 proof-of-competence calculator plus a standards reference. Two standalone HTML files — no build, no server. |
| Hoist duty calculator | [`hoistdutycalculator/`](hoistdutycalculator/) | Selects the required hoist mechanism duty group from load spectrum + utilization — FEM 9.511 / ISO 4301-1:1986 (M1–M8 / 1Dm–5m) and the current ISO 4301-1:2016 cycle-based scheme, side by side. Single standalone HTML file. |

### `craneloadingscalc/`

- `crane-calculator.html` — interactive React calculator (loads, combinations,
  section checks, deflection, bottom-flange bending)
- `crane-standards-deep-dive.html` — standards reference with navigation

Open either file directly in a browser.

### `hoistdutycalculator/`

- `hoist-duty-calculator.html` — interactive React calculator that derives total
  duration of use and total working cycles from hoist kinematics + utilization,
  combines them with the load spectrum factor, and returns the FEM/ISO mechanism
  duty group under both the 1986 and 2016 editions of ISO 4301-1.

Open the file directly in a browser.

## Working in this repo

- No build step — open the HTML files directly.
- Branch off `main` and open a PR for non-trivial changes.
- See [`CLAUDE.md`](CLAUDE.md) for conventions and the shared standards context.

## Adding a tool

Create a new top-level directory containing the tool plus its own `README.md`
and `CLAUDE.md` (tool-specific architecture and engineering invariants).
