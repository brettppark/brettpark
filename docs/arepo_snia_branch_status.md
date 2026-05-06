# AREPO SN Ia Branch Status

<span style="color:red"><b>Current status:</b> the DD/D6 and SD-DDT AREPO branches are diagnostics-only. They compile, run, write passive isotope-like scalars, and export branch diagnostics. The DD/D6 branch now has a stable non-publication 3D CO+CO WD relaxation baseline using a cold `n=3/2` Lane-Emden degeneracy bootstrap and active internal-velocity damping. These runs do not yet perform physical burning, detonation, deflagration, DDT, mass-transfer validation, or publishable ejecta generation. The paper-use gate list is maintained in `docs/paper_grade_physics_validation_gates.md`.</span>

## Branches

| Branch | Command | Executable | Run directory | Active SN Ia modules |
|---|---|---|---|---|
| Base scalar bridge | `./scripts/run_arepo_snia_dev_smoke.sh` | `ArepoSNIaDev` | `production_runs/arepo_snia_dev_smoke` | network scaffold, detonation diagnostics, tracer bridge |
| DD/D6/sub-Chandra | `./scripts/run_arepo_snia_dev_smoke.sh --branch dd-d6` | `ArepoSNIaDDD6Dev` | `production_runs/arepo_snia_dd_d6_dev_smoke` | DD/sub-Chandra branch flag, D6 branch flag, detonation diagnostics, tracer bridge |
| SD Chandrasekhar DDT | `./scripts/run_arepo_snia_dev_smoke.sh --branch sd-ddt` | `ArepoSNIaSDDDTDev` | `production_runs/arepo_snia_sd_ddt_dev_smoke` | SD-DDT branch flag, flame diagnostics, turbulence diagnostics, DDT diagnostics, tracer bridge |
| DD/D6 3D binary pre-explosion | `./scripts/run_arepo_dd_d6_binary_stage.sh --stage relaxation` | `ArepoSNIaDDD6BinaryDev` | `production_runs/arepo_dd_d6_binary_pilot` | 3D self-gravity hydro, component labels, passive composition scalars, hotspot diagnostics |

## Current Stable Development Baseline

The first COMPAS-derived DD baseline that behaves like a pressure-supported WD
binary relaxation is:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013 \
  --source-table data/paper2/progenitor_grid_compas_type6_coco_median_representative.csv \
  --run-dir production_runs/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate \
  --allow-analytic-bootstrap \
  --stage relaxation \
  --np 10 \
  --n-cells-total 16384 \
  --n-background 1024 \
  --position-jitter-fraction 0.01 \
  --spatial-sampling random_mass \
  --analytic-pressure-model lane_emden_degenerate \
  --recompute-contact-separation-from-current-radii
```

It uses the local COMPAS type-6 COCO merger representative:

```text
case_id = COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013
M1 = 0.822540586339 Msun
M2 = 0.683722207538 Msun
Z = 0.013
delay_time = 3.89237585145 Gyr
```

It completed to:

```text
production_runs/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate/output_relaxation/snap_010.hdf5
```

and wrote:

```text
results/visualizations/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate_frames/snap_000.png
...
results/visualizations/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate_frames/snap_010.png
production_runs/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate/orbit_diagnostics_relaxation/relaxation_orbit_diagnostics.png
```

Relaxation diagnostics:

```text
separation_start_code_units = 27.7297965131
separation_end_code_units = 27.7755176468
separation_fractional_change = 0.00164880884
orbital_l_norm_fractional_change = 0.000190581599
final_max_density_code_units = 0.00202553246
```

<span style="color:red"><b>Science warning:</b> this is the current AREPO engineering baseline, not a publishable merger. It is still an analytic cold-degenerate bootstrap, not a pair of source-consistent MESA/WDEC WD structures evolved from the COMPAS ZAMS history. Full Helmholtz EOS, isolated-WD relaxation/convergence, Roche mass-transfer validation, burning/network coupling, and detonation criteria remain required before any SN Ia explosion or observable claim.</span>

## Current COMPAS Type-6 Binary Gate: 2026-05-04

The latest source-row binary engineering gate is:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013 \
  --source-table data/paper2/progenitor_grid_compas_type6_coco_median_representative.csv \
  --run-dir production_runs/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax \
  --allow-analytic-bootstrap \
  --stage relaxation \
  --np 1 \
  --n-cells-total 8192 \
  --n-background 512 \
  --position-jitter-fraction 0.005 \
  --spatial-sampling random_mass \
  --analytic-pressure-model lane_emden_degenerate \
  --recompute-contact-separation-from-current-radii \
  --relaxation-orbits 0.002
```

It produced:

```text
production_runs/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax/output_relaxation/snap_000.hdf5
...
production_runs/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax/output_relaxation/snap_010.hdf5
production_runs/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax/preexplosion_diagnostics_relaxation/COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013_preexplosion_summary.json
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_frames/snap_000.png
...
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_frames/snap_010.png
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_contact_sheet.png
```

Final diagnostic summary:

```text
time_code_units = 0.12942280744627518
mass_total_msun = 1.506262843428542
primary_mass_msun = 0.8225377763580763
secondary_mass_msun = 0.6837047325359187
separation_code_units = 27.72783734112842
max_hotspot_proxy = 4.037089733072138e-5
```

<span style="color:red"><b>Science warning:</b> this run completed the 3D
binary AREPO path with self-gravity, component diagnostics, passive composition
scalars, EOS hook, and relaxation damping. It is deliberately short
(`0.002 orbit`) and analytic-bootstrap. It is not an inspiral, physical
Roche-mass-transfer sequence, merger, detonation, or observable-producing
simulation.</span>

## DD/D6 3D Binary Pre-Explosion Scaffold

The first DD/D6 binary scaffold has been run with:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id PILOT_D6_THIN_HE_BOOS2021 \
  --source-table data/paper2/progenitor_grid_pilot_scaffold.csv \
  --run-dir production_runs/arepo_dd_d6_binary_pilot \
  --allow-analytic-bootstrap \
  --stage relaxation \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512
```

It produced snapshots:

```text
production_runs/arepo_dd_d6_binary_pilot/output_relaxation/snap_000.hdf5
...
production_runs/arepo_dd_d6_binary_pilot/output_relaxation/snap_010.hdf5
```

and diagnostics:

```text
production_runs/arepo_dd_d6_binary_pilot/preexplosion_diagnostics_relaxation/PILOT_D6_THIN_HE_BOOS2021_preexplosion_summary.json
production_runs/arepo_dd_d6_binary_pilot/preexplosion_diagnostics_relaxation/PILOT_D6_THIN_HE_BOOS2021_hotspot_candidates.csv
results/visualizations/arepo_dd_d6_binary_pilot_relaxation_snap010.png
```

<span style="color:red"><b>Science warning:</b> this pilot uses an analytic bootstrap IC from cited Roche-lobe and cold-WD mass-radius relations, not a MESA-mapped thermal/composition profile. It is a workflow proof, not a publishable relaxed WD merger model.</span>

The next mapper step has also been tested with the existing MESA DD He-shell
profile as the primary WD:

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
```

This produced:

```text
production_runs/arepo_dd_d6_binary_mesa_profile_pilot/output_relaxation/snap_010.hdf5
production_runs/arepo_dd_d6_binary_mesa_profile_pilot/preexplosion_diagnostics_relaxation/PILOT_D6_THIN_HE_BOOS2021_preexplosion_summary.json
results/visualizations/mesa_dd_he_shell_profile6_for_arepo_mapping.png
results/visualizations/arepo_dd_d6_binary_mesa_profile_pilot_relaxation_snap010.png
```

<span style="color:red"><b>Science warning:</b> this is still not publishable because only the primary WD is MESA-profile mapped. The secondary is analytic, the primary profile is mass-rescaled to the pilot literature-anchor row, and profile temperatures are converted through an ideal-ion placeholder until the Helmholtz EOS exists.</span>

A both-profile mapper visualization run was also completed:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id PILOT_D6_THIN_HE_BOOS2021 \
  --source-table data/paper2/progenitor_grid_pilot_scaffold.csv \
  --run-dir production_runs/arepo_dd_d6_binary_both_mesa_profile_pilot \
  --primary-profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --secondary-profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --allow-profile-mass-rescale \
  --stage relaxation \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512
conda run -n snia-rt python scripts/plot_arepo_dd_d6_binary_sequence.py \
  production_runs/arepo_dd_d6_binary_both_mesa_profile_pilot \
  --stage relaxation \
  --outdir results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_relaxation_frames
```

This wrote `snap_000.png` through `snap_010.png` under:

```text
results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_relaxation_frames/
```

<span style="color:red"><b>Science warning:</b> this maps both WDs from the same MESA profile and mass-rescales both, so it validates the mapper and snapshot visualization workflow only. It is not a source-consistent binary model.</span>

A secondary-specific MESA-profile mapper run was then completed with a pilot
`0.70 Msun` secondary:

```bash
./scripts/run_mesa_case.sh production_runs/mesa_secondary_co_wd_0p70_relax_from_0p96_pilot
conda run -n snia-rt python scripts/plot_mesa_profile.py \
  production_runs/mesa_secondary_co_wd_0p70_relax_from_0p96_pilot/LOGS/profile5.data \
  --out results/visualizations/mesa_secondary_co_wd_0p70_profile5.png
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_dd_d6_binary_stage.sh \
  --case-id PILOT_D6_THIN_HE_BOOS2021 \
  --source-table data/paper2/progenitor_grid_pilot_scaffold.csv \
  --run-dir production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot \
  --primary-profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --secondary-profile production_runs/mesa_secondary_co_wd_0p70_relax_from_0p96_pilot/LOGS/profile5.data \
  --allow-profile-mass-rescale \
  --stage relaxation \
  --np 1 \
  --n-cells-total 4096 \
  --n-background 512
conda run -n snia-rt python scripts/plot_arepo_dd_d6_binary_sequence.py \
  production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot \
  --stage relaxation \
  --outdir results/visualizations/arepo_dd_d6_binary_secondary_0p70_mesa_pilot_relaxation_frames
```

This wrote `snap_000.png` through `snap_010.png` under:

```text
results/visualizations/arepo_dd_d6_binary_secondary_0p70_mesa_pilot_relaxation_frames/
```

The final relaxation diagnostic summary is:

```text
time_code_units = 2.39503701847443
mass_total_msun = 1.7500000317103361
primary_mass_msun = 1.0467459944227344
secondary_mass_msun = 0.6994332641757433
separation_code_units = 24.03603169827311
```

The orbit-sequence diagnostic wrote:

```text
production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot/orbit_diagnostics_relaxation/relaxation_orbit_diagnostics.csv
production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot/orbit_diagnostics_relaxation/relaxation_orbit_summary.json
production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot/orbit_diagnostics_relaxation/relaxation_orbit_diagnostics.png
```

It reports separation changing from `23.811104917814646` to
`24.03603169827311` code units over the relaxation sequence, so this run is not
an inspiral.

<span style="color:red"><b>Science warning:</b> the secondary profile is a MESA mass-rescale pilot from an existing CO WD seed, not a ZAMS-to-WD/binary-history calculation. The primary is still mass-rescaled from `0.9589008445 Msun` to the source row's `1.05 Msun`. This run validates the two-profile mapper and AREPO snapshot workflow only; it is not a physical spiral-in/merger or explosion.</span>

## New AREPO Modules

| File | Role now | Publishable physics still needed |
|---|---|---|
| `external_codes/arepo-public/src/snia/snia_network.c` | Initializes the SN Ia module stack and calls per-step hooks. | EOS-coupled burning, energy release, timestep limiting, scalar conservation tests. |
| `external_codes/arepo-public/src/snia/snia_diagnostics.c` | Writes global gas, energy, momentum, volume, and scalar mass diagnostics. | Unit metadata and conservation tolerances for physical WD runs. |
| `external_codes/arepo-public/src/snia/snia_detonation.c` | Writes DD/D6 detonation trigger diagnostics and keeps `triggered=0`. | Cited hotspot/edge-lit trigger, imposed-test trigger, restart-stable event logging. |
| `external_codes/arepo-public/src/snia/snia_flame.c` | Writes SD fuel/IGE proxy diagnostics and keeps `flame_advanced=0`. | ADR or level-set deflagration, flame speed prescription, energy release. |
| `external_codes/arepo-public/src/snia/snia_turbulence.c` | Writes velocity-com and velocity-dispersion diagnostics. | Subgrid turbulent velocity/flame interaction model. |
| `external_codes/arepo-public/src/snia/snia_ddt.c` | Writes SD DDT diagnostics and keeps `ddt_triggered=0`. | Calibrated DDT criterion using flame/turbulence state and local density/composition. |
| `external_codes/arepo-public/src/snia/snia_tracer.c` | Records passive-scalar bridge status. | Lagrangian tracer histories and post-processing network export. |
| `external_codes/arepo-public/src/snia/snia_relaxation.c` | Damps internal velocities relative to each WD COM during relaxation; includes off-by-default orbital tangential damping hook. | Isolated-WD validation, conservative inspiral driver, Roche-lobe overflow/mass-transfer diagnostics, convergence tests. |
| `external_codes/arepo-public/src/snia/snia_eos.c` | Replaces ideal-gamma pressure for passive-scalar WD material with a Helmholtz-like degeneracy+thermal+radiation proxy. | Replace with full Timmes/Helmholtz table and validate against MESA profiles/one-zone thermodynamics. |
| `external_codes/arepo-public/src/snia/snia_atmosphere.c` | Controls numerical atmosphere cells, preserves advected WD material by passive scalar threshold, and clamps low-mass atmosphere mesh motion. | Resolution study of atmosphere density, scalar threshold, boundary treatment, and surface mass loss. |
| `external_codes/arepo-public/src/snia/snia_binary.c` | Writes Roche-lobe, separation, component COM, and fuel diagnostics. | Material-based component labels/tracers for true mass transfer after finite-volume fluxes. |
| `external_codes/arepo-public/src/snia/snia_burning.c` | Runtime reduced-alpha source-term path, off in relaxation runs. | Calibrated alpha-chain/table network, energy conservation tests, shock/burning limiter validation. |

## Isolated-WD Gate Added

The current physical path now starts with isolated-WD convergence before any
binary merger or detonation claim. New scripts:

```text
scripts/create_arepo_isolated_wd_ic.py
scripts/plot_arepo_isolated_wd_sequence.py
scripts/run_arepo_isolated_wd_relaxation.sh
scripts/diagnose_arepo_isolated_wd_sequence.py
```

## MESA-Profile Isolated-WD Gate

The isolated-WD IC path can now map a compact MESA `profile*.data` directly
into AREPO:

```bash
AREPO_SKIP_BUILD=1 ./scripts/run_arepo_isolated_wd_relaxation.sh \
  --profile production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data \
  --component primary \
  --run-dir production_runs/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate \
  --np 10 \
  --n-star 4096 \
  --n-background 4096 \
  --dynamical-times 0.01 \
  --spatial-sampling random_mass \
  --position-jitter-fraction 0.005 \
  --profile-pressure-model mesa_pressure
```

This produced:

```text
production_runs/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate/output_relaxation/snap_000.hdf5
...
production_runs/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate/output_relaxation/snap_010.hdf5
production_runs/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate/isolated_wd_sequence_summary.json
results/visualizations/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate_frames/snap_000.png
...
results/visualizations/arepo_isolated_mesa_compact_primary_profile6_4096_np10_gate_frames/snap_010.png
```

Summary:

```text
MESA profile mass = 0.958900844506 Msun
MESA profile radius = 6.20558356906e8 cm
AREPO snapshots = 11
fractional material mass change = -9.06466549e-5
max COM drift = 0.00223087770 code units
max density: 0.07505264078 -> 0.04378609164 code units
```

<span style="color:red"><b>Science warning:</b> this is a successful
MESA-profile-to-AREPO execution, but it does not yet pass the isolated-WD
science gate. The density/radius response over only `0.01` dynamical times means
the EOS/profile-pressure mapping and damping policy need calibration before this
can seed a binary merger or detonation model.</span>

Two shorter pressure/damping variants were then run with the same compact MESA
profile to isolate the failure mode:

```text
production_runs/arepo_isolated_mesa_compact_profile6_2048_np5_nonrelP_tau0p2_gate
production_runs/arepo_isolated_mesa_compact_profile6_2048_np5_mesaP_tau1p0_gate
```

Summary table:

```text
data/paper2/arepo_relaxation_gate_summary_20260504.csv
```

Key results:

```text
nonrel_degenerate pressure, tau=0.2:
  material mass fractional change = -3.65034303e-5
  max COM drift = 0.00506373180 code units
  max density = 0.05478418643 -> 0.04173599397 code units

MESA pressure/gamma closure, tau=1.0:
  material mass fractional change = -3.91669675e-5
  max COM drift = 0.00399150473 code units
  max density = 0.05478418643 -> 0.03893085326 code units
```

<span style="color:red"><b>Interpretation:</b> stronger damping improves COM
drift but does not solve the hydrostatic mismatch. The isolated WD expands in
all tested profile-pressure mappings, so the next physics task is EOS/profile
thermodynamic calibration rather than a longer binary merger run.</span>

## EOS Bridge And Mass-Shell Mapping Update: 2026-05-06

A MESA EOS bridge validator was added:

```text
scripts/validate_mesa_eos_bridge.py
```

It samples a MESA `profile*.data`, calls the local MESA EOS sample program, and
compares MESA profile pressure plus current AREPO proxy pressures against MESA
EOS pressure. For
`production_runs/mesa_dd_wd_he_shell_ignition/LOGS/profile6.data`, the profile
pressure agrees with MESA EOS in the sampled WD interior:

```text
summary = results/eos/mesa_profile6_eos_bridge_validation_summary.json
figure  = results/figures/mesa_profile6_eos_bridge_validation.png

profile_vs_mesa_eos median relative pressure error = 1.739e-4
profile_vs_mesa_eos p95 relative pressure error    = 2.247e-4
```

This validates the MESA profile thermodynamic bridge, not the runtime AREPO EOS.
The active `SNIA_HELMHOLTZ_EOS` branch is still a Helmholtz-like proxy, not a
full Timmes/MESA Helmholtz table implementation.

A radial mapping diagnostic was added:

```text
scripts/diagnose_arepo_profile_mapping.py
```

The IC creator now supports:

```text
--spatial-sampling fibonacci_mass_shells
```

This groups mass coordinates into shells and places cells angularly with a
Fibonacci pattern. It improves the t=0 median mapping relative to the earlier
global `fibonacci_shell` sampler:

```text
mass-shell 8192 snap000:
  density median relative error  = 6.227e-2
  pressure median relative error = 5.495e-2

global fibonacci 8192 snap000:
  density median relative error  = 6.265e-1
  pressure median relative error = 1.011
```

The evolved isolated-WD gate still fails:

```text
run = production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate
summary = production_runs/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate/isolated_wd_sequence_summary.json
visuals = results/visualizations/arepo_isolated_mesa_profile6_2048_np1_massshells_mesaP_0p02tdyn_gate_contact_sheet.png

max density = 0.03725 -> 0.02535 code units
rms material radius = 2.90996 -> 3.48675 code units
material mass fractional change = -4.811e-5
```

Therefore binary merger/explosion work remains behind the isolated-WD gate.

The latest binary merger negative validation is:

```text
run = production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral
orbit summary = production_runs/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral/orbit_diagnostics_merger/merger_orbit_summary.json
visuals = results/visualizations/arepo_dd_type6_coco_merger_negative_gate_4096_np1_inspiral_contact_sheet.png
```

It completed snapshots and diagnostics, but the artificial driver dominates the
angular-momentum evolution:

```text
separation fractional change = -7.574e-4
orbital angular momentum norm fractional change = -6.510e-1
```

This is useful negative validation of the binary execution path, not a physical
spiral-in/merger calculation.

For command-by-command usage, see:

```text
docs/beginner_runbook_mesa_arepo.md
```

## COMPAS Source-Traced MESA Bridge

The COMPAS row011845 primary and secondary were both evolved from their source
ZAMS masses and metallicity with MESA:

```text
primary:   M_ZAMS=4.17945617118 Msun, Z=0.013
secondary: M_ZAMS=4.02604363959 Msun, Z=0.013
```

Bridge table:

```text
data/paper2/mesa_wd_bridge_compas_type6_row011845.csv
data/paper2/mesa_wd_bridge_compas_type6_row011845.json
```

Current bridge status:

```text
primary COMPAS target = 0.822540586339 Msun
primary MESA partial stripped mass = 0.860735653757 Msun
primary latest radius = 1.90460843878 Rsun

secondary COMPAS target = 0.683722207538 Msun
secondary MESA partial stripped mass = 0.852505540423 Msun
secondary latest radius = 2.19215577772 Rsun
```

<span style="color:red"><b>Science warning:</b> these source-traced profiles are
real MESA stellar-evolution outputs, but they are not compact final WD profiles.
They must not be used as physical isolated-WD AREPO inputs until the final WD
settle/cooling bridge is solved or a documented, cited WD-structure bridge is
adopted.</span>

The latest compact-bridge retry used the saved source-traced restart photos:

```text
primary restart photo   = x950 -> co_wd_bridge_seed.mod
secondary restart photo = x650 -> co_wd_bridge_seed.mod
```

Bridge attempt summaries:

```text
data/paper2/mesa_compact_wd_bridge_attempt_status_20260504.csv
production_runs/mesa_compas_type6_row011845_primary_compact_bridge_from_x950/mesa_case_summary_bridge_seed.json
production_runs/mesa_compas_type6_row011845_secondary_compact_bridge_from_x650/mesa_case_summary_bridge_seed.json
```

Current status:

```text
primary   = bridge_seed_saved_settle_not_converged, no final.mod
secondary = bridge_seed_saved_settle_not_converged, no final.mod
```

The primary bridge attempt reached model `1010`, mass `0.86073565376 Msun`,
radius `1.90460927960 Rsun`; the secondary reached model `690`, mass
`0.85250554042 Msun`, radius `2.19211735210 Rsun`. These remain bridge seeds,
not compact WD structures.

Completed smoke run:

```text
production_runs/arepo_isolated_primary_compas_type6_0p823_5120_np10_completion_smoke_v11
```

This uses the local COMPAS Type-6 COCO representative primary
`M_WD=0.822540586339 Msun`, `1024` WD cells, `4096` numerical-atmosphere cells,
self-gravity, the EOS hook, and atmosphere control. It completed `snap_000` to
`snap_010`.

Summary:

```text
time_final_code_units = 5.558828353354014e-05
total_mass_msun_first = 0.8225405863390725
total_mass_msun_last  = 0.8225405863390725
fractional_material_mass_change = -4.39496437705735e-07
max_com_drift_code_units = 0.0006069424001437951
max_density_first_code_units = 0.005794387253402835
max_density_last_code_units  = 0.005796251032148903
```

<span style="color:red"><b>Science warning:</b> this is a completed micro smoke,
not a publishable isolated-WD convergence result. It verifies the AREPO
state-changing physics path and figure/diagnostic path after several failed
negative tests. The publishable gate is still multi-dynamical-time relaxation at
higher resolution, with MESA/WDEC profiles and a full Helmholtz EOS.</span>

## Source-Traced MESA Physical Run

New primary MESA case:

```text
production_runs/mesa_compas_type6_row011845_primary_zams4p179_Z0p013_make_co_wd
```

This case was generated from the local COMPAS/Paper-II candidate row
`COMPAS_TYPE6_COCO_row011845_M0p823_0p684_Z0p013`, using the source-row
primary ZAMS mass `4.17945617118 Msun` and metallicity `Z=0.013`.

The run physically reached the TP-AGB CO-core stage and produced:

```text
zams.mod
end_he_core_burn.mod
co_core.mod
```

Summary from `mesa_case_summary_partial.json`:

```text
star_age_yr = 1.7469943117992684e8
partial_stripped_star_mass_msun = 0.8607356537565443
co_core_mass_msun = 0.8505043961373719
center_X_C12 = 0.3393549170533135
center_X_O16 = 0.6435515602097333
log10_Tc = 8.171084610679186
log10_rhoc = 6.89182762190349
```

<span style="color:red"><b>Science warning:</b> this is a real MESA stellar
evolution run, but not a final settled WD. Envelope removal entered a long
solver-retry plateau and was stopped deliberately. The result is valid as a
source-traced CO-core/provenance run; it must not yet be used as a final
AREPO-ready WD profile without either finishing/tuning the MESA settle stage or
adopting a documented WD-structure bridge.</span>

## Current Visual Outputs

```text
results/figures/mesa_compas_type6_primary_zams4p179_Z0p013_history_partial.png
results/figures/mesa_compas_type6_primary_zams4p179_Z0p013_co_core_profile.png
results/figures/mesa_compas_type6_primary_zams4p179_Z0p013_envelope_removal_profile_partial.png
results/visualizations/arepo_snia_dd_d6_dev_smoke.png
results/visualizations/arepo_snia_sd_ddt_dev_smoke.png
results/visualizations/arepo_dd_d6_binary_pilot_ic.png
results/visualizations/arepo_dd_d6_binary_pilot_relaxation_snap010.png
results/visualizations/mesa_dd_he_shell_profile6_for_arepo_mapping.png
results/visualizations/arepo_dd_d6_binary_mesa_profile_pilot_ic.png
results/visualizations/arepo_dd_d6_binary_mesa_profile_pilot_relaxation_snap010.png
results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_ic.png
results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_relaxation_frames/snap_000.png
...
results/visualizations/arepo_dd_d6_binary_both_mesa_profile_pilot_relaxation_frames/snap_010.png
results/visualizations/mesa_secondary_co_wd_0p70_profile5.png
results/visualizations/arepo_dd_d6_binary_secondary_0p70_mesa_pilot_ic.png
results/visualizations/arepo_dd_d6_binary_secondary_0p70_mesa_pilot_relaxation_frames/snap_000.png
...
results/visualizations/arepo_dd_d6_binary_secondary_0p70_mesa_pilot_relaxation_frames/snap_010.png
production_runs/arepo_dd_d6_binary_secondary_0p70_mesa_pilot/orbit_diagnostics_relaxation/relaxation_orbit_diagnostics.png
results/visualizations/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate_frames/snap_000.png
...
results/visualizations/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate_frames/snap_010.png
production_runs/arepo_dd_type6_coco_median_16384_np10_laneemden_random_relax_candidate/orbit_diagnostics_relaxation/relaxation_orbit_diagnostics.png
results/visualizations/arepo_isolated_primary_compas_type6_0p823_5120_np10_completion_smoke_v11_frames/snap_000.png
...
results/visualizations/arepo_isolated_primary_compas_type6_0p823_5120_np10_completion_smoke_v11_frames/snap_010.png
results/visualizations/arepo_isolated_mesa_compact_profile6_2048_np5_nonrelP_tau0p2_gate_frames/snap_000.png
...
results/visualizations/arepo_isolated_mesa_compact_profile6_2048_np5_nonrelP_tau0p2_gate_frames/snap_010.png
results/visualizations/arepo_isolated_mesa_compact_profile6_2048_np5_mesaP_tau1p0_gate_frames/snap_000.png
...
results/visualizations/arepo_isolated_mesa_compact_profile6_2048_np5_mesaP_tau1p0_gate_frames/snap_010.png
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_frames/snap_000.png
...
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_frames/snap_010.png
results/visualizations/arepo_dd_type6_coco_bridge_gate_8192_np1_short_relax_contact_sheet.png
```

These figures are smoke-test visualizations: hydro fields plus eight passive scalar channels (`He4`, `C12`, `O16`, `Si28`, `Ca40`, `TiCrProxy`, `StableIGE`, `Ni56`). The DD/D6 and SD-DDT figures use the same diagnostic IC, so their hydro panels are intentionally identical; the branch distinction is in the compiled module set and diagnostic output files.

## Next Implementation Milestones

1. Full Helmholtz EOS wrapper and one-zone EOS comparison tests against MESA or Timmes tables.
2. Isolated single-WD relaxation tests for each mass/composition before binary assembly.
3. Replace analytic DD/D6 bootstrap ICs with source-consistent MESA/WDEC-derived WD structures and run a relaxation/convergence sequence before any inspiral or detonation claim.
4. Add binary-orbit diagnostics for DD/D6 runs: component centers, separation,
   orbital angular momentum, Roche-lobe overflow indicators, mass transfer
   through L1, and energy conservation versus time.
5. Add a controlled inspiral/merger driver only after item 4 is validated:
    either start at Roche contact with enough resolution for unstable mass
    transfer, or add an explicitly documented angular-momentum-loss prescription
    for development runs.
6. One-zone reduced burning or tabulated burning test with scalar and energy conservation.
7. DD/D6 planar detonation benchmark before any WD-scale detonation claim.
8. D6 He-shell/core trigger metadata from MESA-mapped WD structures.
9. SD ADR/level-set flame and turbulence diagnostic validation.
10. SD DDT criterion only after the flame/turbulence state variables are trustworthy.
11. Homologous ejecta export to ARTIS/TARDIS, then sncosmo SALT fits for `M_B`, `x1`, and `c`.
