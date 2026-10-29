# CAPM Beta Estimation on Random Stocks (Yahoo Finance)

A Jupyter notebook that estimates CAPM betas for a random sample of US stocks, using monthly data from Yahoo Finance. It compares each stock's realized excess returns with the return CAPM predicts, and plots the results on a Security Market Line.

## Overview

The Capital Asset Pricing Model (CAPM) says a stock's expected excess return is proportional to its sensitivity to market movements, measured by beta:

$$E[R_i] - R_f = \beta_i \,(E[R_m] - R_f)$$

This notebook estimates $\beta_i$ for each stock by regressing its monthly excess returns on the S&P 500's monthly excess returns:

$$R_{i,t} - R_{f,t} = \alpha_i + \beta_i \,(R_{m,t} - R_{f,t}) + \varepsilon_{i,t}$$

Under CAPM, $\alpha_i$ should be close to zero. A large alpha suggests the stock earned more or less than CAPM predicts over the sample period.

## Features

- Randomly samples a set of stocks from a predefined universe of 20 large-cap US tickers (reproducible via a seed).
- Downloads adjusted closing prices for the stocks, the S&P 500 (`^GSPC`), and the 13-week T-bill yield (`^IRX`) as the risk-free proxy.
- Converts prices to monthly returns and annualizes the risk-free rate.
- Estimates alpha, beta, R², and the p-value for beta via OLS.
- Compares realized excess returns with CAPM-predicted excess returns and reports pricing errors.
- Plots the Security Market Line with each stock's realized average return.
- Reports 95% confidence intervals for beta and flags whether beta is significant at the 5% level.
- Prints a plain-language interpretation of each beta.

## Requirements

- Python 3.9 or later (tested with Python 3.13)
- Jupyter Notebook or JupyterLab
- Internet access (for Yahoo Finance)

Python packages:

| Package | Purpose |
|---|---|
| `yfinance` | Download price data from Yahoo Finance |
| `pandas` | Data handling and resampling |
| `numpy` | Numerical operations |
| `matplotlib` | Plotting the Security Market Line |
| `statsmodels` | OLS regression, p-values, confidence intervals |
| `scipy` | Statistical utilities |

Install everything with:

```bash
pip install yfinance pandas numpy matplotlib statsmodels scipy
```

## Installation and Usage

1. Clone or download the notebook file.
2. Install the requirements listed above.
3. Launch Jupyter:
   ```bash
   jupyter notebook
   ```
4. Open the notebook and run the cells in order, from top to bottom.
5. To analyze different stocks, change `RANDOM_SEED` or `N_STOCKS` in Cell 2 and re-run from Cell 3 onward.

## Notebook Structure

| Cell | Description |
|---|---|
| 1 | Installs (optional) and imports libraries |
| 2 | Sets parameters: date range, market and risk-free tickers, sample size, seed |
| 3 | Randomly selects stocks from the universe |
| 4 | Downloads daily prices from Yahoo Finance |
| 5 | Resamples to month-end and computes monthly returns and excess returns |
| 6 | Estimates alpha, beta, R², and beta p-value for each stock |
| 7 | Builds a table of realized vs. CAPM-predicted excess returns and pricing errors |
| 8 | Plots the Security Market Line with realized returns |
| 9 | Computes 95% confidence intervals and significance flags for beta |
| 10 | Prints an interpretation of each beta and the market risk premium |

## Configuration

All user-adjustable settings are in Cell 2:

| Variable | Default | Description |
|---|---|---|
| `START_DATE` | `"2019-01-01"` | Start of the sample period |
| `END_DATE` | `"2024-12-31"` | End of the sample period |
| `MARKET_TICKER` | `"^GSPC"` | Market proxy (S&P 500) |
| `RISK_FREE_TICKER` | `"^IRX"` | Risk-free proxy (13-week T-bill yield, in %) |
| `N_STOCKS` | `5` | Number of stocks to sample |
| `RANDOM_SEED` | `42` | Seed for reproducible sampling |

The stock universe (Cell 3) includes AAPL, MSFT, GOOGL, AMZN, META, NVDA, TSLA, JPM, JNJ, XOM, PG, KO, WMT, DIS, BAC, INTC, CSCO, PFE, NFLX, and ADBE. Edit the `universe` list to change it.

## Methodology

**Returns.** Monthly returns are computed from month-end closing prices. Simple returns are used, not log returns.

**Risk-free rate.** `^IRX` quotes an annualized yield in percent. It is divided by 100 to get a decimal, then by 12 to get a monthly rate.

**Beta estimation.** For each stock, an OLS regression is run with an intercept on monthly excess returns. The slope is beta, and the intercept is alpha. The regression is written with plain NumPy arrays so the parameters are read by position, which avoids label-lookup errors.

**CAPM benchmark.** The expected excess return is beta times the average monthly market excess return, annualized by multiplying by 12. The pricing error is realized minus expected excess return.

**Significance.** Beta's p-value and 95% confidence interval come from the standard OLS errors. A beta is flagged as significant if its p-value is below 0.05.

## Interpreting the Output

- **Beta > 1**: the stock moves more than the market on average (higher systematic risk).
- **0 < Beta < 1**: the stock moves less than the market (defensive).
- **Beta ≤ 0**: the stock is weakly or negatively correlated with the market.
- **R²**: the share of the stock's return variation explained by the market. Low R² means most movement is stock-specific.
- **Pricing error (alpha)**: positive values mean the stock beat CAPM's prediction over the sample; negative values mean it lagged.
- **Points on the SML**: stocks above the line outperformed their beta-implied return, and stocks below underperformed.

## Limitations

- **Small sample.** Five stocks over about six years gives noisy estimates. Confidence intervals are often wide.
- **Single-factor model.** CAPM uses only market risk. Factors such as size, value, and momentum are ignored.
- **Survivorship and selection bias.** The universe is a hand-picked list of large, currently successful companies.
- **Past beta is not future beta.** Betas change over time, and historical estimates may not hold.
- **Market proxy.** The S&P 500 is an imperfect stand-in for the true market portfolio.
- **Data quality.** Yahoo Finance data is free and can contain gaps, splits, or adjustment errors. Check results against another source before relying on them.
- **Not investment advice.** This notebook is for learning and research.

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `KeyError: '^GSPC'` | Column labels differ from expected | Run `print(prices.columns.tolist())` and adjust the labels |
| Empty DataFrame | Ticker typo or no internet | Check tickers and connection |
| `yfinance` returns MultiIndex columns | Newer `yfinance` versions | Use `prices = raw["Close"]` and check column levels |
| Rate-limit errors | Too many requests | Wait a few minutes and re-run Cell 4 |

## Possible Extensions

- Add the Fama-French three-factor model (size and value factors) and compare alphas.
- Use rolling 36-month windows to see how beta changes over time.
- Expand the universe to the full S&P 500 constituents list and run a cross-sectional test.
- Add a Fama-MacBeth regression to test whether beta explains average returns across many stocks.
- Export the results table to CSV for further analysis.

## References

- Sharpe, W. F. (1964). Capital asset prices: A theory of market equilibrium under conditions of risk. *Journal of Finance*.
- Lintner, J. (1965). The valuation of risk assets and the selection of portfolios in stock portfolios and capital budgets. *Review of Economics and Statistics*.
- Yahoo Finance data via the `yfinance` library: https://github.com/ranaroussi/yfinance

## License

Provided for educational use. Add your preferred license before sharing publicly.
