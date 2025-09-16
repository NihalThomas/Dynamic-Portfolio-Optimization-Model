# Professional Usage Guide for Dynamic Portfolio Optimization Model

## Overview
This model implements a dynamic portfolio optimization strategy for a basket of cryptocurrencies, aiming to maximize risk-adjusted returns by targeting a specific annualized volatility. It uses an Exponentially Weighted Moving Average (EWMA) covariance matrix, Ledoit-Wolf shrinkage, tangency portfolio weights, and volatility scaling, with periodic rebalancing and transaction cost considerations. The model compares its performance against a Bitcoin (BTC) benchmark.


## Workflow
1. **Data Retrieval**: Downloads closing prices for specified tickers.
2. **Returns**: Computes daily percentage returns.
3. **Covariance**: Estimates EWMA covariance with Ledoit-Wolf shrinkage.
4. **Optimization**: Calculates tangency weights, scales to target volatility.
5. **Rebalancing**: Updates weights every `REBALANCE_DAYS`, applying turnover smoothing and transaction costs.
6. **Metrics**: Reports annualized return, volatility, Sharpe ratio, max drawdown.
7. **Visualization**: Plots portfolio value vs. BTC benchmark.

## Parameters and Impact
- **`TICKERS`** (Default: `["BTC-USD", "ETH-USD", "SOL-USD", ...]`): More volatile assets increase risk/return; stablecoins reduce volatility.
- **`START_DATE`, `END_DATE`** (Default: `"2024-01-01"`, `"2025-09-15"`): Longer periods improve robustness, shorter focus on recent trends.
- **`LOOKBACK`** (Default: `126`): Longer smooths estimates, shorter increases responsiveness but noise.
- **`EWMA_LAMBDA`** (Default: `0.97`): Higher emphasizes older data, lower reacts faster to recent changes.
- **`TARGET_ANN_VOL`** (Default: `0.12`): Higher allows more risk/return, lower is conservative.
- **`TRADING_DAYS`** (Default: `252`): Use `365` for crypto accuracy, `252` for equity alignment.
- **`REBALANCE_DAYS`** (Default: `7`): Shorter increases responsiveness but costs, longer reduces costs but drifts.
- **`TURNOVER_SHRINK`** (Default: `0.5`): Higher reduces turnover/costs, lower optimizes weights faster.
- **`TRANSACTION_COST_BPS`** (Default: `50`): Higher penalizes turnover, lower allows aggressive rebalancing.
- **`VOL_REGIME_THRESHOLD`** (Default: `0.08`): Higher maintains risk exposure, lower triggers conservative mode.

## Recommendations
- Adjust `TICKERS` for risk-return balance.
- Tune `LOOKBACK`, `EWMA_LAMBDA` for market responsiveness.
- Set `TARGET_ANN_VOL` per risk tolerance.
- Optimize `REBALANCE_DAYS`, `TURNOVER_SHRINK` for cost-efficiency.


## Conclusion
This dynamic portfolio optimization model provides a robust framework for managing cryptocurrency investments. By carefully tuning parameters, users can balance risk, return, and transaction costs to suit their investment objectives. Regular monitoring and adjustment are recommended to adapt to evolving market conditions.
