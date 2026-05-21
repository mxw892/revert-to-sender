# revert-to-sender

a z-score mean-reversion trading strategy backtested in R on AAPL, AMD, and NVDA. the idea: when a stock drifts too far from its recent average, it tends to come back. wait for that moment, get in, ride the snap.

---

## the strategy

prices don't trend forever. they wander away from their moving average and eventually come back. a z-score measures exactly how far away a price currently is, in standard deviation units, from its recent mean:

```
z = (Close - MA20) / SD20
```

where `MA20` is the 20-day rolling average and `SD20` is the 20-day rolling standard deviation. a z-score near zero means the price is sitting right at its mean. a high z-score means it's stretched above it.

**signal logic:**
- **buy** when z-score ≤ 0.001 and not already holding — price is at or below its 20-day average, likely to push higher
- **sell** when z-score ≥ 2.5 and currently holding — price is 2.5 standard deviations above its mean, stretched, likely to pull back

one position at a time. no shorting. fully invested or fully out.

---

## what's in the analysis

**price + MA20 plots** — raw close price overlaid with the 20-day moving average for each ticker. shows where the strategy's reference line sits relative to actual price movement.

**z-score series** — the z-score over time with the buy (0.001) and sell (2.5) threshold lines drawn in. you can see exactly when signals fire and how often the strategy is in or out of the market.

**z-score distribution** — histogram of all z-score values with a KDE overlay and a fitted normal curve on top. lets you see whether the z-score actually behaves like a normal distribution (it mostly does, with heavier tails).

**z-score Q-Q plots** — quantile-quantile plots against a theoretical normal. another angle on the same question: how normal is the distribution of deviations?

**cumulative returns** — strategy equity curve vs. buy-and-hold for each ticker. the real scorecard. blue is the strategy, red dashed is passive holding.

**block bootstrap** — to assess whether the strategy returns are just noise, daily strategy returns are broken into non-overlapping blocks of size k=5 and k=20, block means are computed, and their distributions are plotted. larger blocks invoke the CLT more strongly — if the distribution of block means tightens toward normal as k grows, that's a good sign the returns have some structure.

---

## under the hood

everything runs through a consistent pipeline of R functions applied to a named list of ticker dataframes:

```r
data_list <- lapply(tickers, load_data)   # load and sort CSVs
data_list <- lapply(data_list, signal_logic)   # generate buy/sell signals
data_list <- lapply(data_list, compute_returns)  # compute strategy vs B&H returns
```

rolling stats use `zoo::rollmean` and `zoo::rollapply` with `align="right"` so there's no lookahead — the strategy only ever sees past data. signals are generated in a forward loop with explicit position tracking to avoid vectorized lookahead bias.

---

## files

| file | what it does |
|---|---|
| `Project3.Rmd` | full analysis: data loading, signals, returns, all plots |
| `data/` | put your CSVs here — `AAPL.csv`, `AMD.csv`, `NVDA.csv` |
| `plots/` | generated PDFs land here |

---

## data format

each CSV needs at minimum:

| column | description |
|---|---|
| `time` | datetime string (parseable by `lubridate::as_datetime`) |
| `close` | closing price |

compatible with exports from Alpaca, Yahoo Finance via `quantmod`, or most standard OHLCV sources.

---

## requirements

```r
install.packages(c("tidyverse", "zoo", "lubridate"))
```

open `Project3.Rmd` in RStudio and knit to HTML, or run chunk by chunk. make sure `data/` exists with the three CSVs and `plots/` exists for output.
