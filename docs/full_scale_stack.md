# Full-Scale Stack Status

This document records what is actually installed in this workspace and what is
still required for research-grade Type Ia supernova simulations.

## Installed Locally

### MESA

Path:

```text
external_codes/mesa-26.04.1
```

MESA SDK path:

```text
external_codes/mesasdk
```

Status:

- MESA source `26.04.1` downloaded from Zenodo and unpacked
- MESA SDK `aarch64-macos-26.3.2` unpacked locally
- full `./install` completed successfully
- `star`, `binary`, and `astero` packages built, tested, and exported

macOS note: the SDK PGPLOT library expected `/opt/X11/lib/libX11.6.dylib`. This
machine did not have XQuartz installed with sudo privileges, so
`libpgplot.dylib` was patched to use Homebrew's `libX11.6.dylib` and was then
ad-hoc codesigned.

Environment:

```bash
source ./scripts/env_mesa.sh
```

### AREPO

Path:

```text
external_codes/arepo-public
```

Status:

- public AREPO cloned from the official MPCDF GitLab repository
- Makefile patched with a local `HomebrewArm` system type
- built successfully with OpenMPI, GSL, HDF5, GMP, FFTW, and hwloc
- `production_runs/arepo_shocktube` completed a 1D shock-tube smoke test
- `ArepoSNIaDev` builds with SN Ia development hooks and completed
  `production_runs/arepo_snia_dev_smoke`
- `ArepoSNIaDDD6Dev` builds with DD/D6/sub-Chandrasekhar diagnostics modules and
  completed `production_runs/arepo_snia_dd_d6_dev_smoke`
- `ArepoSNIaSDDDTDev` builds with SD-DDT flame/turbulence/DDT diagnostics modules
  and completed `production_runs/arepo_snia_sd_ddt_dev_smoke`

Important limitation: this public tree is not a complete SN Ia/DD merger
production setup. It does not ship the private or paper-specific SN Ia nuclear
networks, detonation/ignition modules, tracer export, or detailed WD merger setup
used in many production calculations.

The local branch now has compile-time hook scaffolding for those pieces. See
`docs/arepo_snia_reference_matrix.md` and `docs/arepo_snia_development_plan.md`.

Environment:

```bash
source ./scripts/env_arepo.sh
```

### Radiative Transfer and SALT Fitting

Conda environment:

```text
snia-rt
```

Installed:

- TARDIS `2026.4.26`
- sncosmo `2.12.1`
- ARTIS `sn3d` and `exspec`, built from `external_codes/artis`

Environment:

```bash
source ./scripts/env_rt.sh
```

ARTIS environment:

```bash
source ./scripts/env_artis.sh
```

The working macOS ARTIS build command was:

```bash
cd external_codes/artis
export OMPI_CXX=/opt/homebrew/opt/llvm/bin/clang++
make -j4 EIGEN=OFF OPTIMIZE=OFF
```

GCC failed on this machine because the Homebrew GCC fixed headers expected
`_bounds.h`, which is absent from the installed CommandLineTools SDK. Apple
clang 15 failed because ARTIS currently requests `-std=c++26`. Homebrew LLVM
clang 20 built both binaries.

TARDIS is useful for rapid spectral experiments and line identification. For
full time-dependent SN Ia light curves and robust `M_B`, `x1`, `c` comparisons,
you will still want SEDONA, ARTIS, SuperNu, or equivalent ejecta radiative
transfer. sncosmo is installed for SALT2/SALT3 fitting once synthetic photometry
exists.

## Run Seeds in This Workspace

### MESA CO WD Grid Seed

```text
production_runs/mesa_make_co_wd_grid_seed
```

Based on MESA's `make_co_wd` test suite. Use this to build CO WD progenitor grids
over initial mass and metallicity before cooling/accretion experiments.

### SD Chandrasekhar Ignition Seed

```text
production_runs/mesa_sd_wd_c_core_ignition
```

Based on MESA's `wd_c_core_ignition` test suite. It starts from a CO WD,
removes the helium layer, then accretes a C/O mixture until carbon burning
reaches thermonuclear-runaway conditions. This is the local starting point for
varying cooling time, accretion rate, metallicity, and ignition density in SD
studies.

### DD or D6 He-Shell Ignition Seed

```text
production_runs/mesa_dd_wd_he_shell_ignition
```

Based on MESA's `wd_he_shell_ignition` test suite. It loads a CO WD with a
surface layer and accretes helium until helium ignition. This is a starting
point for sub-Chandrasekhar double-detonation/D6 progenitor grids.

### AREPO Hydro Smoke Test

```text
production_runs/arepo_shocktube
```

This confirms the local AREPO binary runs and writes snapshots. It is not a SN Ia
calculation.

### AREPO SN Ia Development Smoke Tests

```text
production_runs/arepo_snia_dev_smoke
production_runs/arepo_snia_dd_d6_dev_smoke
production_runs/arepo_snia_sd_ddt_dev_smoke
```

These confirm the local SN Ia AREPO development executables compile with the
branch-specific module flags and `PASSIVE_SCALARS=8`. The runs write
`PassiveScalars` to HDF5 snapshots, branch manifests, and diagnostics files.
They are visualized at:

```text
results/visualizations/arepo_snia_dev_smoke.png
results/visualizations/arepo_snia_dd_d6_dev_smoke.png
results/visualizations/arepo_snia_sd_ddt_dev_smoke.png
```

They are still diagnostic shock-tube wiring tests, not physical SN Ia explosions.
The manifests explicitly record `physics_state_changes=disabled`.

### COMPAS Smoke Tests

```text
production_runs/compas_smoke
production_runs/compas_wd_smoke
production_runs/compas_pilot_wrapper_test
```

These confirm the local read-only COMPAS executable can produce BSE CSV files
inside SNIA_Sim and that WD binaries can be exported to
`BSE_Double_Compact_Objects.csv`.

## Verification Commands

Check installed components:

```bash
./scripts/check_full_stack.sh
```

Run lightweight smoke checks:

```bash
./scripts/check_full_stack.sh --smoke
```

Run a MESA seed:

```bash
./scripts/run_mesa_case.sh production_runs/mesa_sd_wd_c_core_ignition
```

Run AREPO shock tube:

```bash
./scripts/run_arepo_shocktube.sh
```

Run AREPO SN Ia development smoke:

```bash
./scripts/run_arepo_snia_dev_smoke.sh
./scripts/run_arepo_snia_dev_smoke.sh --branch dd-d6
./scripts/run_arepo_snia_dev_smoke.sh --branch sd-ddt
conda run -n snia-rt python scripts/plot_arepo_snia_dev_snapshot.py \
  production_runs/arepo_snia_dev_smoke/output/snap_005.hdf5
conda run -n snia-rt python scripts/arepo_snapshot_to_ejecta_table.py \
  production_runs/arepo_snia_dev_smoke/output/snap_005.hdf5 \
  --case-id arepo_snia_dev_smoke \
  --outdir production_runs/arepo_snia_dev_smoke/ejecta_export
```

## What Still Needs to Be Built for the Science Goal

To obtain final `M_B`, `x1`, and `c` from progenitor parameters, the remaining
research work is:

1. Define the SD and DD progenitor parameter grids.
2. Run MESA grids and export final profiles at ignition or pre-ignition.
3. Write a controlled MESA-to-hydro mapping with units, composition, and EOS
   consistency checks.
4. Add or obtain SN Ia-specific hydro physics for the explosion phase.
5. Evolve ejecta to homologous expansion and export isotope distributions.
6. Run time-dependent radiative transfer for synthetic photometry.
7. Fit synthetic light curves with sncosmo SALT2/SALT3 to recover `x1` and `c`;
   compute `M_B` from the synthetic rest-frame B-band peak.
