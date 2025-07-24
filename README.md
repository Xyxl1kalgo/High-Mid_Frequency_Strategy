# High\Mid_Frequency_Strategy
Strategy based on volume, correlation etc


# Quantitative Trading Strategy: Data Analysis & Initial Backtesting

This document details the preliminary data analysis conducted for a quantitative trading strategy. Understanding the statistical properties and time-series characteristics of our key indicators is crucial for building a robust and effective trading model. Following the data analysis, we introduce the initial backtesting setup, demonstrating how the strategy's rules are applied to historical data without optimization.

---

### 1. Statistical Properties of Key Features

The following table summarizes the statistical properties of the primary indicators across the entire dataset: **Volume Imbalance**, **Log Return**, and **Correlation with SPY**.

| Statistic   | volume_imbalance | log_return | corr_spy |
| :---------- | :--------------- | :--------- | :------- |
| **count** | 9750.00          | 9750.00    | 9750.00  |
| **mean** | 5415.41          | 0.000006   | 0.577182 |
| **std** | 188740.50        | 0.000378   | 0.458974 |
| **min** | -3,074,698.00    | -0.006196  | -1.000000|
| **25%** | -54,959.75       | -0.000143  | 0.333685 |
| **50% (median)** | 7,484.50         | 0.000004   | 0.735768 |
| **75%** | 59,766.75        | 0.000157   | 0.960383 |
| **max** | 6,982,345.00     | 0.003466   | 1.000000 |

**Key Observations:**

* **`volume_imbalance`:**
    * The **mean is very close to zero** (5,415), indicating no inherent long-term bias towards buying or selling pressure on average.
    * The **high standard deviation (188,740)** and wide **min/max range (-3M to +7M)** show that `volume_imbalance` is highly volatile and frequently experiences significant deviations from its mean. This confirms that our chosen thresholds (e.g., `100,000` and `-100,000` for buy/sell signals) target truly **extreme and actionable events**.
* **`log_return`:**
    * As expected for high-frequency financial data, the **mean is virtually zero**, and the **standard deviation is very small (0.000378)**, reflecting typical minute-by-minute price changes. This indicator's primary role is for analyzing volatility and building advanced models, rather than direct signal generation in this strategy.
* **`corr_spy`:**
    * Good to know that in data, I have not delete corr with SPY, due to this, max is 1 and histogramm have relocate to the right side
    * The **mean (0.577)** indicates that assets in our dataset generally exhibit a **moderate positive correlation** with the broader market (SPY).
    * Crucially, the **min value of -1.00** confirms that periods of strong **negative correlation with SPY do exist** in the data. This is a significant finding that validates the strategy's ability to potentially short assets that move inversely to the market, allowing for both trend-following (high positive correlation) and contrarian/hedging (negative correlation) approaches.

---

### 2. Distribution Visualizations

![Feature Distributions: Volume Imbalance, Log Return, Correlation with SPY](/feature_distributions_your_data.png)

**Visual Confirmation:**

* The histogram for **Volume Imbalance** visually confirms a bell-shaped distribution centered near zero, with noticeable "fat tails" extending to both positive and negative extremes. This reinforces that large imbalances are infrequent but impactful events.
* The **Log Return** histogram is sharply peaked around zero, indicating that most minute-to-minute price changes are small.
* The **Correlation with SPY** histogram shows a distribution skewed towards positive values, but with a noticeable presence of values extending into the negative range, visually confirming the `describe()` findings and the viability of negative correlation signals.

---

### 3. Time Series Properties: Stationarity and Autocorrelation

To understand the temporal behavior of our indicators, we performed the Augmented Dickey-Fuller (ADF) test for stationarity and plotted the Autocorrelation Function (ACF) for **AAPL** as a representative example.

**ADF Test Results for AAPL:**

* **`close`:** p-value = 0.4166 (Not Stationary)
    * **Interpretation:** Stock prices (`close`) are inherently non-stationary, exhibiting trends where their statistical properties change over time. This is a standard characteristic of financial time series.
* **`volume_imbalance`:** p-value = 0.0000 (Highly Stationary)
    * **Interpretation:** This is an excellent result. The near-zero p-value strongly indicates that **Volume Imbalance is a stationary series**. This implies that its statistical properties, such as its mean-reverting tendency (returning to zero), are consistent over time. This makes it a highly reliable indicator for strategies that capitalize on deviations from equilibrium.
* **`log_return`:** p-value = 0.0000 (Highly Stationary)
    * **Interpretation:** As expected, log returns are also highly stationary. This property is fundamental for many quantitative models, allowing for robust statistical analysis and predictability based on volatility.
* **`corr_spy`:** p-value = 0.0000 (Highly Stationary)
    * **Interpretation:** The strong stationarity of `corr_spy` is a valuable finding. It means that the relationship between the asset and the broader market, while dynamic, maintains consistent statistical characteristics. This enhances the reliability of using correlation thresholds for signal generation.

**ACF Plots for AAPL:**

![Autocorrelation Plots for AAPL: Volume Imbalance & Log Return](/autocorrelation_plots_your_data.png)

* **`volume_imbalance` ACF Plot:**
    * **Expected Behavior:** The ACF plot for `volume_imbalance` typically shows autocorrelation coefficients (blue bars) that **decay quickly** into or near the confidence interval (blue shaded area).
    * **Implication:** This rapid decay confirms the mean-reverting nature of volume imbalance. Large positive or negative imbalances are typically followed by a swift return towards zero. This strongly supports a **mean-reversion trading strategy** where positions are taken on extreme `volume_imbalance` signals with an expectation of a relatively rapid normalization.
* **`log_return` ACF Plot:**
    * **Expected Behavior:** The ACF plot for `log_return` generally shows very little to no significant autocorrelation beyond the initial lag.
    * **Implication:** This is consistent with the Efficient Market Hypothesis for high-frequency data, suggesting that past returns offer minimal direct predictive power for future returns, reinforcing that this indicator is primarily for measuring volatility rather than direct signal generation.

---

### 4. Strategic Design Implications: Embracing Mean-Reversion

Based on this comprehensive data analysis, especially the strong mean-reverting nature of `volume_imbalance` and the presence of both positive and negative correlations with SPY, our trading strategy's core logic is informed as follows:

The strategy is designed as a **contrarian / mean-reversion model**, aiming to profit from price reversals after extreme market imbalances.

* **Volume Imbalance (VI) as the Primary Reversal Signal:**
    * The stationarity and rapid autocorrelation decay of `volume_imbalance` make it an excellent candidate for identifying **overbought (excess buying pressure)** and **oversold (excess selling pressure)** conditions.
    * We anticipate that extreme deviations in `volume_imbalance` will lead to a **reversal** back towards equilibrium.

* **Correlation with SPY (`corr_spy`) for Market Context and Confirmation:**
    * The confirmed presence of both strong positive and negative correlations with SPY (and its stationarity) allows for nuanced market positioning. This indicator helps confirm if the asset's extreme behavior is isolated or part of a broader market move.

* **Simple Moving Average (SMA) for Trend Confirmation:**
    * The SMA is used to confirm the current short-term trend or the severity of the deviation from it.

---

### 5. Backtesting Implementation (Initial Setup)

With a clear understanding of our indicators' behavior and the revised strategy logic, we proceed to the backtesting phase to simulate the strategy's performance on historical data.

Our initial backtesting setup focuses on applying the defined entry and exit rules without any parameter optimization at this stage. This allows us to establish a baseline performance and identify the fundamental behavior of the strategy.

**Backtesting Framework:**

We utilize a robust backtesting framework (e.g., [Backtrader](https://www.backtrader.com/)) to:

1.  **Load Historical Data:** Ingest cleaned and pre-processed minute-level OHLCV data, including our calculated `volume_imbalance`, `log_return`, and `corr_spy`.
2.  **Define Strategy Rules:** Implement the exact entry and exit conditions based on the thresholds determined from our data analysis:
    * **Long Entry (Contrarian Buy):**
        * When `volume_imbalance` falls **below `sell_threshold_volume_imbalance`** (e.g., -100,000, indicating significant selling pressure/oversold).
        * **AND** `corr_spy` is **below `correlation_threshold_sell`** (e.g., -0.5, indicating strong negative correlation or divergence from market).
        * **AND** `close` price is **below SMA(period)** (confirming a short-term downtrend or oversold condition).
        * *Logic:* We buy when the asset is experiencing extreme selling pressure, is moving against the general market, and is below its short-term average, anticipating a reversal upwards.
    * **Short Entry (Contrarian Sell):**
        * When `volume_imbalance` **exceeds `buy_threshold_volume_imbalance`** (e.g., 100,000, indicating significant buying pressure/overbought).
        * **AND** `corr_spy` is **above `correlation_threshold_buy`** (e.g., 0.5, indicating strong positive correlation or over-extension with market).
        * **AND** `close` price is **above SMA(period)** (confirming a short-term uptrend or overbought condition).
        * *Logic:* We sell (short) when the asset is experiencing extreme buying pressure, is moving strongly with the general market, and is above its short-term average, anticipating a reversal downwards.
    * **Exit Conditions (Active Reversal):**
        * The strategy employs an aggressive **`exit_on_reverse_signal = True`** parameter. This means if an existing position's entry conditions are no longer met, **OR** if the conditions for an immediate opposite trade are met, the current position will be closed, and a new position in the opposite direction will be opened in the same bar. This allows for rapid capitalization on mean-reverting swings.
3.  **Execute Trades:** Simulate trade execution with specified commission, slippage (if modeled), and capital management rules.
4.  **Generate Performance Metrics:** Calculate key metrics such as total return, Sharpe Ratio, drawdown, win rate, etc.
