# Forecasting the U.S. Gasoline-Diesel Price Spread

A time series analysis comparing exponential smoothing, ARIMA and ARIMAX models for forecasting the weekly price gap between U.S. retail gasoline and diesel.

**Result:** ARIMA(4,1,3) was the most accurate model, with a rolling one-step-ahead holdout RMSE of **$0.0232 per gallon**, about **20% lower error** than a persistence forecast (next week = this week).

📄 **Full report:** [Project_Report.pdf](Project_Report.pdf)

## Why this matters
Gasoline mostly powers passenger vehicles, while diesel powers freight and commercial transport. The spread between the two reflects supply, demand and seasonal conditions, so forecasting it is useful for transportation businesses, policymakers and consumers sensitive to fuel costs.

## Data
- **Fuel prices:** weekly U.S. retail gasoline (all grades, all formulations) and diesel prices from the U.S. Energy Information Administration (EIA), via [Kaggle](https://www.kaggle.com/datasets/mruanova/us-gasoline-and-diesel-retail-prices-19952021). January 1995 to December 2021, 1,361 weekly observations, no missing values.
- **External regressors:** WTI crude oil prices from the Federal Reserve Bank of St. Louis (FRED) and heating oil prices from the EIA.

**Target:** spread = gasoline price − diesel price, in dollars per gallon.

## Method
1. **Exploratory analysis:** ACF and PACF plots showed the spread is non-stationary, with a structural shift around 2008 after which diesel stayed consistently more expensive than gasoline. First differencing was enough to make the series stationary. CCF analysis showed gasoline and diesel prices move together in the same week, with neither consistently leading the other.
2. **Baselines:** persistence (last observed value) and the training mean.
3. **Exponential smoothing:** Holt linear and seasonal Holt-Winters (26- and 52-week periods).
4. **ARIMA:** started from the `auto.arima()` suggestion of ARIMA(3,1,2), then searched nearby (p, d, q) combinations and seasonal variants, since the lowest-AIC model is not always the best forecaster.
5. **ARIMAX:** added one-week-lagged crude oil and heating oil prices as external regressors.

**Evaluation:** every model was scored with rolling one-step-ahead forecasts over a 104-week holdout (January 2020 to December 2021), with coefficients fixed at their training estimates.

## Results

| Model | Holdout RMSE ($/gal) |
|---|---|
| Training mean | 0.2460 |
| Persistence | 0.0291 |
| Holt-Winters (52-week season) | 0.0309 |
| Holt-Winters (26-week season) | 0.0305 |
| Holt linear | 0.0253 |
| **ARIMA(4,1,3)** | **0.0232** |
| ARIMAX (lagged crude + heating oil, ARIMA(2,0,1) errors) | 0.0232 |

**Key findings**
- All three model families beat the persistence baseline.
- Seasonal components did not help, for either exponential smoothing or ARIMA.
- Crude and heating oil prices did not improve on ARIMA, which suggests the spread's own history already captures what matters for short-term forecasts.
- ARIMA(4,1,3) is recommended: it ties ARIMAX on accuracy but is simpler. Its residuals pass the Ljung-Box test for white noise (p = 0.365).

## How to run
Requires R with these packages:

    install.packages(c("forecast", "tidyverse", "zoo"))

Clone the repository, set it as your working directory, and knit the R Markdown files in this order:

| File | What it does |
|---|---|
| `stat443-project-EDA.Rmd` | Exploratory analysis and stationarity checks |
| `stat443-project-persistence-iid.Rmd` | Baseline models |
| `stat443-project-expsmo.Rmd` | Exponential smoothing models |
| `stat443-project-arima.Rmd` | ARIMA and seasonal ARIMA search |
| `stat443-project-arimax.Rmd` | ARIMAX models with external regressors |

Knitted PDFs of every file are in [`Project_Rmd_PDFs/`](Project_Rmd_PDFs).

## Project structure

    ├── Project_Report.pdf              final report
    ├── Project_Rmd_PDFs/               knitted PDF of each analysis file
    ├── stat443-project-*.Rmd           analysis code
    ├── PET_PRI_GND_DCUS_NUS_W.csv      weekly gasoline and diesel prices (EIA)
    ├── oilprices.csv                   daily WTI crude oil prices (FRED)
    ├── heatingoil.csv                  daily heating oil prices (EIA)
    └── *_candidates.csv                holdout RMSE for every ARIMA, seasonal ARIMA and ARIMAX candidate

## License
MIT. See [LICENSE](LICENSE).
