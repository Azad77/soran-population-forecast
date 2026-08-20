# Soran Population Forecast

Reproducibility package for *Population Dynamics and Climate in Soran, Iraq*.
Everything in `outputs/` is regenerated from the two raw workbooks in `data/` by
a single notebook. Nothing is hand-edited.

```
Soran_Population_Forecast.ipynb   run this
requirements.txt                  exact package versions used
data/
├── population_annual.xlsx        official annual series, 2010–2024
├── climate_monthly_raw.xlsx      Soran station log, 2015–2024 (120 station-months)
└── legacy_population_monthly.xlsx  earlier series, kept only for reconciliation
outputs/
├── run_manifest.json             seed, package versions, SHA-256 of each input
├── tables/                       17 CSV files
└── figures/                      7 PNG files at 300 dpi
```

## How to run

```bash
pip install -r requirements.txt
jupyter lab Soran_Population_Forecast.ipynb   # Run All
```

The notebook resolves `data/` relative to its own folder. The seed is fixed at 42,
and a re-run reproduces every output file byte-for-byte.

## What this release changes

This supersedes the previous release, which was built on an undocumented monthly
population series (2015–2024, 104,000 → 201,403). The population input is now
the official annual series for the Soran district centre — **15 observations,
2010–2024** (56,761 → 91,589) — from the Statistics Directorate of the Soran
Independent Administration. Section 1.2 of the notebook reconciles the two series;
the earlier one is a within-year linear interpolation at roughly double the
district-centre level.

Three findings follow from the corrected input:

1. **2010–2023 are inter-censal estimates, not counts.** They are reproduced to
   within **0.10 %** by a single linear declining-growth rule. A naive random walk
   with drift forecasts them out-of-sample at **0.087 % MAPE**, better than any
   fitted specification tested here. Those accuracies measure the smoothness of the
   administrative series, not forecasting skill.
2. **2024 is a level shift.** The 2023 → 2024 step is +18.4 % against a 2.0 %/yr
   trend — 178 residual standard deviations — because the 2024 census round
   replaced an estimate. No specification anticipates it; every one misses 2024 by
   at least 11,600 persons.
3. **The climate–demography association cannot be tested on these data.** The
   pre-2024 values are administratively smoothed, so their year-on-year variation
   carries no independent demographic signal. Of twelve variable–lag combinations
   (four climate variables at lags 0, 1 and 2), one reaches nominal significance
   (humidity at lag 0, ρ = 0.78, p = 0.013) and does not survive multiplicity
   correction (Holm–Bonferroni p = 0.150).

## Method

- **Validation** — expanding-window (rolling-origin) cross-validation on the
  fifteen annual observations: train 2010–(2009+k), forecast 2010+k, for
  k = 8,…,14, giving seven one-step-ahead errors over test years 2018–2024.
  Successive folds share most of their training data, so the errors are not
  treated as independent; fold-level errors are reported individually in
  `fold_level_absolute_errors.csv`.
- **Benchmarks** — a naive last-observation forecast, a random walk with drift
  and an OLS linear trend are evaluated under the identical protocol, with MASE
  and RelMAE reported alongside MAPE, MAE, RMSE and R².
- **Model comparison** — Diebold–Mariano with squared-error loss and the
  Harvey–Leybourne–Newbold small-sample correction, reported as exploratory given
  seven forecasts (six excluding the rebasing fold).
- **ARIMA order** — selected by AIC grid search over p, d, q ≤ 2 with
  `seasonal_order=(0,0,0,0)`; the full search is in `arima_order_selection.csv`.
- **Uncertainty** — the projection carries an explicit scenario range over the
  assumed growth rate. Residual-bootstrap prediction intervals are deliberately
  not reported: inter-censal residuals of ~100 persons reflect a deterministic
  administrative rule, so intervals built from them would be indefensibly narrow.

## Headline result

Projection anchored on the 2024 census level, with the growth rate extrapolated
from its 2011–2023 trend (`outputs/tables/final_forecast_2025_2030.csv`):

| Year | Projection | Scenario range | Assumed growth |
|---|---|---|---|
| 2025 | 93,342 | 93,285 – 93,559 | 1.914 % |
| 2026 | 95,071 | 94,897 – 95,572 | 1.852 % |
| 2027 | 96,773 | 96,420 – 97,627 | 1.790 % |
| 2028 | 98,445 | 97,847 – 99,727 | 1.728 % |
| 2029 | 100,085 | 99,173 – 101,873 | 1.666 % |
| 2030 | 101,691 | 100,395 – 104,064 | 1.604 % |

An ARIMA(0,2,2) with a 2024 step dummy corroborates this independently, landing
within 488 persons of it in 2030. The scenario range covers the growth-rate
assumption only, not uncertainty in the 2024 census level itself.

Smooth trends fitted across the 2024 discontinuity are reported **because they
fail**: a second-degree polynomial and a log-linear trend project a 2025
population of 88,083 and 84,891 respectively — below the observed 2024 value of
91,589.

## Climate data quality

The station log carries transcription artefacts that a blank-cell check does not
catch. Six rules identify them; five remove and impute the affected cell, one
flags for review only. All 75 hits (57 corrected, 18 flagged and retained) are
logged in `outputs/tables/data_quality_flags.csv`. Three clusters dominate:

* **2016** — the maximum-temperature column is filled down with a constant
  16.0 °C from April to December, putting the monthly maximum below the monthly
  mean for seven months (21 cells corrected).
* **2020** — June–September carry a repeated placeholder block across five
  variables, plus summer minima near 8 °C implying a 31 °C diurnal range (28 cells).
* **2023** — the evaporation pan reads 0.9–7.3 mm/month from January to August
  against a July norm of ~280 mm; the instrument was out of service (9 cells).

Uncorrected, these depress the 2016 mean annual maximum by 10 °C and reduce the
2023 annual evaporation total to a fifth of its true value.

## Data provenance

* **Population 2010–2023** — Statistics Directorate, Soran Independent
  Administration, *Estimated population of the Soran Independent Administration
  area (Soran central district), 2010–2023*. Base: the 2009–2010 building and
  population enumeration, with growth applied as an estimated percentage rate.
* **Population 2024** — 2024 population and housing census round; Soran district
  total 179,596 (50.21 % male / 49.79 % female).
* **Climate 2015–2024** — Soran agro-meteorological station (36°39′N, 44°32′E,
  680 m a.s.l.), monthly agro-meteorological log.

## Citation

If you use this code or data, please cite the accompanying paper (under review)
and this repository.
