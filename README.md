# Quant Watchlist

Daily-updating quantitative analysis report for a watchlist of US equities and ETFs.

**Live report:** [your domain here once configured]

## What this is

A cross-sectional quant analysis tool that runs on GitHub Actions every day
and publishes its output to GitHub Pages. The report includes:

- Cross-sectional screener (sortable table of risk/return metrics for each name)
- QVM+L factor scores (Quality, Value, Momentum, Low Volatility) — industry-standard
  multi-factor screening as practiced by MSCI, S&P, and AQR
- Normalized cumulative-return chart showing relative performance
- Risk vs. return scatter with Sharpe iso-lines
- Drawdown comparison overlaying all names
- Hierarchically clustered correlation heatmap
- Long-only mean-variance efficient frontier with GMV / tangency / equal-weight /
  risk-parity reference points
- Rule-based strategy backtest (buy & hold, equal weight, risk parity,
  SMA crossover, 12-1 momentum, mean reversion) with realistic transaction costs
- Per-ticker technical drilldowns with SMA/Bollinger/RSI/MACD
- Curated academic literature linking each method to its primary source

## Setup

See [SETUP.md](SETUP.md) for step-by-step instructions on connecting this to
your custom domain.

## Stack

- Python 3.12, pandas, NumPy (<2 for compatibility), scipy, statsmodels
- yfinance for daily price data
- Plotly for charts
- GitHub Actions for scheduling and CI
- GitHub Pages for hosting

## Disclaimer

For educational and demonstrative purposes. Not investment advice. Past
performance does not predict future returns. Single-period rankings are noisy
estimates subject to multiple-testing concerns (Harvey, Liu & Zhu 2016).
