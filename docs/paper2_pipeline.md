# Paper II Pipeline: Progenitor Grid to Observables

## Scope

The immediate target is not the full Paper I catalogue. It is a controlled,
source-table-driven grid that starts from stellar initial conditions and ends
with SN Ia observables.

<span style="color:red; font-weight:bold">Do not publish any row that is not traceable to Paper I, a population-synthesis output, a MESA/IFMR bridge, or a cited simulation grid.</span>

Each grid row should preserve:

- ZAMS mass and metallicity
- evolutionary time to CO-WD formation
- WD mass, C/O profile, central density, central temperature
- cooling/delay-time proxy
- channel label: SD, DD, D6/sub-Chandrasekhar
- explosion/ejecta quantities: `M_Ni56`, `M_ej`, velocity scale
- RT/SALT observables: `M_B`, `x1`, `c`

## Three-Stage Simulation Design

### Stage 0: Population Source Table

For source-table generation, local COMPAS is available at:

```text
/Users/brett/Desktop/COMPAS/src/COMPAS
```

The COMPAS checkout must remain read-only for this project. Run it only through
SNIA_Sim wrappers so all outputs stay here:

```bash
./scripts/run_compas_paper2.py --pilot --number-of-systems 20
```

For a real population run, provide the Paper I/SSP-motivated COMPAS options or
a COMPAS grid file stored under `data/paper2/`, not inside the COMPAS checkout.
Then convert the output into a candidate table:

```bash
./scripts/compas_output_to_progenitor_grid.py production_runs/<compas_run>/COMPAS_Output --co-only --require-hubble-merger
```

The resulting `progenitor_grid_compas_candidates.csv` is still not the final
science grid. It must be weighted/selected against Paper I SSP/BPS logic, and
rows promoted to `data_status=science_input` only after that provenance is
documented.

The field-level rulebook for missing values is:

```text
docs/source_value_strategy.md
```

### Stage 1: MESA Progenitor Grid

Use MESA to evolve from initial stellar conditions to WD states. The grid
definition is:

```text
configs/paper2_grid.toml
```

The real science source table must be:

```text
data/paper2/progenitor_grid.csv
```

If it is missing, the generator stops. That is intentional. The source table
must come from Paper I SSP/BPS output, COMPAS/BPASS/COSMIC-like population
synthesis, or explicitly cited literature anchor models.

Generate science run directories:

```bash
./scripts/make_paper2_grid.py
```

For wiring tests only, use the scaffold:

```bash
./scripts/make_paper2_grid.py --pilot --limit 3 --out production_runs/paper2_grid_pilot
```

<span style="color:red; font-weight:bold">The pilot scaffold is labelled `pilot_placeholder_do_not_publish` in every row and metadata file.</span>

Plot a completed MESA case:

```bash
./scripts/plot_mesa_history.py production_runs/paper2_grid/<case>/stage1_mesa_make_co_wd
```

### Stage 2: Hydro and Explosion

MESA handles secular evolution and cooling. Hydro begins when the problem is
dynamical:

- SD: explosion and ejecta-companion interaction, not Myr accretion
- DD/D6: merger, unstable mass transfer, He-shell ignition, CO detonation

AREPO can be developed as the hydro/explosion code, but public AREPO needs a
SN Ia branch before this is publishable. Required additions:

- WD EOS
- nuclear network
- energy release coupling
- detonation/ignition logic
- tracer/isotope export
- homologous ejecta export

The first development hook is now in place and documented in
`docs/arepo_snia_development_plan.md`. Rerun the smoke test and plot the scalar
export diagnostic with:

```bash
./scripts/run_arepo_snia_dev_smoke.sh
conda run -n snia-rt python scripts/plot_arepo_snia_dev_snapshot.py \
  production_runs/arepo_snia_dev_smoke/output/snap_005.hdf5
conda run -n snia-rt python scripts/arepo_snapshot_to_ejecta_table.py \
  production_runs/arepo_snia_dev_smoke/output/snap_005.hdf5 \
  --case-id arepo_snia_dev_smoke \
  --outdir production_runs/arepo_snia_dev_smoke/ejecta_export
```

For the first DD/D6 3D binary pre-explosion scaffold:

```bash
./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id PILOT_D6_THIN_HE_BOOS2021 \
  --source-table data/paper2/progenitor_grid_pilot_scaffold.csv \
  --run-dir production_runs/arepo_dd_d6_binary_pilot \
  --allow-analytic-bootstrap \
  --stage relaxation \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512
conda run -n snia-rt python scripts/plot_arepo_dd_d6_binary_snapshot.py \
  production_runs/arepo_dd_d6_binary_pilot/output_relaxation/snap_010.hdf5 \
  --manifest production_runs/arepo_dd_d6_binary_pilot/binary_ic_manifest.json \
  --out results/visualizations/arepo_dd_d6_binary_pilot_relaxation_snap010.png
```

<span style="color:red; font-weight:bold">This DD/D6 run is a workflow pilot:
the IC is analytically bootstrapped and must be replaced with MESA-mapped WD
profiles before it can be treated as a science model.</span>

To test the MESA-profile mapper with the existing DD He-shell MESA output:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id PILOT_D6_THIN_HE_BOOS2021 \
  --source-table data/paper2/progenitor_grid_pilot_scaffold.csv \
  --run-dir production_runs/arepo_dd_d6_binary_mesa_profile_pilot \
  --allow-analytic-bootstrap \
  --primary-profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --allow-profile-mass-rescale \
  --stage relaxation \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512
conda run -n snia-rt python scripts/plot_mesa_profile.py \
  production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --out results/visualizations/mesa_dd_he_shell_profile6_for_arepo_mapping.png
```

<span style="color:red; font-weight:bold">This mixed-profile pilot is closer to
the science path but still not a science model: only the primary WD is MESA
mapped, the secondary remains analytic, and the MESA primary mass is rescaled to
the pilot literature-anchor row.</span>

To save a figure for every AREPO snapshot in a run:

```bash
conda run -n snia-rt python scripts/plot_arepo_dd_d6_binary_sequence.py \
  production_runs/arepo_dd_d6_binary_both_mesa_profile_pilot \
  --stage relaxation \
  --outdir results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_relaxation_frames
```

The sequence plotter writes one PNG per `snap_*.hdf5` plus `frames.txt`, the
ordered list of generated figures.

### SD/DD Channel Consistency

It is scientifically consistent for the Paper II population catalogue to mix SD
and DD/D6 channels as long as the catalogue is channel-aware rather than
pretending a single hydro stage applies to every progenitor.

For SD, the long accretion and cooling history belongs in MESA or a stellar
binary-evolution source table. AREPO or FLASH should start only when the problem
becomes dynamical: final simmering/convection, flame/DDT/explosion, or
ejecta-companion interaction.

For DD/D6, the contact, stream impact, merger, and pre-explosion geometry are
the dynamical problem, so AREPO is the natural first hydro target. The same
post-hydro RT and SALT fitting should then be used for all channels so the final
observable columns `M_B`, `x1`, and `c` are comparable.

### Stage 3: Radiative Transfer and SALT Fitting

Hydro output must be converted to homologous ejecta input for RT. ARTIS has been
added as the preferred 3D RT candidate; TARDIS and sncosmo are already installed
in the `snia-rt` environment.

RT produces synthetic photometry. Fit it with:

```bash
conda run -n snia-rt ./scripts/fit_sncosmo_lightcurve.py synthetic_photometry.csv --model salt3
```

The result gives `x0`, `x1`, and `c`. `M_B` should be reported from rest-frame
B-band peak photometry or from the calibrated SALT convention used in the paper.

## Emulator Meaning

An emulator is a surrogate model trained or calibrated on a finite grid of
expensive MESA+hydro+RT simulations. It is not a replacement for RT in the
anchor grid. Its role is to interpolate between simulated points and propagate
uncertainties across a larger progenitor population.

For a small paper grid, the paper can show direct RT results. For a larger
population, an emulator is the statistically clean bridge.

## Figure Plan

Minimum paper figures:

1. MESA grid map: final WD mass and C/O profile versus ZAMS mass and metallicity.
2. Delay/cooling map: WD cooling time or delay-time proxy versus channel.
3. MESA histories: central `T-rho` evolution and composition history for
   representative SD and DD/D6 progenitors.
4. Hydro snapshots: density/composition slices from AREPO or FLASH at selected
   dynamical times.
5. Ejecta summaries: `M_Ni56`, `M_ej`, velocity, and isotope stratification.
6. RT light curves: multi-band synthetic photometry for representative models.
7. SALT map: `M_B`, `x1`, and `c` across progenitor-grid axes.
8. Cosmology implication: predicted drift in standardized magnitude or
   `Delta mu(z)` induced by SD/DD mixture evolution.
