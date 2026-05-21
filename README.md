# Z-Score Mean-Reversion Trading Strategy

A backtested algorithmic trading strategy implemented in R, applied to AAPL, AMD, and NVDA.

## Overview

The strategy uses a 20-day rolling z-score to identify when a stock's price has deviated significantly from its recent mean. It buys when the z-score falls near zero (price close to its mean) and sells when the z-score rises above a threshold (price extended above mean). Performance is compared against a buy-and-hold baseline.

## Strategy Logic

- **Signal:** Compute z-score = `(Close - MA20) / SD20` using a 20-day rolling window
- **Buy:** Enter when z-score ≤ 0.001 (price at or below 20-day mean, not currently holding)
- **Sell:** Exit when z-score ≥ 2.5 (price significantly above 20-day mean)
- **Return:** `strategy_return = position × daily_return`

## Analysis

- Cumulative strategy returns vs. buy-and-hold for all three tickers
- Z-score time series with buy/sell threshold lines
- Z-score distribution (histogram + KDE vs. normal fit)
- Z-score Q-Q plots to assess normality
- Block bootstrap means (k=5 and k=20) to estimate return distribution via CLT

## Files

| File | Description |
|---|---|
| `Project3.Rmd` | Full analysis — data loading, signals, returns, plots |
| `data/` | Expected location for `AAPL.csv`, `AMD.csv`, `NVDA.csv` |
| `plots/` | Output directory for generated PDFs |

## Requirements

```r
install.packages(c("tidyverse", "zoo", "lubridate"))
```

## Data Format

Each CSV should have at minimum a `time` column (parseable datetime) and a `close` column (closing price). Compatible with exports from most financial data APIs (e.g. Alpaca, Yahoo Finance via `quantmod`).

## Running

Open `Project3.Rmd` in RStudio and knit to HTML, or run chunk by chunk. Ensure `data/AAPL.csv`, `data/AMD.csv`, and `data/NVDA.csv` are present and a `plots/` directory exists.
