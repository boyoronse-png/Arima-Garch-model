# ARIMA–GARCH Volatility Modelling of Equity Returns

## Project Objective

This project analyses the volatility dynamics of equity returns using ARIMA–GARCH models implemented in R. Financial return series are well known to exhibit characteristics such as volatility clustering, persistence, and large spikes during periods of market stress. The objective of this project is to estimate conditional volatility for stock returns and examine how volatility evolves over time.

Daily price data for Apple (AAPL) and Caterpillar (CAT) are obtained from Yahoo Finance. Logarithmic returns are calculated from adjusted closing prices to ensure stationarity and to capture percentage changes in asset prices.

## Data and Pre-processing

Daily adjusted closing prices are downloaded using the `quantmod` package in R. Log returns are computed using the standard transformation:

r_t = log(P_t) − log(P_{t−1})

where (P_t) represents the adjusted closing price at time (t).

Log returns are used instead of raw prices because financial returns are typically stationary, while price levels often follow non-stationary processes.

## Model Specification

To capture both the mean dynamics and time-varying volatility of the return series, an ARIMA–GARCH modelling framework is used.

For Apple (AAPL), the following specification is estimated:

Mean equation: ARMA(4,3)
Variance equation: eGARCH(4,3)

The eGARCH model is chosen because it models the logarithm of the conditional variance, allowing the model to capture asymmetric responses to positive and negative shocks without imposing non-negativity constraints.

For Caterpillar (CAT), a simpler specification is estimated:

Mean equation: ARMA(0,0)
Variance equation: sGARCH(1,1)

The GARCH framework captures the persistence of volatility by modelling the conditional variance as a function of past squared shocks and past variance.

Model estimation is performed using the `rugarch` package with maximum likelihood estimation.

## Model Results and Interpretation
## Apple (AAPL): eGARCH(4,3) with ARMA(4,3)

To capture both mean dynamics and time-varying volatility in Apple stock returns, an ARMA(4,3)–eGARCH(4,3) specification was estimated.

The ARMA component models serial dependence in the mean return process, while the eGARCH specification captures conditional heteroskedasticity and allows volatility to respond asymmetrically to market shocks.

The model parameters were estimated using maximum likelihood estimation with the rugarch package in R.

Diagnostic tests were performed on the standardized residuals to evaluate model adequacy.

## Diagnostic Tests

Several tests were applied to assess the quality of the model fit:

• Information Criteria (AIC/BIC) were used to evaluate model fit and compare specifications.
• Ljung–Box tests on standardized residuals check whether any serial correlation remains in the residuals.
• Ljung–Box tests on squared residuals test whether additional ARCH effects remain after fitting the model.
• Nyblom stability tests examine whether model parameters remain stable over the sample period.
• Probability Integral Transform (PIT) with Kolmogorov–Smirnov test assesses whether the assumed error distribution adequately fits the data.

The diagnostic results suggest that the model captures most of the serial dependence and volatility structure present in the return series.

## Volatility Behaviour

The estimated conditional volatility series shows clear volatility clustering, a well-known feature of financial return data. Periods of relatively calm market conditions are followed by bursts of high volatility.

The largest volatility spike occurs around 2020, corresponding to the COVID-19 market disruption. Following this shock, volatility gradually declines toward its long-run level, demonstrating the persistence of volatility captured by the GARCH framework.

These results indicate that the eGARCH model effectively captures the time-varying risk dynamics present in Apple stock returns.

## Caterpillar (CAT): sGARCH(1,1)

For Caterpillar stock, a simpler sGARCH(1,1) specification with no ARMA terms in the mean equation was estimated.

The mean equation assumes that returns follow a white-noise process, while the conditional variance is modelled using a standard GARCH structure.

The GARCH(1,1) specification captures volatility persistence through:

• ARCH term (α) — the impact of recent shocks on volatility
• GARCH term (β) — the persistence of past volatility

Diagnostic tests indicate that the model captures the primary volatility dynamics of the CAT return series, with volatility clustering visible during periods of market stress.

## Key Findings

The empirical results highlight several stylised facts of financial return series:

• Stock returns fluctuate around zero with little serial correlation in the mean
• Volatility is time-varying rather than constant
• Large shocks lead to temporary increases in market volatility
• Volatility decays gradually after major shocks

The ARMA–GARCH framework successfully captures these dynamics, making it a useful tool for modelling financial market risk.

## Results
### AAPL Log Returns
![AAPL Returns](images/aapl_returns.png)

The return series fluctuates around zero, which is typical for financial return data. Most daily movements are relatively small, but occasional large spikes occur during periods of market stress. These spikes represent large price movements in the stock.

The clustering of large movements indicates volatility clustering, where large shocks tend to occur close together in time.

### AAPL Conditional Volatility
![AAPL Volatility](images/aapl_volatility.png)

The conditional volatility estimated from the eGARCH model shows clear periods of heightened market uncertainty. Volatility spikes correspond to periods where financial markets experienced increased turbulence.

The largest spike appears around 2020, coinciding with the COVID-19 market disruption. After such shocks, volatility gradually declines back toward its long-run level, demonstrating the persistence of volatility captured by the GARCH model.

The log return plot illustrates the irregular nature of financial returns. Most observations fluctuate near zero, but occasional large shocks appear throughout the sample.

The conditional volatility plot generated from the eGARCH model clearly illustrates volatility clustering. Several spikes are visible across the sample period, with the largest spike occurring around 2020 during the COVID-19 financial market disruption.

These spikes reflect periods where uncertainty in financial markets increased significantly, causing large fluctuations in asset prices. The gradual decline following these spikes indicates the persistence of volatility, a key property that GARCH models are designed to capture.

## Conclusion

Overall, the ARIMA–GARCH framework provides an effective approach for modelling financial return volatility. The models capture volatility clustering and the persistence of shocks, demonstrating how conditional variance evolves through time. Such models are widely used in financial econometrics for risk management, forecasting volatility, and understanding market behaviour.
