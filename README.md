# Portfolio-Optimization
# Portfolio Optimization Lab

This project is a small lab for experimenting with classic portfolio-construction techniques in Python.  
It walks through getting price data, estimating return and risk, and building several equity portfolios used in quantitative finance.

## Universe and Data

- Large-cap U.S. equities (AAPL, MSFT, GOOGL, AMZN, META, NVDA, JPM, XOM, JNJ, PG)  
- Adjusted close prices downloaded from Yahoo Finance (`yfinance`)  
- Daily log-returns computed from prices  
- Annualization based on 252 trading days  

From these returns the notebook estimates:

- The **mean return vector**  
- The **covariance matrix** of log-returns  
- The **correlation matrix** implied by the covariance

Two heatmaps help visualize the dependence structure:

- `correlation_heatmap.png` – correlations (scale-free co-movement, between −1 and 1)  
- `covariance_heatmap.png` – covariances (co-movement in return units, influenced by volatility)

Correlation is easier to interpret and is central to clustering in Hierarchical Risk Parity.  
Covariance keeps the actual scale of risk and is what enters variance-based optimizations such as Markowitz and Max-Sharpe portfolios.

## Portfolios Implemented

Four portfolios are constructed from the same return and risk estimates:

1. **Equal-Weight Portfolio (EW)**  
   - Each asset gets the same capital weight.  
   - No optimization, no estimation risk beyond choosing the universe.  
   - Serves as a simple, transparent benchmark.

2. **Minimum-Variance Portfolio (Markowitz MinVar)**  
   - Objective: minimize portfolio variance  
     \[
     \min_w \; w^\top \Sigma w
     \]
   - Subject to:
     - Long-only: \( w_i \ge 0 \)  
     - Fully invested: \( \sum_i w_i = 1 \)  
   - Implemented with `scipy.optimize.minimize`.  
   - Tends to concentrate in lower-volatility and less-correlated names.  
   - Good for risk reduction, but sensitive to estimation error in the covariance matrix and can become unintuitively concentrated.

3. **Maximum Sharpe Ratio Portfolio (Markowitz MaxSharpe)**  
   - Incorporates both expected return and risk.  
   - Objective: maximize Sharpe ratio relative to a risk-free rate \( r_f \):
     \[
     \max_w \; \frac{w^\top \mu - r_f}{\sqrt{w^\top \Sigma w}}
     \]
   - Implemented by minimizing the negative Sharpe ratio with the same long-only, fully-invested constraints.  
   - Typically allocates more to higher-return, higher-risk assets (e.g., high-beta tech names).  
   - Can deliver much higher growth in-sample, but is even more sensitive to noisy mean-return estimates and may be unstable out-of-sample.

4. **Hierarchical Risk Parity (HRP)**  
   - A purely risk-based method that avoids direct inversion of the covariance matrix.  
   - Main steps:
     1. Convert covariances to a distance metric based on the **correlation matrix**.  
     2. Perform hierarchical clustering to build a tree (dendrogram) of assets.  
     3. Allocate risk top-down:  
        - Split the tree into clusters.  
        - Allocate portfolio weight across clusters in proportion to their total variance (lower-risk clusters get more weight).  
        - Within each cluster, allocate using inverse variance or similar rules.  
   - The result distributes **risk**, not capital, more evenly across uncorrelated clusters.  
   - Often yields diversified, more stable portfolios that are less sensitive to estimation noise than classic Markowitz solutions.

## Method Comparison

The chart `equity_curve.png` compares the cumulative performance of:

- Equal-Weight (EW)  
- Markowitz Minimum-Variance  
- Markowitz Maximum-Sharpe  
- Hierarchical Risk Parity (HRP)

Qualitative comparison:

- **Equal-Weight**  
  - Simple, diversification by construction.  
  - No reliance on estimated parameters, so it is surprisingly hard to beat out-of-sample.

- **MinVar**  
  - Lowest volatility and drawdowns in many regimes.  
  - May lag in strong bull markets because it systematically de-emphasizes high-volatility winners.  
  - Heavily driven by the covariance matrix; small changes in estimates can produce very different allocations.

- **MaxSharpe**  
  - Aggressive tilt toward assets with high estimated risk-adjusted returns.  
  - In the equity curve you will typically see the steepest growth but also deeper drawdowns.  
  - Very sensitive to estimation error in both means and covariances; prone to overfitting in-sample.

- **HRP**  
  - Uses the **correlation structure** to cluster assets (e.g., tech names vs. defensives vs. energy/financials).  
  - Attempts to give each cluster a balanced share of portfolio risk.  
  - Usually sits between EW and MinVar in terms of volatility, and between EW and MaxSharpe in terms of return.  
  - More robust when the covariance matrix is noisy, especially with many assets or short histories.

In other words:

- Covariance-driven Markowitz portfolios (MinVar, MaxSharpe) are powerful but fragile.  
- HRP is more robust and diversification-oriented, leveraging correlation patterns across assets.  
- EW is the no-assumption baseline: easy to implement and a good reference for evaluating whether more complex methods are truly adding value.

## Correlation vs Covariance in Practice

Looking at the heatmaps:

- The **correlation matrix** shows that all assets are positively correlated, reflecting broad U.S. equity market exposure. Clusters of stronger correlation (e.g., among large tech names) explain why HRP forms natural groups.  
- The **covariance matrix** combines these correlations with each asset’s volatility. High-volatility stocks contribute large entries both on the diagonal (variance) and off-diagonal (covariance), which strongly influences Markowitz allocations.

Markowitz methods care about the absolute scale of risk (covariance), whereas HRP focuses first on how assets move together (correlation) and then on their relative variances when allocating risk.

## Technologies

- Python  
- `pandas`, `numpy` for data handling  
- `yfinance` for price data  
- `scipy` for numerical optimization and clustering  
- `matplotlib` for plots and heatmaps  

## How to Run

1. Open the notebook in JupyterLab (or another Jupyter environment).  
2. Run all cells from top to bottom.  
3. The following figures are generated automatically:
   - `correlation_heatmap.png`  
   - `covariance_heatmap.png`  
   - `equity_curve.png`  

## Possible Extensions

- Out-of-sample walk-forward testing of all methods  
- Transaction-cost and turnover analysis  
- Regularized or shrinkage covariance estimators  
- Alternative risk measures (CVaR, drawdown)  
- Adding factor-based views or Black–Litterman on top of the Markowitz framework

