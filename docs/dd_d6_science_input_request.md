# DD/D6 Science Input Request

Use this file when selecting real CO+CO WD merger candidates for Paper II. The
goal is to replace pilot rows with source-consistent rows that can drive:

```text
COMPAS/Paper I row -> MESA/WDEC WD structures -> AREPO DD/D6 merger -> RT -> SALT3
```

## Best Input To Provide

The most useful input is a **WD-WD state at double-WD formation and/or at
Roche-contact onset**, plus the original ZAMS provenance.

If only one level can be provided, prefer the **WD-WD/contact state**. AREPO
does not need the full main-sequence history directly; it needs the actual WD
objects and orbit it will evolve. MESA needs enough history/provenance to build
the WD thermal/composition profiles without arbitrary choices.

For local COMPAS `sn_type=6` CO+CO WD merger rows, use:

```bash
conda run -n snia-rt python scripts/compas_type6_to_paper2_grid.py \
  /Users/brett/Desktop/COMPAS/compas_python_utils/preprocessing/sn_type_6_data.csv
```

This writes candidate rows under `data/paper2/` and keeps the COMPAS checkout
read-only. The converter preserves the ZAMS and double-WD formation periods,
but also derives an AREPO contact-start period/separation for candidate
hydrodynamic runs. Those contact values use Nauenberg WD radii plus Eggleton
Roche-lobe geometry and must be replaced or validated with MESA/WDEC structures
for publication.

## Required Per DD/D6 Candidate

```text
case_id
channel = DD or D6
statistical_weight
source_reference or Paper I table/subhalo/SSP/bin id
metallicity_z
delay_time_gyr
primary_wd_mass_msun
secondary_wd_mass_msun
primary_wd_type = CO/HeCO/ONe
secondary_wd_type = CO/HeCO/He/ONe
dwd_formation_time_gyr
coalescence_time_gyr or gw_time_from_dwd_gyr
orbital_period_at_dwd_formation_days or separation_at_dwd_formation_au
eccentricity_at_dwd_formation
```

## Strongly Preferred For AREPO Contact/Merger

```text
orbital_period_at_contact_s or separation_at_contact_cm
which_star_fills_roche_lobe
spin_state = synchronized / nonrotating / specified angular velocity
primary_cooling_age_at_contact_gyr
secondary_cooling_age_at_contact_gyr
primary_effective_temperature_at_contact_k
secondary_effective_temperature_at_contact_k
primary_central_temperature_k
secondary_central_temperature_k
primary_central_density_g_cm3
secondary_central_density_g_cm3
primary_c12_o16_profile_source
secondary_c12_o16_profile_source
```

## D6 / Double-Detonation-Specific Fields

```text
primary_helium_shell_mass_msun
primary_helium_shell_base_density_g_cm3
primary_helium_shell_base_temperature_k
secondary_surface_he_mass_msun
mass_transfer_rate_at_contact_msun_per_year
helium_accretion_history_source
detonation_trigger_reference_or_policy
```

## ZAMS / COMPAS Provenance Fields

```text
compas_seed
zams_mass_1_msun
zams_mass_2_msun
initial_binary_period_days
initial_semi_major_axis_au
initial_eccentricity
initial_mass_ratio
common_envelope_events
mass_transfer_history_summary
final_stellar_type_1
final_stellar_type_2
```

## Minimal Acceptable Science Row

For a first real non-pilot DD/D6 AREPO contact run, the minimum is:

```text
case_id
channel
statistical_weight
metallicity_z
delay_time_gyr
primary_wd_mass_msun
secondary_wd_mass_msun
primary_wd_type
secondary_wd_type
primary_cooling_age_at_contact_gyr
secondary_cooling_age_at_contact_gyr
separation_at_contact_cm or orbital_period_at_contact_s
helium_shell_mass_msun if D6
source_reference/provenance
```

If contact separation is missing, we can compute a documented Roche-contact
estimate with Eggleton's formula for a candidate run. For a final paper run, the
contact setup should be traceable to the selected BPS/MESA evolution path.

## Why This Is Needed

COMPAS gives population-consistent binary outcomes and weights. MESA/WDEC gives
the detailed WD thermodynamic and chemical structure. AREPO then evolves only
the final dynamical phase: relaxation, contact, unstable mass transfer, merger,
and possible detonation.

Do not ask AREPO to evolve the main-sequence binary for Myr-Gyr. That belongs to
stellar evolution/BPS. AREPO is for the seconds-to-hours dynamical phase once
the WD-WD system is already compact.
