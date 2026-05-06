# Type Ia Simulation Pipeline

## What This Folder Can Run Now

This workspace now has two layers.

The production layer contains locally installed external codes:

- MESA for WD progenitor evolution, cooling, accretion, and ignition setup
- public AREPO for moving-mesh hydrodynamics tests and future SN Ia extensions
- TARDIS and sncosmo in the `snia-rt` conda environment

The exploratory layer in `src/snia_sim` is a fast, one-dimensional toy scaffold.
It does four practical things:

1. Creates reproducible SD and DD progenitor initial conditions.
2. Stores density, temperature, composition, and metadata in HDF5.
3. Uses a calibrated toy explosion/radiation model to estimate `M_B`, `x1`, and `c`.
4. Runs parameter sweeps to identify which cases deserve expensive simulations.

The model is intentionally transparent. It preserves the expected qualitative
mapping:

- larger DD primary WD mass -> more `56Ni`, brighter `M_B`, broader `x1`
- thicker DD helium shell -> redder `c` through high-velocity Ti/Cr-like ashes
- longer SD cooling time -> higher ignition density, more stable IGE, less `56Ni`
- higher SD metallicity -> more neutron excess, dimmer and redder events

## Recommended Production Stack

### Single Degenerate

Use MESA for secular evolution. It should carry the WD through cooling, stable or
unstable accretion, simmering setup, and final 1D profiles. Do not try to evolve
millions of years of SD accretion in AREPO or FLASH.

Use a multidimensional hydro code only for the dynamical stages:

- pre-ignition convection: MAESTROeX or a low-Mach method
- explosion hydrodynamics: FLASH, CASTRO, or AREPO
- ejecta-companion interaction: AREPO, FLASH, or CASTRO

AREPO can be useful for SD if the problem is genuinely dynamical, such as Roche
lobe overflow, common-envelope-like flow, impact of ejecta on the companion, or
asymmetric circumstellar material. For the secular accretion history, MESA is the
right tool.

### Double Degenerate

Use AREPO for DD mergers and D6-like dynamically driven double detonations. Its
moving Voronoi mesh is well matched to inspiral, tidal disruption, angular
momentum transport, and highly asymmetric ejecta.

The public AREPO tree installed here is a working hydro code, but not yet a
complete SN Ia DD/D6 production code. Research-grade DD explosions require
additional physics modules: self-gravity choices, degenerate equation of state,
nuclear burning networks, detonation criteria, refinement strategy, tracer or
composition fields, and homologous-ejecta export.

For sub-Chandrasekhar double detonations where you only need the explosion after
the binary state is specified, FLASH or CASTRO can also be appropriate.

### Radiative Transfer and SALT Parameters

Hydrodynamics gives density, velocity, temperature, and isotope distributions. It
does not directly give `M_B`, `x1`, or `c`.

For observables:

- SEDONA, ARTIS, or SuperNu: time-dependent light curves and spectra
- CMFGEN: detailed 1D non-LTE spectra
- TARDIS: rapid spectral modeling and line-identification experiments
- sncosmo with SALT2/SALT3: fit synthetic photometry to recover `x0`, `x1`, `c`

For serious color predictions, prefer non-LTE or at least a carefully validated
non-LTE proxy. LTE models commonly over-redden SN Ia spectra because they
mis-handle the Fe III/Fe II ionization balance.

## Suggested Promotion Path

1. Build MESA WD grids in `production_runs/mesa_make_co_wd_grid_seed`.
2. For SD, evolve `production_runs/mesa_sd_wd_c_core_ignition` over cooling,
   accretion rate, and metallicity grids until carbon ignition.
3. For DD/sub-Chandrasekhar, evolve
   `production_runs/mesa_dd_wd_he_shell_ignition` over WD mass, cooling age, and
   helium-shell mass grids.
4. Map final MESA profiles into the hydro code with a documented unit system and
   composition mapping.
5. Extend/replace public AREPO with the SN Ia physics modules needed for
   detonation and nucleosynthesis, or use FLASH/CASTRO for the explosion phase.
6. Evolve ejecta to homologous expansion.
7. Convert ejecta to the input format required by SEDONA/ARTIS/SuperNu, or use
   TARDIS for fast spectral experiments.
8. Fit synthetic photometry with sncosmo SALT2/SALT3 to recover `x0`, `x1`, and
   `c`; convert `x0` or peak synthetic photometry into `M_B`.
