# Source Strategy For Non-Arbitrary Progenitor Values

Paper II must not choose progenitor values because they are convenient. Every
source-table field needs a provenance path.

## Preferred Provenance Chain

1. **Paper I SSP/BPS row**: channel, SSP age, metallicity, event delay time,
   binary parameters, and statistical weight.
2. **COMPAS row**: ZAMS masses, initial separation/period, mass ratio,
   DWD/compact-object formation time, final WD masses, and gravitational-wave
   coalescence time.
3. **MESA bridge**: CO-WD mass, C/O profile, central temperature/density,
   cooling state, accretion history, and ignition conditions.
4. **Hydro/RT literature anchor**: explosion/ejecta parameters only when a full
   local hydro/RT run is not yet available; must be labelled as a literature
   anchor, not as our self-consistent result.

## Field-Level Rules

| Field | Preferred source | If missing |
| --- | --- | --- |
| `metallicity_z` | Paper I SSP metallicity or COMPAS initial metallicity | Use the parent SSP metallicity bin; cite IllustrisTNG/Paper I selection. |
| `zams_mass_1_msun`, `zams_mass_2_msun` | COMPAS initial binary row | Reconstruct only from a documented BPS/SSP sample; do not infer from final WD mass alone for science rows. |
| `delay_time_gyr` | Paper I/BPS event delay time | For DD candidates, use COMPAS `Time + Coalescence_Time`; for SD, use binary age at ignition/accretion endpoint. |
| `cooling_time_gyr` | MESA WD cooling segment tied to binary delay | Compute as the time between CO-WD formation and accretion/merger trigger. If unavailable, row remains candidate-only. |
| `primary_wd_mass_msun`, `secondary_wd_mass_msun` | COMPAS final DWD masses, then refined by MESA WD models | Use IFMR only as a documented bridge for candidate design, not as final paper-grade structure. |
| `helium_shell_mass_msun` | MESA helium-accretion model or published double-detonation grid | Leave blank for DD candidates until a D6/double-detonation selection is justified. |
| `accretion_rate_msun_per_year` | COMPAS mass-transfer phase plus MESA accretion experiment | Use literature steady-burning anchors only for pilot/literature-anchor rows. |
| `statistical_weight` | Paper I SSP/BPS event weight | Raw COMPAS Monte Carlo rows may carry unit raw weight only as candidates; science rows need normalized weights. |

## Candidate Versus Science Rows

COMPAS-derived rows first go to:

```text
data/paper2/progenitor_grid_compas_candidates*.csv
```

These rows are labelled:

```text
candidate_requires_selection_not_science_input
```

They become science rows only after:

1. selection is tied to Paper I or a documented BPS distribution,
2. weights are normalized,
3. missing cooling/structure fields are supplied by MESA or explicitly marked as
   literature-anchor quantities,
4. `data_status` is changed to `science_input`.

The actual MESA/hydro/RT grid generator reads only:

```text
data/paper2/progenitor_grid.csv
```

and rejects non-science rows by design.

## Citation Anchors

- Paper I local manuscript: `/Users/brett/Downloads/SN_Ia_Population_Machine_I.pdf`.
- IllustrisTNG data context: Nelson et al. 2019, doi:10.1186/s40668-019-0028-x.
- COMPAS: Team COMPAS et al. 2022, doi:10.21105/joss.03838.
- BPASS: Eldridge et al. 2017, doi:10.1017/pasa.2017.51.
- MIST/MESA stellar tracks: Choi et al. 2016, doi:10.3847/0004-637X/823/2/102.
- IFMR bridge: Cummings et al. 2018, doi:10.3847/1538-4357/aadfd6; Cunningham et al. 2024, doi:10.1093/mnras/stad3275.
- SD accreting WD stability: Wolf et al. 2013, doi:10.1088/0004-637X/777/2/136.
- DD violent mergers: Pakmor et al. 2012, doi:10.1088/2041-8205/747/1/L10.
- D6 observational motivation: Shen et al. 2018, doi:10.3847/1538-4357/aad55b.
- Double-detonation grids: Polin et al. 2019, doi:10.3847/1538-4357/aafb6a; Boos et al. 2021, doi:10.3847/1538-4357/ac07a2.
- Non-LTE RT: Shen et al. 2021, doi:10.3847/2041-8213/abe69b.
- ARTIS: Kromer & Sim 2009, doi:10.1111/j.1365-2966.2009.15256.x.
- SALT3: Kenworthy et al. 2021, doi:10.3847/1538-4357/ac30d8.

## DD/D6 Analytic Bootstrap Policy

The DD/D6 binary IC generator may fill missing pilot-only geometry from cited
closed-form relations:

- Roche-lobe contact: Eggleton 1983, doi:10.1086/160960.
- Cold WD radius scale: Nauenberg 1972, doi:10.1086/151802.

Rows generated this way must keep:

```text
analytic_bootstrap_do_not_publish
```

in their manifest. Science rows must instead provide WD structure from MESA,
WDEC, or a cited hydrodynamic setup paper. A bootstrap separation or radius is
acceptable for testing the file format and AREPO workflow; it is not acceptable
as a final Paper II progenitor model.

## SD Long-Accretion Policy

Do not attempt to simulate the entire SD accretion lifetime in AREPO. Stable
hydrogen/helium accretion, WD cooling, crystallization, and ignition-density
selection are secular stellar-evolution problems. Use MESA, Paper I/BPS
metadata, and cited accreting-WD prescriptions to define the final WD state.
AREPO/FLASH enters only for dynamical phases: final pre-ignition 3D flow,
deflagration/DDT/explosion, or ejecta-companion impact.
