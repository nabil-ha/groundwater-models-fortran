# Groundwater Models in Fortran

Working versions of two classic groundwater programs whose original FORTRAN 77 listings were printed in the appendices of a groundwater hydrology textbook. I digitized the printed listings, cleaned them up, and got them compiling and running with modern `gfortran`.

| Program | What it computes | Original listing |
|---|---|---|
| [`grwater/`](grwater) **GRWATER** | 2-D finite-difference groundwater flow: the areal distribution of heads in confined or unconfined aquifers, including an aquifer switching between the two. Fully implicit central-difference scheme solved by Gauss elimination, with impermeable, constant-head and underflow boundaries. | Appendix C, "Finite Difference Ground-Water Model" |
| [`odast/`](odast) **ODAST** | 1-D analytical solute transport: dimensionless concentration C/C0 over distance and time, considering convection, dispersion, decay and adsorption (retardation). | Appendix B |

## What I did

- Transcribed both programs from the printed pages into source files and fixed the transcription errors needed to compile (a few scanning typos remain in the comments).
- **GRWATER:** converted from fixed-form FORTRAN 77 to free-form Fortran 90 (`!` comments, `&` continuations) and added interactive prompts for every input value.
- **ODAST:** reads its input from `INPUT.TXT` in free format and writes the results table to `OUTPUT.TXT`. A sample input and its output are included.

## Build and run

Any Fortran compiler works; these steps use `gfortran`.

> **macOS with Homebrew GCC:** if linking fails with `library 'System' not found`, run `export SDKROOT=$(xcrun --show-sdk-path)` first.

### ODAST

```bash
cd odast
gfortran -o odast odast.f
./odast            # reads INPUT.TXT, writes OUTPUT.TXT
```

`INPUT.TXT` format:

| Line | Contents |
|---|---|
| 1 | `NUMX, NUMT`: number of distances and number of times |
| 2 | `NUMX` distances from the source (m) |
| 3 | `NUMT` times since the start of operation (years) |
| 4+ | One scenario per line: `D1, V1, R, T0, ALAM1, ALFA1`. A line of zeros ends the input. |

Where `D1` = dispersion coefficient (m²/day), `V1` = pore-water velocity (m/day), `R` = retardation factor, `T0` = total period of waste recharge (years), `ALAM1` = decay factor of the solute (1/day), `ALFA1` = decay factor of the source (1/day).

Compare your run against [`sample-OUTPUT.TXT`](odast/sample-OUTPUT.TXT).

### GRWATER

```bash
cd grwater
gfortran -o grwater grwater.f90
./grwater
```

The program prompts for a title, the grid size (`NR`, `NC`, up to 12 × 12), whether the aquifer is confined (`ICFAQ`), the time step `DT`, the simulation time `ST` and the print interval `FWTOP`. It then asks for the physical data (grid spacing, permeability, effective porosity, bedrock and ground-surface elevations, storage coefficient) and the initial heads. Units are English (feet, days, acre-feet).

Boundary types are encoded in the initial head values:

| Initial head `H(I,J)` | Meaning |
|---|---|
| < 10000 | water-table elevation (no boundary) |
| 10000 – 20000 | impermeable boundary |
| 20000 – 30000 | underflow boundary |
| 30000 – 40000 | constant-head boundary |

Pumping is set in subroutine `QFIX`. The example sets a well at grid (2,2) discharging 0.5 acre-ft/day; edit it for your own problem.

## Credit

The numerical models and their original FORTRAN 77 listings belong to the authors of the textbook in which they were published (Appendices B and C). This repository contains my digitized and modernized versions, for study and reuse. If you hold the rights and would like this changed, please open an issue.
