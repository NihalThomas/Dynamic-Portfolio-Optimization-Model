# Professional Usage Guide for Dynamic Portfolio Optimization Model

## Overview
This model implements a dynamic portfolio optimization strategy for a basket of cryptocurrencies, aiming to maximize risk-adjusted returns by targeting a specific annualized volatility. It uses an Exponentially Weighted Moving Average (EWMA) covariance matrix, Ledoit-Wolf shrinkage, tangency portfolio weights, and volatility scaling, with periodic rebalancing and transaction cost considerations. The model compares its performance against a Bitcoin (BTC) benchmark.


## Model Workflow
1. **Data Retrieval**: Downloads closing prices for specified tickers from Yahoo Finance.
2. **Returns Calculation**: Computes daily percentage returns.
3. **Covariance Estimation**: Uses EWMA to estimate covariance, followed by Ledoit-Wolf shrinkage for robustness.
4. **Portfolio Optimization**: Calculates tangency portfolio weights and scales them to target volatility.
5. **Rebalancing**: Adjusts weights every `REBALANCE_DAYS`, incorporating turnover constraints and transaction costs.
6. **Performance Metrics**: Computes portfolio value, annualized return, volatility, Sharpe ratio, and maximum drawdown.
7. **Visualization**: Plots portfolio value against a BTC benchmark.

## Parameters and Their Impact
Below is a detailed description of the configurable parameters in the model and how changes to these parameters affect its behavior and performance.

### 1. `TICKERS`
- **Description**: List of cryptocurrency tickers (e.g., `["BTC-USD", "ETH-USD", ...]`).
- **Default**: `["BTC-USD", "ETH-USD", "SOL-USD", "AAVE-USD", "LINK-USD", "USDC-USD", "USDT-USD", "PAXG-USD"]`
- **Impact**:
  - **Adding/Removing Tickers**: Including more volatile assets (e.g., altcoins) increases portfolio risk but may enhance returns. Adding stablecoins (e.g., USDC, USDT) reduces volatility but may lower returns.
  - **Diversification**: More tickers generally improve diversification, reducing idiosyncratic risk, but increase transaction costs due to more frequent rebalancing across assets.
  - **Data Quality**: Ensure tickers are valid and have sufficient historical data to avoid `NaN` values, which can disrupt calculations.

### 2. `START_DATE` and `END_DATE`
- **Description**: Date range for historical price data (format: `YYYY-MM-DD`).
- **Default**: `START_DATE = "2024-01-01"`, `END_DATE = "2025-09-15"`
- **Impact**:
  - **Longer Periods**: Extending the date range provides more data for covariance estimation, improving robustness but potentially including outdated market regimes.
  - **Shorter Periods**: Shortening the range focuses on recent market conditions but may lead to overfitting or insufficient data for reliable covariance estimates.

### 3. `LOOKBACK`
- **Description**: Number of days used for estimating mean returns and covariance matrix.
- **Default**: `126` (approximately 6 months).
- **Impact**:
  - **Increase**: A longer lookback smooths estimates, capturing longer-term trends but may lag in fast-changing markets (e.g., crypto bull/bear cycles).
  - **Decrease**: A shorter lookback makes the model more responsive to recent market conditions but increases noise in covariance and mean estimates, potentially leading to unstable weights.

### 4. `EWMA_LAMBDA`
- **Description**: Decay factor for EWMA covariance calculation, controlling weight given to recent returns.
- **Default**: `0.97`
- **Impact**:
  - **Higher (closer to 1)**: Emphasizes older data, producing smoother covariance estimates but slower adaptation to market changes.
  - **Lower (e.g., 0.94)**: Increases weight on recent returns, making the model more reactive but potentially less stable due to noise in recent data.

### 5. `TARGET_ANN_VOL`
- **Description**: Target annualized portfolio volatility.
- **Default**: `0.12` (12%).
- **Impact**:
  - **Increase (e.g., 0.20)**: Allows higher volatility, increasing potential returns but also risk and drawdowns.
  - **Decrease (e.g., 0.08)**: Reduces portfolio risk, leading to more conservative allocations (e.g., higher weights in stablecoins), but may cap upside potential.

### 6. `TRADING_DAYS`
- **Description**: Number of trading days in a year for annualizing returns and volatility.
- **Default**: `252`
- **Impact**:
  - **Crypto Context**: Cryptocurrencies trade 365 days a year, so setting to `365` may be more accurate for crypto markets, increasing annualized metrics slightly.
  - **Traditional Markets**: Using `252` aligns with equity market conventions but slightly underestimates crypto volatility/returns.

### 7. `REBALANCE_DAYS`
- **Description**: Frequency of portfolio rebalancing (in days).
- **Default**: `7` (weekly).
- **Impact**:
  - **Decrease (e.g., 1)**: Daily rebalancing increases responsiveness to market changes but significantly raises transaction costs, reducing net returns.
  - **Increase (e.g., 30)**: Monthly rebalancing reduces transaction costs but may miss short-term optimization opportunities, leading to drift from the target volatility.

### 8. `TURNOVER_SHRINK`
- **Description**: Smoothing factor for portfolio weights to reduce turnover (blends new weights with previous weights).
- **Default**: `0.5`
- **Impact**:
  - **Increase (e.g., 0.8)**: Higher smoothing retains more of the previous portfolio, reducing turnover and transaction costs but potentially deviating from optimal weights.
  - **Decrease (e.g., 0.2)**: Less smoothing allows faster shifts to new weights, improving optimization but increasing turnover and costs.

### 9. `TRANSACTION_COST_BPS`
- **Description**: Transaction cost per trade in basis points (bps).
- **Default**: `50` (0.5%).
- **Impact**:
  - **Increase (e.g., 100)**: Higher costs penalize turnover, reducing net returns, especially with frequent rebalancing or high `TURNOVER_SHRINK`.
  - **Decrease (e.g., 20)**: Lower costs allow more aggressive rebalancing, improving adherence to optimal weights but requiring realistic exchange fee assumptions.

### 10. `VOL_REGIME_THRESHOLD`
- **Description**: Threshold for recent market volatility to trigger a reduced target volatility.
- **Default**: `0.08` (8% annualized).
- **Impact**:
  - **Increase (e.g., 0.12)**: Makes the model less likely to enter a low-volatility regime, maintaining higher risk exposure during volatile periods.
  - **Decrease (e.g., 0.05)**: Triggers low-volatility regime more often, reducing target volatility in moderately volatile markets, leading to more conservative allocations.

## Key Functions and Their Roles
- **`ewma_cov`**: Computes EWMA covariance matrix, emphasizing recent data based on `EWMA_LAMBDA`.
- **`ledoit_wolf_shrinkage`**: Regularizes the covariance matrix to reduce estimation errors, balancing sample covariance with a structured matrix.
- **`tangency_weights`**: Calculates optimal weights maximizing the Sharpe ratio, ensuring non-negative weights.
- **`volatility_scale`**: Scales weights to achieve the target annualized volatility, adjusting for market conditions.
- **`max_drawdown`**: Measures the worst peak-to-trough loss, a key risk metric.

## Performance Metrics
The model outputs:
- **Final Value**: Portfolio value at the end of the period.
- **Total Return**: Cumulative return over the period.
- **Annual Return**: Annualized return based on `TRADING_DAYS`.
- **Annual Volatility**: Annualized standard deviation of daily returns.
- **Sharpe Ratio**: Annual return divided by annual volatility.
- **Max Drawdown**: Maximum percentage loss from a peak.

## Sensitivity Analysis
- **High Volatility Assets**: Increasing weights in volatile assets (via `TICKERS`) boosts potential returns but increases drawdowns.
- **Rebalancing Frequency**: More frequent rebalancing (`REBALANCE_DAYS`) improves tracking of optimal weights but incurs higher costs.
- **Volatility Targeting**: Lower `TARGET_ANN_VOL` reduces risk but may underperform in bullish markets.
- **Covariance Estimation**: Adjusting `EWMA_LAMBDA` or `LOOKBACK` affects how quickly the model adapts to market shifts, balancing stability and responsiveness.


## Example Usage
To run the model with different parameters:
1. Modify `TICKERS` to include desired assets.
2. Adjust `START_DATE`, `END_DATE`, and `LOOKBACK` for the desired historical period.
3. Tune `EWMA_LAMBDA`, `TARGET_ANN_VOL`, `REBALANCE_DAYS`, `TURNOVER_SHRINK`, and `TRANSACTION_COST_BPS` based on market conditions and goals.
4. Execute the script to generate performance metrics and a comparison plot against BTC.


## Conclusion
This dynamic portfolio optimization model provides a robust framework for managing cryptocurrency investments. By carefully tuning parameters, users can balance risk, return, and transaction costs to suit their investment objectives. Regular monitoring and adjustment are recommended to adapt to evolving market conditions.
