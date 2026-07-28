# nintendo-time-regression-forecasting-model
# Nintendo Quarterly Revenue Forecasting (AFM 244)

Time series regression model forecasting Nintendo Co., Ltd. (NTDOY) quarterly revenue, built in Python using `statsmodels`. This project was completed as part of AFM 244 (Week 11 participation activity) at the University of Waterloo.

## Overview

The goal of this project is to model and forecast Nintendo's quarterly revenue using an OLS regression that accounts for:

- **Seasonality** — Nintendo's fiscal Q3 (holiday quarter) consistently shows revenue spikes.
- **Structural change** — The March 2017 launch of the Nintendo Switch, which shifted the level and trend of revenue going forward.

The model is trained on historical data (2001–2023) and used to forecast the subsequent four quarters, with 80% confidence intervals.

## Data

- **Source file:** `qSales_2024.csv` (quarterly sales/financials panel data)
- **Filter:** Rows where `tic == 'NTDOY'`, with missing `saleq` (quarterly sales) values dropped
- **Date range:** FY2001 Q1 through FY2023 Q4 (83 observations)

## Methodology

1. **Data preparation**
   - Converted `datadate` to datetime
   - Filtered dataset to Nintendo only, dropped rows with missing revenue (`saleq`)

2. **Feature engineering**
   - `time`: incrementing integer time index (1, 2, 3, ...)
   - `holiday_dv`: dummy variable equal to 1 when `fqtr == 3` (Nintendo's holiday quarter, since Nintendo's fiscal year ends in March)
   - `switch_dv`: dummy variable equal to 1 for all dates on/after 2017-03-01 (Switch launch)
   - `holiday_interaction`: `time * holiday_dv`
   - `switch_interaction`: `time * switch_dv`

3. **Train/test split**
   - 75% of observations (chronologically first) used for training
   - Remaining 25% (most recent) held out for testing

4. **Model**
   - OLS regression (`statsmodels.api.OLS`) with a constant term
   - Predictors: `time`, `holiday_dv`, `switch_dv`, `holiday_interaction`, `switch_interaction`
   - Target: `saleq` (quarterly revenue)

5. **Evaluation**
   - Mean Absolute Percentage Error (MAPE) computed on the test set

6. **Forecasting**
   - Constructed future independent variables for the next 4 quarters (time = 84–87)
   - Used `model.get_prediction().summary_frame()` to generate point forecasts and 80% confidence intervals

## Results

**Fitted model:**

```
Revenue = 3007.67 − 42.32·time + 2290.91·holiday_dv − 3755.20·switch_dv
          − 1.91·holiday_interaction + 86.26·switch_interaction
```

**Model fit:** MAPE = 18.83% on the test set — the model's forecasts were off by about 18.8% on average, indicating a reasonably good but imperfect fit.

## Interpretation

- The **holiday dummy** confirms a strong seasonal lift in Nintendo's fiscal Q3.
- The **Switch dummy and interaction terms** capture the shift in revenue level and trend following the console's 2017 launch.
- The relatively high MAPE suggests the model captures broad patterns (seasonality, structural break) but doesn't fully account for revenue volatility tied to specific product cycles (e.g., individual game/hardware launches) not represented in the feature set.

## Tech Stack

- Python 3
- `pandas`, `numpy`
- `statsmodels` (OLS regression)
- `matplotlib` (visualization)
- Google Colab (development environment)

## Repository Structure

```
├── qSales_2024.csv          # Source data (quarterly financials panel)
├── nintendo_forecast.ipynb  # Analysis notebook
└── README.md
```

## Notes

- I am not fully certain of the exact fiscal-year-end convention behind the holiday quarter classification beyond what's captured by `fqtr == 3` in the source data — worth verifying against Nintendo's official fiscal calendar if this is reused elsewhere.
- Confidence intervals in the forecast output are fairly wide, reflecting real uncertainty in quarterly revenue prediction from a relatively simple linear specification.
