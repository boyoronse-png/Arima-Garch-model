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

## Results
### AAPL Log Returns
![AAPL Returns](images/aapl_returns.png)

The log return series shows volatility clustering, where periods of small movements are followed by bursts of larger fluctuations.

### AAPL Conditional Volatility
![AAPL Volatility](images/aapl_volatility.png)

The conditional volatility estimated from the eGARCH model highlights spikes during periods of market stress, particularly around the COVID-19 market disruption in 2020.
The estimated models successfully capture the key characteristics of financial return series.

The return series for AAPL shows periods of relatively low variability punctuated by bursts of high volatility. This behaviour is consistent with volatility clustering, where large price movements tend to be followed by further large movements.

The estimated conditional volatility from the eGARCH model demonstrates persistent volatility dynamics. Shocks to the market increase volatility sharply, after which volatility gradually decays toward its long-run level.

Diagnostic tests, including Ljung–Box tests on standardized residuals and squared residuals, suggest that the models capture most of the serial dependence and volatility structure present in the data.

## Interpretation of Charts

The log return plot illustrates the irregular nature of financial returns. Most observations fluctuate near zero, but occasional large shocks appear throughout the sample.

The conditional volatility plot generated from the eGARCH model clearly illustrates volatility clustering. Several spikes are visible across the sample period, with the largest spike occurring around 2020 during the COVID-19 financial market disruption.

These spikes reflect periods where uncertainty in financial markets increased significantly, causing large fluctuations in asset prices. The gradual decline following these spikes indicates the persistence of volatility, a key property that GARCH models are designed to capture.

## Conclusion

Overall, the ARIMA–GARCH framework provides an effective approach for modelling financial return volatility. The models capture volatility clustering and the persistence of shocks, demonstrating how conditional variance evolves through time. Such models are widely used in financial econometrics for risk management, forecasting volatility, and understanding market behaviour.
