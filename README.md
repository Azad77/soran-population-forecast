# Soran Population Forecast

Forecasting pipeline for annual population in Soran, Iraq (2015–2024 observed,
2025–2030 forecast), combining a monthly climate record with three candidate
forecasting models — polynomial trend, MLP, and ARIMAX — evaluated with
expanding-window (rolling-origin) cross-validation and compared using a
Diebold–Mariano test. Final forecasts are reported with 95% bootstrap
prediction intervals.

## Method summary

- **Validation:** expanding-window (leave-future-out) cross-validation on the
  real annual observations only — no single static train/test split.
- **Interpolation:** the MLP is trained on a cubic-spline monthly upsample of
  the population series, fit strictly *inside* each cross-validation fold, so
  no information from a held-out year leaks into the interpolation.
- **Exogenous predictors:** off by default (pure univariate trend
  extrapolation). Any exogenous series supplied must be genuinely
  independently measured — not a mathematical transform of population itself.
- **Model naming:** the time-series model has `seasonal_order=(0,0,0,0)`
  (no sub-annual frequency in an annual series), so it is correctly reported
  as ARIMAX rather than SARIMAX.
- **Uncertainty:** final forecast intervals are model-specific bootstrap
  prediction intervals built from rolling-origin residuals (polynomial, MLP),
  or the model's own analytic confidence interval (ARIMAX).

## Repository layout

```
population_forecast.py   # pipeline: data loading, models, CV, forecasting, plots
requirements.txt
data/                     # input workbooks (not included; see below)
outputs/                  # generated tables, figures, and a run manifest
```

## Input data

Two Excel workbooks are required, each with a single header row:

1. **Monthly climate workbook** (`--climate-xlsx`) — one row per station-month
   with (at minimum) columns `years`, `month`, `c.avg`, `precipition.depth`,
   `humidity.avg`. Optional columns `c.max`, `c.min`, `humidity.max`,
   `humidity.min`, `pan.evap.mm`, and `soil.c.<depth>cm_1` (depth in
   `{5,10,20,50,100}`) are picked up automatically when present.
2. **Population workbook** (`--population-xlsx`) — one row per year-month
   with columns `years`, `month`, and `population.soran`. The December
   (year-end) reading is used as each year's annual value.

Both files are expected to share one data-entry quirk that the loader
corrects automatically: each year's December row is written as the first row
of the *following* year's block. See `fix_december_labels()` in
`population_forecast.py`.

## Usage

```bash
pip install -r requirements.txt

python population_forecast.py \
    --climate-xlsx data/climate_monthly.xlsx \
    --population-xlsx data/population_annual.xlsx \
    --out-dir outputs
```

To include independently measured exogenous predictors (e.g. real GDP or
housing-stock series), pass `--exog-mode independent` and supply
`gdp_independent` / `housing_independent` series (indexed by year) to
`run_pipeline()` if calling the module directly rather than via the CLI.

## Outputs

Written to `<out-dir>/`:

- `tables/climate_monthly_clean.csv` — cleaned monthly meteorological series
- `tables/climate_population_correlation.csv` — detrended, lagged Spearman
  correlation between population growth and climate variables
- `tables/rolling_origin_forecast_errors.csv` — per-fold forecast errors for
  every model
- `tables/rolling_origin_error_summary.csv` — MAPE / RMSE / R² per model
- `tables/diebold_mariano_results.csv` — pairwise model-comparison tests
- `tables/final_forecast.csv` — 2025–2030 point forecasts with 95% prediction
  intervals
- `figures/final_forecast.png`, `figures/correlation_heatmap.png`
- `requirements.txt` (exact installed package versions) and
  `run_manifest.json` (random seed and run configuration)

## Reproducibility

A fixed random seed (`SEED = 42`, in `population_forecast.py`) is used for the
MLP, bootstrap resampling, and any other stochastic step. Re-running the
pipeline against the same input workbooks reproduces identical output files.

## License

Add a license file appropriate for your project before publishing.
