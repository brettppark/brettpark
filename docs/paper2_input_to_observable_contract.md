# Paper II Input-to-Observable Contract

<span style="color:red"><b>Do not publish pilot rows.</b> The only science entry point is `data/paper2/progenitor_grid.csv` with `data_status=science_input` and explicit provenance.</span>

## User-Provided Input

Place the selected weighted progenitor grid here:

```text
data/paper2/progenitor_grid.csv
```

Use `data/paper2/progenitor_grid_template.csv` as the schema. Every science row must include:

- `case_id`
- `data_status=science_input`
- `channel`: `SD`, `DD`, or `D6`
- `source_model`
- `source_reference`
- `selection_rationale`
- `statistical_weight`
- `metallicity_z`
- `delay_time_gyr`

Channel-specific requirements:

- SD: include initial stellar condition and accretion/cooling provenance, especially `zams_mass_msun`, `cooling_time_gyr`, `companion`, and `accretion_rate_msun_per_year` when known.
- DD: include `primary_wd_mass_msun`, `secondary_wd_mass_msun`, delay/coalescence time, metallicity, and cooling provenance.
- D6: include the DD fields plus `helium_shell_mass_msun` and any orbit/secondary metadata available from BPS or literature selection.

The row can cite Paper I table IDs, COMPAS seeds, SSP bin IDs, MIST/IFMR/MESA bridge values, or literature DOI/ADS entries. Missing quantities should be blank only if the downstream stage can derive them with a recorded method.

## Commands After Input Arrives

Generate the case directories:

```bash
./scripts/make_paper2_grid.py
```

Run a MESA stage:

```bash
./scripts/run_mesa_case.sh production_runs/paper2_grid/<case_id>/stage1_mesa_make_co_wd
```

Plot a MESA history:

```bash
./scripts/plot_mesa_history.py production_runs/paper2_grid/<case_id>/stage1_mesa_make_co_wd
```

Run current AREPO SN Ia development smoke:

```bash
./scripts/run_arepo_snia_dev_smoke.sh
```

Convert an AREPO snapshot to the neutral ejecta bridge format:

```bash
conda run -n snia-rt python scripts/arepo_snapshot_to_ejecta_table.py \
  production_runs/<hydro_case>/output/<snapshot>.hdf5 \
  --case-id <case_id> \
  --outdir production_runs/<hydro_case>/ejecta_export
```

Fit RT synthetic photometry after ARTIS/TARDIS output exists:

```bash
conda run -n snia-rt python scripts/fit_sncosmo_lightcurve.py \
  production_runs/<rt_case>/synthetic_photometry.csv \
  --model salt3
```

## Expected Paper Products

For each `case_id`, the final paper-grade folder should contain:

- source row and provenance metadata;
- MESA history/profile figures;
- hydro snapshot figures and ejecta summary;
- RT spectra/light curves;
- SALT fit output: `M_B`, `x1`, `c`, covariance or fit quality;
- log of code versions, compile options, and all assumptions.

## Current Blocking Physics

The folder is now ready for source-table-driven case creation and smoke-tested code execution, but the AREPO branch still needs physical EOS, burning, detonation, tracer, and RT conversion validation before real observable production.

