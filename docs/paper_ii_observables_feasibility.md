# Paper II Feasibility: SN Ia Observables from Population Machine Progenitors

## Scientific Goal

Paper I builds a cosmology plus BPS framework that tags synthetic SNe Ia with
channel, delay time, host properties, and progenitor properties. Paper II can
extend that catalogue by assigning physically motivated observables:

- peak absolute B-band magnitude, `M_B`
- SALT stretch, `x1`
- SALT color, `c`
- optionally ejecta velocity, nickel mass, and standardized Hubble residual

The clean scientific thesis is:

> The redshift evolution of SD/DD and near/sub-Chandrasekhar progenitor mixtures
> induces evolution in the distribution of `M_B`, `x1`, and `c`, producing a
> measurable distance-modulus bias if a single global standardization is assumed.

## What AREPO Did So Far

There are now two AREPO verification runs in this workspace.

The baseline run is a 1D Sod shock-tube smoke test. It checks that the public
AREPO binary can read HDF5 initial conditions, evolve a hydrodynamic Riemann
problem, and write snapshots.

The SN Ia development run builds `ArepoSNIaDev` with `SNIA_NETWORK`,
`SNIA_DETONATION_TRIGGER`, `SNIA_TRACER_EXPORT`, and `PASSIVE_SCALARS=8`. It
verifies compile-time hooks, end-of-step source-term wiring, passive-scalar
snapshot output, and hydro-to-RT table export. It is also not yet a physical SN
Ia simulation.

Visualization:

```text
results/visualizations/arepo_shocktube_profiles.png
results/visualizations/arepo_shocktube_density_zoom.png
results/visualizations/arepo_snia_dev_smoke.png
```

## Can We Put SN Ia Physics into AREPO?

Yes, in principle. AREPO is a moving-mesh hydrodynamics code, so it can host a
SN Ia branch. This is the route taken by several DD merger studies using
non-public or heavily modified AREPO branches.

The public AREPO tree installed here is only the base. It currently has:

- hydrodynamics and moving Voronoi mesh
- self-gravity options
- HDF5 snapshot input/output
- passive scalar infrastructure
- MHD infrastructure
- local SN Ia scaffold hooks added under `src/snia/`

It does not currently have:

- degenerate WD equation of state suitable for production SN Ia burning
- alpha-chain or larger nuclear reaction network
- physical energy release coupled to composition evolution
- validated detonation trigger and front/capturing logic
- DDT or double-detonation ignition logic
- Lagrangian tracer particles or detailed isotope export
- homologous-ejecta post-processing products for radiative transfer

Therefore, an AREPO-only explosion branch is possible but is a research-code
development project, not a configuration switch.

## Does AREPO Remove the Need for FLASH?

Possibly yes.

FLASH/CASTRO are not conceptually required if AREPO is extended and validated for
the explosion physics. AREPO can be the sole hydrodynamics/explosion code for the
paper if it includes and validates:

- WD initial condition mapping
- self-gravity and orbital dynamics
- burning network and energy release
- detonation/ignition criteria
- convergence tests against known explosion models
- isotope/tracer output

For DD/D6, AREPO is especially attractive because moving mesh is well matched to
inspiral, tidal disruption, mass transfer, and asymmetric ejecta.

For SD, AREPO should not replace MESA for secular accretion over millions of
years. It is appropriate for dynamical phases: Roche-lobe overflow snapshots,
common-envelope-like flow, ejecta-companion interaction, and the explosion
hydrodynamics after MESA provides the WD profile.

## Does AREPO Remove the Need for SEDONA/ARTIS/SuperNu?

No.

Hydrodynamics plus nucleosynthesis gives ejecta mass, velocity, temperature, and
isotopes. It does not directly give `M_B`, `x1`, or `c`. Those observables require
radiative transfer and light-curve fitting.

To remove SEDONA/ARTIS/SuperNu, we would have to implement a time-dependent
multi-frequency radiative-transfer solver inside AREPO. That is technically
possible but scientifically inefficient and would create a second major code
validation project. For a publishable Paper II, the clean approach is:

```text
MESA/BPS progenitors -> AREPO or FLASH explosion/ejecta -> RT light curves -> sncosmo SALT fits
```

TARDIS is already installed and is useful for spectra and line diagnostics, but
it is not the best sole tool for production multi-band SN Ia light curves. A
time-dependent ejecta RT code remains the right tool for `M_B`, `x1`, and `c`.

## Recommended Paper II Strategy

### Phase 1: Population-to-Observable Emulator

Build a literature-calibrated or simulation-grid-calibrated mapping from
progenitor properties to observables:

- SD inputs: delay time/cooling age, metallicity, WD mass, accretion regime,
  ignition density proxy
- DD/D6 inputs: primary WD mass, secondary WD mass, mass ratio, helium shell
  mass, metallicity, delay time
- outputs: `M_B`, `x1`, `c`, `M_Ni56`, ejecta mass, velocity proxy

This can directly extend Paper I's catalogue and quantify the predicted
redshift-dependent distributions and Hubble residual drift.

### Phase 2: Anchor Models with Full Simulations

Run a deliberately small grid of full physical models to anchor the emulator:

- MESA SD Chandrasekhar ignition models
- MESA sub-Chandrasekhar helium-shell ignition models
- selected AREPO/FLASH hydro explosions
- RT light curves
- sncosmo SALT2/SALT3 fitting

The paper does not need to simulate every one of Paper I's synthetic events.
It needs a defensible mapping from population parameters to observables, with
uncertainty propagation.

### Phase 3: Full AREPO SN Ia Branch, If Needed

If the project goal is also to publish a methods/code paper, develop a SN Ia
AREPO branch:

1. Add isotope/passive scalar layout.
2. Add WD EOS or integrate a tabulated Helmholtz-like EOS.
3. Add a small alpha-chain network and burning timestep limiter.
4. Couple nuclear energy release to hydro.
5. Add detonation trigger logic for He shell and CO core.
6. Add tracer particles or high-order scalar isotope export.
7. Validate with 1D/2D test problems and published benchmarks.
8. Export homologous ejecta to RT.

This is publishable, but it is a larger methods project than Paper II needs.

## Bottom Line

- AREPO can replace FLASH only if we build and validate the missing SN Ia
  explosion physics.
- AREPO cannot replace radiative transfer for `M_B`, `x1`, and `c`.
- For a realistic follow-up to Paper I, the strongest route is a hybrid:
  population catalogue plus MESA-informed progenitor grids plus hydro/RT
  calibration plus sncosmo fitting.
