# Arima-Garch-model
# =========================
# CLEAN + LOAD PACKAGES
# ========================

library(quantmod)
library(rugarch)
library(tseries)                                     

# =========================
# DOWNLOAD DATA
# =========================
aapl_xts <-(
  getSymbols("AAPL", from = "2010-01-01",           
             src = "yahoo", auto.assign = FALSE),                                       
  )

head(aapl_xts)
tail(aapl_xts)

# =========================
# DAILY LOG RETURNS
# =========================

aapl_adj  <- Ad(aapl_xts)                           
ret_AAPL  <- na.omit(diff(log(aapl_adj)))
colnames(ret_AAPL) <- "r"                           

head(ret_AAPL)                                      
summary(ret_AAPL)                                   

# =========================
# COEFFICIENT TABLE
# =========================

coef_table <- function(fit, robust = TRUE) {
  cf  <- coef(fit)                                   # Vector of parameter estimates from the fitted model.
  se  <- sqrt(diag(vcov(fit, robust = robust)))      # Standard Error.
  t   <- cf / se                                     # t-statistic: how many SEs away from zero is the estimate?
  p   <- 2 * (1 - pnorm(abs(t)))                     # Two-tailed p-value under Normal approx.
  data.frame(Estimate = cf, `Std.Error` = se,        
             `t value` = t, `Pr(>|t|)` = p)
}

# =========================
# DIAGNOSTICS (FIT QUALITY + RESIDUAL TESTS)
# =========================
run_diagnostics <- function(fit, lags = 10) {
  cat("\n--- Info Criteria (AIC/BIC/etc.) ---\n")
  print(infocriteria(fit))                           # Returns named vector: Akaike, Bayes (BIC), Shibata, Hannan–Quinn.

  z  <- residuals(fit, standardize = TRUE)           # Standardized residuals.
  z2 <- z^2                                          # Square them: used to detect remaining volatility structure (ARCH).

  cat("\n--- Ljung-Box on standardized residuals ---\n")
  print(Box.test(z,  lag = lags, type = "Ljung-Box")) #H0: no autocorrelation up to 'lags'. High p => good (mean modeled well).

  cat("\n--- Ljung-Box on standardized squared residuals ---\n")
  print(Box.test(z2, lag = lags, type = "Ljung-Box"))# H0: no autocorrelation in z^2. High p => good (variance modeled well).

  cat("\n--- Nyblom Stability Test ---\n")
  print(nyblom(fit))                                 # Tests for parameter constancy. Report which params have low p (unstable).

  cat("\n--- Distribution GoF via PIT + KS (Uniform[0,1]) ---\n")
  pit_vals <- tryCatch(pit(fit),                     # PIT maps standardized residuals through the assumed CDF.
                       error = function(e) rep(NA_real_, length(z)))
  if (all(is.finite(pit_vals))) {                    # If PIT is available for this spec (it is for most common ones)...
    print(ks.test(pit_vals, "punif"))                # Kolmogorov–Smirnov test: H0 is Uniform(0,1). High p supports your dist.
  } else {
    cat("PIT not available for this fit; KS test skipped.\n")  # Some distributions/specs may not expose PIT; safe to skip.
  }
}

# =========================
# SANITY CHECK:(sGARCH(1,1))
# =========================
spec_test <- ugarchspec(                          
  variance.model = list(model = "sGARCH",            # sGARCH.
                        garchOrder = c(1, 1)),       # Order (1,1): one ARCH term (alpha1), one GARCH term (beta1).
  mean.model     = list(armaOrder = c(0, 0),         # No AR/MA terms in mean (pure constant mean).
                        include.mean = TRUE),     
  distribution.model = "norm"                        # Assume Normal errors initially (you can try "std" for Student-t later).
)

fit_test <- ugarchfit(spec = spec_test,              # Fit the simple model to Apple returns.
                      data = ret_AAPL)

coef_table(fit_test, robust = TRUE)                  # Print robust-inference table: estimates, SEs, t-stats, p-values.
run_diagnostics(fit_test, lags = 10)                 # Print AIC/BIC + residual/squared-residual LB + Nyblom + PIT/KS.

# =========================
# APPLE MODEL: eGARCH(4,3) + ARMA(4,3)
# =========================

spec_AAPL <- ugarchspec(                            
  variance.model = list(model = "eGARCH",            # eGARCH: log-variance model; handles asymmetry without positivity constraints.
                        garchOrder = c(4, 3)),       # eGARCH(4,3)
  mean.model     = list(armaOrder = c(4, 3),         
                        include.mean = TRUE),        
  distribution.model = "norm"                       
)

fit_AAPL <- ugarchfit(                               # Estimate the Apple model parameters by ML under the chosen spec.
  spec = spec_AAPL,                                  # Use the spec we just defined.
  data = ret_AAPL,                                   # Use Apple log returns as input.
  solver = "hybrid",                                
  fit.control = list(scale = 1)                      # Scaling can help numerical stability for likelihood optimization.
)

cat("\n=========================\nAPPLE: eGARCH(4,3) with ARMA(4,3), normal\n=========================\n")
cat("\n--- Coefficients (robust SEs) ---\n")
print(coef_table(fit_AAPL, robust = TRUE))           
run_diagnostics(fit_AAPL, lags = 10)                 #(AIC/BIC, tests, etc.).

# =========================
# CAT: sGARCH(1,1), MEAN (0,0)
# =========================

cat_xts <- getSymbols("CAT", from = "2010-01-01",    
                      src = "yahoo", auto.assign = FALSE)

cat_adj  <- Ad(cat_xts)                              # Adjusted Close prices for CAT.
ret_CAT  <- na.omit(diff(log(cat_adj)))              # Daily log returns for CAT.
colnames(ret_CAT) <- "r"                             # Renamed the return column "r"

spec_CAT <- ugarchspec(                            
  variance.model = list(model = "sGARCH",            # Standard GARCH variance.
                        garchOrder = c(1, 1)),     
  mean.model     = list(armaOrder = c(0, 0),         
                        include.mean = FALSE),       
  distribution.model = "norm"                      
)

fit_CAT <- ugarchfit(                                # Fit CAT model to its returns.
  spec = spec_CAT, data = ret_CAT,
  solver = "hybrid", fit.control = list(scale = 1)
)

cat("\n=========================\nCAT: sGARCH(1,1) with mean (0,0), normal\n=========================\n")
cat("\n--- Coefficients (robust SEs) ---\n")
print(coef_table(fit_CAT, robust = TRUE))            # Robust coefficient table for CAT.
run_diagnostics(fit_CAT, lags = 10)                  # Diagnostics for CAT.


