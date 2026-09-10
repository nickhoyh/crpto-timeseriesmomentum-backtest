# Crypto Momentum Backtest

A Python notebook exploring a volatility-targeted time-series momentum (TSMOM) strategy for Bitcoin, Ethereum, and Solana. Rolling walk-forward optimization selects a momentum lookback, and the notebook compares out-of-sample performance against an equal-weighted crypto benchmark.

## Getting started

The analysis lives in [`main.ipynb`](main.ipynb). Its recorded Python version is 3.13.7.

Create a virtual environment and install the notebook dependencies:

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install jupyterlab ipykernel numpy pandas yfinance matplotlib
python -m jupyterlab main.ipynb
```

On Windows, activate the environment with `.venv\Scripts\activate` instead. Select the environment's Python kernel and run the cells from top to bottom. An internet connection is required to download price data from Yahoo Finance through `yfinance`. The first cell also installs pandas, yfinance, and matplotlib.

## Method

1. Download daily closing prices for `BTC-USD`, `ETH-USD`, and `SOL-USD`, requesting January 1, 2020 through August 1, 2026 (exclusive). Forward-fill missing prices and drop rows with remaining missing values, so the usable period depends on all three assets having data.
2. Take the sign of each asset's return over the selected momentum lookback to determine its position direction.
3. Scale positions using 30-day realized volatility, a 50% annualized volatility target per asset, and a maximum absolute weight of 1.0 per asset before dividing exposure across the three assets. Signals and sizing are lagged by one day.
4. Deduct transaction costs proportional to the absolute daily change in weights.
5. Every 90 observations, select the lookback with the highest annualized Sharpe ratio over the preceding 365 observations and use its returns for the next out-of-sample block. Evaluation begins at observation index 515 to allow training and warm-up history.

The notebook uses 365 days for annualization.

## Configuration

Edit these values directly in the notebook:

| Setting | Default | Location |
| --- | --- | --- |
| Assets and date range | BTC, ETH, SOL; 2020-01-01 to 2026-08-01 | Data download cell |
| Momentum lookbacks | 15, 30, 45, 60, 90, 120 days | Strategy parameters cell |
| Volatility lookback | 30 days | `calc_strat_returns` |
| Annualized volatility target | 0.50 | `calc_strat_returns` |
| Maximum position magnitude | 1.0 | `calc_strat_returns` |
| Transaction cost | 60 basis points (0.6%) per unit of turnover | `calc_strat_returns` |
| Training window / evaluation step | 365 / 90 observations | Strategy parameters cell |

## Outputs

Running the notebook produces:

- A strategy-versus-benchmark table of CAGR, annualized volatility, Sharpe ratio, and maximum drawdown.
- Cumulative return curves on a logarithmic scale.
- Drawdown plots for the strategy and benchmark.
- A history of the selected momentum lookback.

Results appear inline. Dependencies are not pinned and prices are fetched live, so reruns may differ from saved notebook outputs.

## Implementation notes

Some notebook descriptions differ from the current code:

- **Position direction:** `np.sign(mom)` produces negative weights when momentum is negative. The implemented strategy is long/short, despite the prose describing it as long-only. Short financing and borrow costs are not modeled.
- **Transaction costs:** `cost_bps=60` is divided by 10,000, giving **0.6%**, rather than the 0.06% stated in the prose.
- **Benchmark:** averaging asset returns each day represents a portfolio rebalanced daily to equal weights, although plots label it buy-and-hold. Benchmark rebalancing costs are not deducted.
- **Parameter changes:** each lookback's returns and turnover are calculated separately before walk-forward selection. Switching lookbacks does not explicitly account for turnover between the previously held positions and the newly selected positions.
