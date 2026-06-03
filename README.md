# cranetools

Crane calculation tools — gantry and overhead crane design to EN 15011 /
EN 13001 / EN 1993-6.

Each tool is self-contained in its own directory.

## Tools

### [`craneloadingscalc/`](craneloadingscalc/)

EN 15011 / EN 13001 proof-of-competence loading calculator plus a standards
reference document. Two standalone HTML files — no build, no server.

- `crane-calculator.html` — interactive React calculator (loads, combinations,
  section checks, deflection, bottom-flange bending)
- `crane-standards-deep-dive.html` — standards reference with navigation

Open either file directly in a browser.

## Working in this repo

- No build step — open the HTML files directly.
- Branch off `main` and open a PR for non-trivial changes.
- See [`CLAUDE.md`](CLAUDE.md) for conventions and the shared standards context.

## Adding a tool

Create a new top-level directory containing the tool plus its own `README.md`
and `CLAUDE.md` (tool-specific architecture and engineering invariants).
