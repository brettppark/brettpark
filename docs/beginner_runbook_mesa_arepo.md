# SNIA_Sim Beginner Runbook: MESA -> AREPO -> RT

This runbook is written for someone who has never used MESA or AREPO. It explains what each step does, how to run it, what files are produced, and how to decide whether a run is only an engineering test or usable science.

<span style="color:red"><b>Current science boundary:</b> the present AREPO branch can create and evolve WD-like gas structures and DD/D6 binary scaffolds, but it has not passed the isolated-WD science gate. Do not use the current AREPO snapshots as physical SN Ia merger, detonation, Ni56-yield, or observable results.</span>

## 0. What We Are Building

The paper goal is:

```text
initial binary / stellar conditions
  -> final WD structure and composition
  -> 3D hydro merger/accretion/explosion state
  -> homologous ejecta and isotope distribution
  -> radiative transfer light curves/spectra
  -> SALT/SALT3 parameters M_B, x1, c
```

The intended production pipeline has three stages:

1. **MESA / population source stage**
   - Build or read source-traced progenitors.
   - Record ZAMS masses, metallicity, periods, delay times, WD masses, WD composition, cooling state, and profile provenance.
   - Output: `profile*.data`, `history.data`, `*.mod`, summary JSON/CSV, and paper figures.

2. **AREPO hydro stage**
   - Map final WD profiles into 3D Voronoi gas cells.
   - Validate isolated WD hydrostatic stability first.
   - Only then build binary contact, mass-transfer, merger, D6, or SD DDT runs.
   - Output: `IC.hdf5`, `snap_*.hdf5`, binary/orbit diagnostics, and snapshot figures.

3. **Radiative-transfer / observable stage**
   - Convert homologous ejecta and isotope information into RT input.
   - Run ARTIS/SEDONA/TARDIS-style RT.
   - Fit synthetic photometry with sncosmo/SALT3.
   - Output: light curves, spectra, `M_B`, `x1`, `c`, and fitter metadata.

At the moment, this repository has real MESA and AREPO scaffolding, but the hydro stage is still failing the isolated-WD physical validation gate. That is useful: it tells us exactly what physics and mapping must be fixed before any paper claim.

## 1. Basic Folder Map

Always start here:

```bash
cd /Users/brett/Desktop/SNIA_Sim
```

Important folders:

```text
configs/                 AREPO compile-time configuration files
data/paper2/             source tables, gate summaries, paper-facing CSVs
docs/                    methods notes, development plans, this runbook
external_codes/          local copies of MESA, AREPO, ARTIS, SDKs
production_runs/         all MESA/AREPO run directories and logs
results/eos/             EOS/profile mapping CSV and JSON diagnostics
results/figures/         MESA/EOS/paper figures
results/visualizations/  AREPO snapshot PNG frames and contact sheets
scripts/                 repeatable command wrappers and analysis scripts
```

COMPAS lives outside this repository:

```text
/Users/brett/Desktop/COMPAS
```

Treat COMPAS as read-only. If a script reads COMPAS, every output must be written under `/Users/brett/Desktop/SNIA_Sim`.

## 2. Environment Check

The Python analysis environment is named `snia-rt`.

Quick check:

```bash
conda run -n snia-rt python -c "import h5py, numpy, matplotlib, sncosmo; print('snia-rt ok')"
```

Full stack smoke check:

```bash
./scripts/check_full_stack.sh --smoke
```

Notes:

- MESA is under `external_codes/mesa-26.04.1`.
- AREPO is under `external_codes/arepo-public`.
- The current AREPO SN Ia executable for DD/D6 binary work is `external_codes/arepo-public/ArepoSNIaDDD6BinaryDev`.
- MPI runs may need local socket permissions on macOS. If a run fails before doing science with an MPI/socket error, rerun from the normal terminal or approve the MPI command in Codex.

## 3. Science Rules For This Project

Use these rules before adding any value to a grid:

1. Do not invent a physical value just to make the pipeline run.
2. If a value is provisional, mark it as a scaffold or pilot.
3. Every source value should have provenance: COMPAS row, MESA run, literature table, or clearly named diagnostic assumption.
4. A binary AREPO run is not a science run until both isolated WDs pass hydrostatic convergence.
5. An explosion run is not a science run until the EOS, burning, detonation trigger, and tracer/isotope export pass controlled tests.
6. An observable is not a science observable until the ejecta are mapped into RT and fitted with sncosmo/SALT3 with saved metadata.

Use red placeholders when real values are still missing:

```html
<span style="color:red"><b>NEEDS REAL SCIENCE INPUT:</b> replace this pilot value with a COMPAS/MESA/literature-derived value before paper use.</span>
```

## 4. Make Or Read A Progenitor Grid

For the current CO+CO WD merger source, the user provided COMPAS type-6 data:

```text
/Users/brett/Desktop/COMPAS/compas_python_utils/preprocessing/sn_type_6_data.csv
```

That file has:

```text
og_primary     ZAMS primary mass
og_secondary   ZAMS secondary mass
og_period      ZAMS log-period in days
eccentricity   initial eccentricity
metallicity    absolute metallicity, for example 0.001
f_primary      CO WD primary mass when the CO+CO WD pair formed
f_secondary    CO WD secondary mass when the CO+CO WD pair formed
f_period       period when the CO+CO WD pair formed
time           merger time in Myr
sn_type        6 = CO+CO WD merger
```

Create the local Paper II source table without editing COMPAS:

```bash
conda run -n snia-rt python scripts/compas_type6_to_paper2_grid.py \
  --input /Users/brett/Desktop/COMPAS/compas_python_utils/preprocessing/sn_type_6_data.csv \
  --out data/paper2/progenitor_grid_compas_type6_coco_candidates.csv \
  --representative-out data/paper2/progenitor_grid_compas_type6_coco_median_representative.csv
```

The representative case used in the latest DD negative gate is:

```text
case_id = COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013
M1      = 0.822540586339 Msun
M2      = 0.683722207538 Msun
Z       = 0.013
```

## 5. Run MESA Cases

MESA evolves 1D stellar structures. We use it to build or validate WD profiles before mapping into AREPO.

Run an existing MESA case:

```bash
./scripts/run_mesa_case.sh production_runs/mesa_dd_wd_he_shell_ignition
```

Common MESA outputs:

```text
LOGS/history.data      time series: mass, radius, luminosity, age, core quantities
LOGS/profile*.data     radial structure at saved model numbers
*.mod                  restart/model files
mesa_case_summary*.json project summary written by our helper scripts
```

The current compact profile used for hydro validation is:

```text
production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data
```

It is useful for EOS/profile mapping tests, but it is not yet the final source-traced COMPAS WD pair. The source-traced COMPAS bridge attempts are recorded separately and currently remain noncompact bridge seeds.

## 6. Validate MESA Thermodynamics Before AREPO

Before mapping a MESA profile into AREPO, check whether the profile pressure is internally consistent with MESA's EOS sample program.

Run:

```bash
MPLCONFIGDIR=/tmp/mpl-snia conda run -n snia-rt python scripts/validate_mesa_eos_bridge.py \
  production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --samples 80 \
  --min-logrho 3.0 \
  --out-csv results/eos/mesa_profile6_eos_bridge_validation.csv \
  --out-json results/eos/mesa_profile6_eos_bridge_validation_summary.json \
  --out-figure results/figures/mesa_profile6_eos_bridge_validation.png
```

Read the summary:

```bash
conda run -n snia-rt python -m json.tool results/eos/mesa_profile6_eos_bridge_validation_summary.json
```

How to interpret:

- `profile_vs_mesa_eos`: compares MESA profile pressure to MESA EOS pressure. This should be small in the WD interior.
- `proxy_profile_pressure_vs_mesa_eos`: compares the pressure value we inject into the AREPO IC path to MESA EOS pressure.
- `proxy_nonrel_degenerate_vs_mesa_eos`: shows how bad a simple non-relativistic degeneracy approximation is.
- `hydrostatic_residual`: checks the 1D profile's radial pressure support.

Latest result:

```text
MESA profile pressure vs MESA EOS median relative error = 1.739e-4
profile-pressure proxy median relative error            = 9.281e-5
nonrel-degenerate proxy median relative error           = 5.085e-1
```

Meaning:

```text
The MESA profile thermodynamics are internally consistent in the WD interior.
The present AREPO runtime EOS is still not a full Timmes/MESA Helmholtz EOS.
```

## 7. Create An Isolated-WD AREPO Initial Condition

AREPO needs a 3D HDF5 initial condition. The isolated-WD step maps one MESA profile into gas cells and adds a low-density numerical atmosphere.

Recommended diagnostic IC/run command:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_isolated_wd_relaxation.sh \
  --profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --component primary \
  --run-dir production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate \
  --np 1 \
  --n-star 2048 \
  --n-background 2048 \
  --dynamical-times 0.02 \
  --relaxation-tau-fraction 0.2 \
  --spatial-sampling fibonacci_mass_shells \
  --position-jitter-fraction 0.002 \
  --profile-pressure-model mesa_pressure
```

Important options:

```text
--profile
  MESA profile*.data file to map.

--component
  Label used in the manifest. For isolated tests this is usually primary.

--run-dir
  Output directory. Use a new directory for each experiment.

--np
  Number of MPI ranks.

--n-star
  Number of AREPO cells placed inside the WD.

--n-background
  Number of low-density atmosphere/background cells.

--dynamical-times
  How long to evolve in units of the WD dynamical time.

--relaxation-tau-fraction
  Velocity damping timescale during relaxation, in units of the dynamical time.

--spatial-sampling
  random_mass: random mass-coordinate sampling.
  fibonacci_shell: one global Fibonacci sequence; currently poor radial-bin mapping.
  fibonacci_mass_shells: grouped mass shells with Fibonacci angular placement; current best t=0 median mapping.

--position-jitter-fraction
  Small coordinate jitter to prevent pathological Voronoi degeneracies.

--profile-pressure-model
  mesa_pressure: use pressure from the MESA profile for IC internal energy/gamma closure.
```

If you only want to create `IC.hdf5` without running AREPO:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_isolated_wd_relaxation.sh \
  --profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --component primary \
  --run-dir production_runs/arepo_isolated_prepare_only_example \
  --np 1 \
  --n-star 2048 \
  --n-background 2048 \
  --dynamical-times 0.02 \
  --spatial-sampling fibonacci_mass_shells \
  --profile-pressure-model mesa_pressure \
  --prepare-only
```

Files created in the run directory:

```text
IC.hdf5                    AREPO initial condition
isolated_wd_manifest.json  provenance, units, mass, radius, EOS choices
param_relaxation.txt       AREPO parameter file for relaxation
param.txt                  copy used by AREPO
arepo_isolated_wd_relaxation.log
output_relaxation/snap_*.hdf5
```

What is inside `IC.hdf5`:

```text
PartType0/Coordinates      gas cell positions
PartType0/Velocities       gas cell velocities
PartType0/Masses           cell masses
PartType0/InternalEnergy   cell thermal energy variable
PartType0/PassiveScalars   He/C/O/Ni/etc scaffold channels
PartType0/SNIaStarID       1 for primary material, 2 for secondary material, 0 for background
```

## 8. Diagnose The Isolated WD

After a run, summarize conservation and expansion:

```bash
conda run -n snia-rt python scripts/diagnose_arepo_isolated_wd_sequence.py \
  production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate \
  --stage relaxation \
  --out production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate/isolated_wd_sequence_summary.json
```

Plot every snapshot:

```bash
conda run -n snia-rt python scripts/plot_arepo_isolated_wd_sequence.py \
  production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate \
  --stage relaxation \
  --outdir results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames
```

Make a contact sheet:

```bash
conda run -n snia-rt python scripts/make_image_contact_sheet.py \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_000.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_001.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_002.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_003.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_004.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_005.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_006.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_007.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_008.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_009.png \
  results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_frames/snap_010.png \
  --out results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_contact_sheet.png \
  --columns 4 \
  --thumb-width 430
```

Compare the AREPO radial bins to the original MESA profile:

```bash
MPLCONFIGDIR=/tmp/mpl-snia conda run -n snia-rt python scripts/diagnose_arepo_profile_mapping.py \
  --profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --snapshot production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate/output_relaxation/snap_000.hdf5 \
  --bins 48 \
  --out-csv results/eos/arepo_profile6_2048_massshells_mapping_snap000.csv \
  --out-json results/eos/arepo_profile6_2048_massshells_mapping_snap000_summary.json \
  --out-figure results/figures/arepo_profile6_2048_massshells_mapping_snap000.png
```

Repeat for the last snapshot:

```bash
MPLCONFIGDIR=/tmp/mpl-snia conda run -n snia-rt python scripts/diagnose_arepo_profile_mapping.py \
  --profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --snapshot production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate/output_relaxation/snap_010.hdf5 \
  --bins 48 \
  --out-csv results/eos/arepo_profile6_2048_massshells_mapping_snap010.csv \
  --out-json results/eos/arepo_profile6_2048_massshells_mapping_snap010_summary.json \
  --out-figure results/figures/arepo_profile6_2048_massshells_mapping_snap010.png
```

How to judge:

```text
material_mass_msun
  Should be nearly conserved. Large drift means stellar material is numerically leaking into atmosphere or labels are contaminated.

max_density_code_units
  Should not drop strongly during a short relaxation. A strong drop means the WD expands.

rms_material_radius_code_units
  Should remain stable over multiple dynamical times. Growth means the star is not in hydrostatic equilibrium.

max_com_drift_code_units
  Should remain small. Large drift means imbalance or mesh/atmosphere artifacts.

density/pressure radial-bin error
  Should be small at t=0 and remain controlled after relaxation.
```

Latest isolated-WD result:

```text
run = production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate
n_snapshots = 11
material mass fractional change = -4.811e-5
max density = 0.03725 -> 0.02535 code units
rms material radius = 2.90996 -> 3.48675 code units
status = failed science gate
```

Meaning:

```text
The code path runs, writes snapshots, and plots correctly.
The star still expands quickly. Do not use it to seed a physical binary yet.
```

## 9. Create And Run A DD/D6 Binary Scaffold

Only run this after understanding the isolated-WD gate result. A binary run before isolated-WD convergence is an engineering path test, not a physical merger.

Current negative validation command:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013 \
  --source-table data/paper2/progenitor_grid_compas_type6_coco_median_representative.csv \
  --run-dir production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral \
  --allow-analytic-bootstrap \
  --stage merger \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512 \
  --position-jitter-fraction 0.005 \
  --spatial-sampling random_mass \
  --analytic-pressure-model lane_emden_degenerate \
  --recompute-contact-separation-from-current-radii \
  --merger-orbits 0.002 \
  --enable-inspiral
```

Important options:

```text
--case-id
  Selects one row in the source table.

--source-table
  CSV containing progenitor/binary values.

--allow-analytic-bootstrap
  Allows analytic Lane-Emden-like WD structures. This must be considered non-publication.

--primary-profile / --secondary-profile
  Use MESA profile*.data files instead of analytic WDs. This is the intended science path after isolated-WD gates pass.

--stage relaxation
  Damped binary relaxation.

--stage merger
  Merger/inspiral stage using the active parameter file.

--recompute-contact-separation-from-current-radii
  Recomputes binary separation from active WD radii and Eggleton Roche-lobe logic.

--enable-inspiral
  Turns on the artificial inspiral driver. This is a development driver, not a validated gravitational-wave inspiral.

--enable-burning
  Enables reduced burning hook if compiled. Do not use for paper until one-zone and detonation tests pass.

--enable-detonation
  Enables detonation trigger hook if compiled. Do not use for paper until benchmarked.
```

Files produced:

```text
IC.hdf5
binary_ic_manifest.json
param_relaxation.txt
param_merger.txt
output_relaxation/snap_*.hdf5 or output_merger/snap_*.hdf5
arepo_dd_d6_relaxation.log or arepo_dd_d6_merger.log
preexplosion_diagnostics_*/...json
orbit_diagnostics_*/...csv, ...json, ...png
```

## 10. Diagnose The Binary

Plot every binary snapshot:

```bash
conda run -n snia-rt python scripts/plot_arepo_dd_d6_binary_sequence.py \
  production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral \
  --stage merger \
  --outdir results/visualizations/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral_frames
```

Make orbit diagnostics:

```bash
conda run -n snia-rt python scripts/diagnose_arepo_dd_d6_binary_orbit_sequence.py \
  production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral \
  --stage merger \
  --outdir production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/orbit_diagnostics_merger
```

Latest binary negative gate result:

```text
run = production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral
snapshots = snap_000.hdf5 ... snap_010.hdf5
separation = 27.74889 -> 27.72788 code units
separation fractional change = -7.574e-4
orbital angular momentum norm fractional change = -6.510e-1
status = completed negative validation
```

Meaning:

```text
The binary code path runs and writes diagnostics.
The artificial driver dominates angular momentum loss.
This is not yet a physical GW inspiral, Roche mass-transfer, or merger calculation.
```

## 11. RT / Observable Stage

<span style="color:red"><b>Not active for paper science yet:</b> do not run RT for final observables until AREPO produces stable homologous ejecta with validated isotope/tracer export.</span>

AREPO itself does not directly produce:

```text
M_B
x1
c
synthetic spectra
synthetic photometry
```

The intended sequence is:

```text
AREPO final ejecta snapshot
  -> ejecta table / isotope grid
  -> ARTIS/SEDONA/TARDIS radiative transfer
  -> synthetic photometry CSV
  -> sncosmo SALT3 fit
  -> M_B, x1, c
```

Preview-export an AREPO snapshot into an ejecta table:

```bash
conda run -n snia-rt python scripts/arepo_snapshot_to_ejecta_table.py \
  production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/output_merger/snap_010.hdf5 \
  --case-id COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013 \
  --outdir production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/ejecta_export_preview \
  --max-cells 20000
```

This preview export is for file-interface testing only. It is not homologous ejecta and it does not contain validated isotope yields.

The current sncosmo fitting adapter expects a synthetic photometry CSV with:

```text
time,band,flux,fluxerr,zp,zpsys
```

Fit such a CSV with SALT3:

```bash
conda run -n snia-rt python scripts/fit_sncosmo_lightcurve.py \
  path/to/rt_synthetic_photometry.csv \
  --model salt3 \
  --z 0.0 \
  --out path/to/rt_synthetic_photometry.salt3_fit.json
```

The output JSON contains fitted SALT parameters including `x0`, `x1`, and `c`.
The absolute magnitude `M_B` must be computed from the rest-frame B-band peak or
from the calibrated `x0` convention used by the chosen SALT model. That
calibration step must be documented in the paper methods.

## 12. How To Read An AREPO Snapshot In Python

Minimal example:

```python
from pathlib import Path
import h5py
import numpy as np

snap = Path("/Users/brett/Desktop/SNIA_Sim/production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/output_merger/snap_010.hdf5")

with h5py.File(snap, "r") as f:
    gas = f["PartType0"]
    time = float(f["Header"].attrs["Time"])
    x = gas["Coordinates"][:]
    v = gas["Velocities"][:]
    m = gas["Masses"][:]
    rho = gas["Density"][:]
    pressure = gas["Pressure"][:] if "Pressure" in gas else None
    scalars = gas["PassiveScalars"][:] if "PassiveScalars" in gas else None
    star_id = gas["SNIaStarID"][:] if "SNIaStarID" in gas else np.zeros(len(m), dtype=int)

primary = star_id == 1
secondary = star_id == 2

print("time", time)
print("total mass", m.sum())
print("primary mass", m[primary].sum())
print("secondary mass", m[secondary].sum())
print("max density", rho.max())
```

Useful snapshot fields:

```text
Coordinates      cell positions
Velocities       cell velocities
Masses           cell masses
Density          finite-volume gas density
InternalEnergy   gas internal energy variable
Pressure         pressure if written by the active AREPO config
PassiveScalars   composition/tracer scaffold channels
SNIaStarID       component label: 0 background, 1 primary, 2 secondary
ParticleIDs      AREPO particle/cell IDs
```

## 13. What Must Be Fixed Before A Paper-Grade Binary

Current blockers:

1. **Full runtime EOS**
   - Current `SNIA_HELMHOLTZ_EOS` is a Helmholtz-like proxy, not the final Timmes/MESA Helmholtz runtime EOS.
   - The MESA profile pressure is internally consistent, but AREPO runtime thermodynamics must be replaced or calibrated.

2. **MESA-to-AREPO mapping**
   - `fibonacci_mass_shells` improves t=0 median radial mapping.
   - Outer bins and evolved snapshots still fail.

3. **Isolated WD convergence**
   - The isolated WD expands over only `0.02` dynamical times.
   - Need a resolution ladder and multi-dynamical-time stability.

4. **Binary contact/mass transfer**
   - Need to run only after isolated primary and secondary WDs are stable.
   - Need material flux through L1 and angular-momentum conservation after the artificial driver is off.

5. **Burning/network/detonation**
   - Current hooks are scaffolds.
   - Need one-zone burn tests, energy conservation, timestep limiting, and detonation benchmark.

6. **RT and observables**
   - AREPO does not directly output `M_B`, `x1`, or `c`.
   - Need homologous ejecta export, RT, synthetic photometry, then sncosmo/SALT3 fitting.

## 14. Troubleshooting

If MESA fails:

```text
Check the run directory's terminal output, history.data, and mesa_case_summary*.json.
If final.mod is missing, do not treat the run as a final compact WD.
```

If AREPO stops with an MPI/socket error:

```text
This can be a macOS/Codex sandbox permission issue. Rerun from a normal terminal or approve the MPI command.
```

If AREPO stops with a domain decomposition error:

```text
Treat it as a failed numerical gate. Save the log. Do not delete the run; it is useful evidence for the methods log.
```

If matplotlib complains about cache permissions:

```bash
MPLCONFIGDIR=/tmp/mpl-snia conda run -n snia-rt python ...
```

If a run is too slow:

```text
Reduce --n-star, --n-background, or --dynamical-times.
Use --prepare-only to inspect IC creation before committing to a hydro run.
```

If a star looks like it is dissolving:

```text
That is a failed hydrostatic equilibrium gate, not a real physical result.
Check the isolated_wd_sequence_summary.json and radial profile mapping diagnostics.
```

## 15. Current Reference Outputs

EOS validation:

```text
results/eos/mesa_profile6_eos_bridge_validation_summary.json
results/figures/mesa_profile6_eos_bridge_validation.png
```

Isolated WD mass-shell gate:

```text
production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate/isolated_wd_sequence_summary.json
results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_contact_sheet.png
```

Binary negative gate:

```text
production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/orbit_diagnostics_merger/merger_orbit_summary.json
results/visualizations/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral_contact_sheet.png
```

Gate summary:

```text
data/paper2/snia_physics_gate_summary_20260506.csv
```

Use this file when continuing the project after a context reset. It tells you what to run, what to inspect, and what not to claim yet.
