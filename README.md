# Electricity Consumption Forecasting and Energy Analytics

## Overview
Problem: Conventional electricity-consumption forecasting systems primarily focus on predicting future demand and often provide limited information about abnormal consumption, peak-demand risk, prediction uncertainty, or the factors responsible for changes in electricity usage.

Proposed solution: Develop an R-based intelligent energy analytics framework that combines time-series forecasting, machine learning, anomaly detection, peak-demand prediction, explainable AI and scenario-based energy optimization.

It is built on a Three-Layer Intelligence Architecture:
1. **Predictive Intelligence**: Multi-horizon forecasting, peak-risk prediction, and uncertainty intervals.
2. **Diagnostic Intelligence**: Forecast-residual anomaly detection, SHAP feature drivers, and load behavior profiling.
3. **Prescriptive Intelligence**: What-if energy optimization and personalized recommendations.

## Dataset
We use the **UCI Individual Household Electric Power Consumption** dataset.
- ~2 million rows (Dec 2006 - Nov 2010), one observation per minute.
- Target Variable: `Global_active_power` (in kilowatts)
- Features include extensive sub-metering, engineered lag variables, and rolling statistical indicators.

| Check | Value you should see in `01_load.R` |
|---|---|
| Rows | **2,075,259** |
| Missing `Global_active_power` | **25,979 (1.25%)** |
| Gap spells | **71** |
| Longest gap | **7,226 minutes (~120.4 h), 17 Aug 2010 21:02 – 22 Aug 2010 21:27** |
| File size | ~127 MB txt |

Direct zip (old `/ml/machine-learning-databases/` links 404):

`https://archive.ics.uci.edu/static/public/235/individual+household+electric+power+consumption.zip`

If that path rotates, use the **Download** button on the dataset page. Put `household_power_consumption.txt` in `data/raw/`.

**Weather:** Open-Meteo archive for Sceaux (`data/external/weather_sceaux.csv` is already included). Climatology proxy if the API is blocked.

**Not a substitute:** CRAN `mixvlmc::powerconsumption` — cleaned **2008, 10-minute, 52,704 rows**. Use it only to dry-run scripts.

Rest-of-house (UCI note): `(Global_active_power * 1000 / 60) − Sub_metering_1 − 2 − 3` (watt-hours per minute). Do **not** multiply kW by 1000 without `/60`.

## Project layout

```text
Electricity-Consumption-Forecasting/
├── R/                 00_utils.R … 10_run_all.R
├── data/raw/          household_power_consumption.txt
├── data/external/     weather_sceaux.csv
├── data/processed/    hourly / daily / features .rds (after pipeline)
├── app/app.R          Shiny dashboard
├── notebooks/analysis.Rmd
├── outputs/           figures/ models/ forecasts/ tables/
├── report/            report.Rmd  slides.Rmd
└── README.md
```

## How to run (RStudio)

1. Open `Electricity-Consumption-Forecasting.Rproj`.
2. `source("R/00_setup.R")` — installs packages, creates folders.
3. `source("R/10_run_all.R")` — download check + modules 01–09.  
   `05_models.R` is the slow step (about 10–30 minutes). Prophet is optional.
4. `shiny::runApp("app")`
5. Knit `report/report.Rmd` and `report/slides.Rmd`.

Scripts also work if you `setwd("R")` and `source("01_load.R")` in order; `00_utils.R` locates the project root either way.

## Modules

| Script | What it does |
|---|---|
| `00_download.R` | UCI zip + Open-Meteo + optional mixvlmc |
| `01_load.R` | `fread`, POSIXct, missingness, quality score |
| `02_clean.R` | pad, short-gap interpolation, 15 min→monthly |
| `03_features.R` | calendar, lags, Fourier, weather, CDD/HDD, ToD |
| `04_eda.R` | ~20 figures (profile, heatmap, STL, sub-meters, …) |
| `05_models.R` | chronological daily + hourly horse-race |
| `06_evaluate.R` | leaderboard plots, ACF, Ljung–Box |
| `07_anomaly.R` | STL+IQR, collective flags, theft injection, PELT |
| `08_cluster.R` | k-means on **z-scored daily shapes** |
| `09_cost.R` | TANGEDCO slabs, CO₂, shift, PV, +2 °C |
| `app/app.R` | KPIs, EDA, forecast ribbons, upload CSV |

## Dashboard

Tabs: Overview · EDA · Sub-meters · Model comparison · Actual vs predicted · Forecast · Anomalies · Clusters · Cost & carbon · Solar & what-if · Upload CSV.

Date range, hourly/daily/monthly grain, horizon slider, filtered-CSV download. Seasonal naive always runs; saved XGBoost/RF forecasts appear after module 05.

## Cost overlay (read this before the viva)

- **Slabs:** TNERC LT-IA from 1 July 2024 (bi-monthly Rs/kWh: 4.80 / 6.45 / 8.55 / 9.65 / 10.70 / 11.80). Check: 500 units → **Rs 2,565**.
- **CO₂:** CEA Baseline Database v20.0, FY 2023–24 weighted average **0.727 kg/kWh**.
- **ToD:** peak 06:00–10:00 & 18:00–22:00 (+25%), off-peak 22:00–05:00 (−5%) as in TNERC HT/selected LT. **Ordinary domestic LT-IA is still slab-only**; ToD is a load-shift simulator.

## Headline result

After `05_models.R`, copy RMSE/MAPE from `outputs/tables/model_comparison.csv` into this sentence:

> XGBoost with lag and weather features achieved MAPE of **X%** on a 7-day chronological hourly holdout, versus **Y%** for seasonal naive (lag 168) and **Z%** for the lag/weather linear model.

Single-home MAPE is usually much higher than the 2–5% quoted on **national** demand. The fair comparison is versus naive on the **same split**.

## Package stack

**Core:** `data.table`, `dplyr`, `tidyr`, `lubridate`, `ggplot2`, `zoo`, `imputeTS`, `forecast`, `tseries`, `xgboost`, `ranger`, `rpart`, `changepoint`, `cluster`, `shiny`, `shinydashboard`, `plotly`, `DT`, `rmarkdown`

**Optional:** `prophet`, `tsibble`, `feasts`, `anomalize`, `padr`, `mixvlmc`, `mgcv`, `e1071`

## Licence / ethics

UCI data are from a real French household. Do not present TANGEDCO numbers as that family’s bill. Do not claim the anomaly injector is a theft-prosecution tool.
