# cranetools

A collection of standalone crane-engineering design tools for gantry and
overhead cranes, built to EU/UK standards (EN 15011 / EN 13001 / EN 1993-6).

Each tool is self-contained in its own top-level directory.

## Tools

| Tool | Directory | Summary |
|---|---|---|
| Crane loading calculator | [`craneloadingscalc/`](craneloadingscalc/) | EN 15011 / EN 13001 proof-of-competence calculator plus a standards reference. Two standalone HTML files — no build, no server. |
| Hoist duty calculator | [`hoistdutycalculator/`](hoistdutycalculator/) | Selects the required hoist mechanism duty group from load spectrum + utilization — FEM 9.511 / ISO 4301-1:1986 (M1–M8 / 1Dm–5m) and the current ISO 4301-1:2016 cycle-based scheme, side by side. Single standalone HTML file. |
| FM1 calcs | [`fm1calcs/`](fm1calcs/) | Live rebuild of the firm's Form FM1 design-practice spreadsheet — proof-of-competence checks to BS 2853:1957. Single standalone HTML file. |
| Crane drive sizer | [`cranedrivesizer/`](cranedrivesizer/) | Sizes the drive (motor power/torque, gearbox ratio, brake torque) for the hoist, long-travel and cross-travel mechanisms, with the anti-skid drive check for travel. Complements the hoist duty calculator. Single standalone HTML file. |

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

### `cranedrivesizer/`

- `crane-drive-sizer.html` — interactive React calculator that sizes the drive
  for the hoist, long-travel and cross-travel mechanisms: required motor power &
  torque (steady and accelerating), gearbox ratio, minimum brake torque, and the
  anti-skid (drive) check for travel. The deliberate complement to the hoist duty
  calculator, which classifies the duty group but does not size the motor
  (it names FEM 9.683 for that).

Open the file directly in a browser.

## Working in this repo

- No build step — open the HTML files directly.
- Branch off `main` and open a PR for non-trivial changes.
- See [`CLAUDE.md`](CLAUDE.md) for conventions and the shared standards context.

## Adding a tool

Create a new top-level directory containing the tool plus its own `README.md`
and `CLAUDE.md` (tool-specific architecture and engineering invariants).
