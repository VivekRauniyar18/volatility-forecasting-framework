# Volatility Modeling & Out-of-Sample Forecasting Framework

A quantitative framework for forecasting S&P 500 volatility using multiple statistical models and evaluating their performance under a chronological out-of-sample design.

The project compares four approaches:

- Naive Persistence
- Ornstein-Uhlenbeck (OU)
- EWMA
- GARCH(1,1)

Forecasts are evaluated against 20-trading-day forward realized volatility using MAE, RMSE, Bias, and QLIKE loss. The analysis also includes quarterly expanding-window parameter re-estimation and performance comparisons across normal- and high-volatility regimes.

---

## Research Question

Can statistical volatility models improve forecasts of 20-day forward S&P 500 realized volatility relative to a simple persistence benchmark?

The project focuses not only on which model produces lower forecast errors, but also on:

- whether more complex models outperform simple persistence,
- whether periodic parameter re-estimation improves forecasting performance,
- how model performance changes during high-volatility periods,
- and whether conclusions remain consistent across different forecast loss functions.

---

## Data

Daily S&P 500 (^GSPC) market data from January 2000 through December 2025 are used to construct daily log returns.

Daily log returns are calculated as:

$$
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
$$

A 40-trading-day rolling annualized volatility series is used as the state variable for the OU model:

$$
\sigma_t = SD(r_{t-39}, \ldots, r_t)\sqrt{252}
$$

The data are divided chronologically into an 80% training sample and a 20% testing sample to preserve the time-series structure and prevent future observations from entering model estimation.

---

## Forecast Target

The primary target is 20-trading-day forward realized volatility:

$$
RV_{t,20} = \sqrt{\frac{252}{20}\sum_{i=1}^{20}r_{t+i}^{2}}
$$

Future returns are used only to construct the evaluation target and are never used as model inputs when forecasts are generated.

The final out-of-sample evaluation contains 1,280 forecast observations from October 2020 through December 2025.

---

## Models

### Naive Persistence

The current rolling volatility level is used as the forecast for future volatility.

This provides a simple benchmark for determining whether additional model complexity produces meaningful improvements.

### Ornstein-Uhlenbeck

Volatility is modeled as a mean-reverting stochastic process:

$$
dX_t = \kappa(\theta-X_t)dt+\sigma dW_t
$$

The OU transition likelihood is implemented directly and model parameters are estimated using maximum likelihood estimation.

The fitted process produced a mean-reversion half-life of approximately 1.32 years, indicating relatively slow mean reversion.

### EWMA

EWMA updates conditional variance using exponentially declining weights:

$$
\sigma_t^2 = \lambda\sigma_{t-1}^2 + (1-\lambda)r_t^2
$$

with:

$$
\lambda = 0.94
$$

This allows volatility forecasts to respond more quickly to recent market shocks.

### GARCH(1,1)

Conditional variance is modeled as:

$$
\sigma_t^2 = \omega + \alpha\epsilon_{t-1}^2 + \beta\sigma_{t-1}^2
$$

The fitted model exhibits high volatility persistence, with:

$$
\alpha+\beta \approx 0.984
$$

Multi-step variance forecasts are generated recursively and aggregated into 20-day volatility forecasts.

---

## Out-of-Sample Evaluation

Forecasts are evaluated using four metrics:

- Mean Absolute Error (MAE)
- Root Mean Squared Error (RMSE)
- Forecast Bias
- QLIKE loss

Bias is defined as:

$$
Bias = Actual - Forecast
$$

Therefore, negative bias indicates systematic overprediction of volatility.

QLIKE is calculated from realized and forecast variance as:

$$
QLIKE
=
\frac{RV}{FV}
-
\ln\left(\frac{RV}{FV}\right)
-
1
$$

where lower values indicate better forecast performance.

---

## Results

| Model | MAE | RMSE | Bias | QLIKE |
|---|---:|---:|---:|---:|
| Naive | 0.04784 | 0.07115 | -0.00499 | 0.31700 |
| OU Fixed | 0.04751 | 0.07051 | -0.00518 | 0.30928 |
| OU Quarterly | 0.04751 | 0.07050 | -0.00510 | 0.30985 |
| EWMA | 0.04507 | 0.06631 | -0.00497 | 0.29025 |
| GARCH Fixed | 0.04397 | 0.06321 | -0.01284 | 0.22884 |
| GARCH Quarterly | 0.04392 | 0.06291 | -0.01251 | 0.22994 |

GARCH produced the lowest overall MAE and RMSE, while fixed-parameter GARCH produced the lowest QLIKE loss.

However, GARCH also exhibited the largest negative forecast bias, indicating a tendency to overpredict realized volatility on average.

OU produced only a small improvement over naive persistence. This is consistent with the slow estimated mean-reversion speed of the fitted OU process.

EWMA provided a meaningful improvement over both persistence and OU while retaining a relatively simple model structure.

Quarterly expanding-window re-estimation produced only modest changes. OU performance remained nearly unchanged, while GARCH showed small improvements in MAE, RMSE, and bias. Fixed GARCH nevertheless retained a slightly lower QLIKE loss.

---

## Volatility-Regime Analysis

To examine whether model performance changes during stressed markets, the out-of-sample period is separated using the 75th percentile of realized volatility.

The resulting high-volatility threshold is approximately 17.86% annualized volatility.

| Model | Normal MAE | High-Vol MAE | Normal QLIKE | High-Vol QLIKE |
|---|---:|---:|---:|---:|
| Naive | 0.03883 | 0.07489 | 0.21134 | 0.63399 |
| OU Quarterly | 0.03847 | 0.07463 | 0.20582 | 0.62196 |
| EWMA | 0.03648 | 0.07085 | 0.20600 | 0.54300 |
| GARCH Quarterly | 0.03658 | 0.06594 | 0.17455 | 0.39609 |

Forecast errors increase substantially during high-volatility periods across all models.

The relative performance difference becomes more pronounced during these periods. GARCH produces substantially lower high-volatility MAE and QLIKE loss than the other specifications.

During normal-volatility periods, EWMA and GARCH produce very similar MAE, suggesting that some of GARCH's relative advantage is concentrated in more volatile market environments.

---

## Key Findings

1. Model complexity does not automatically produce large improvements over simple persistence.

2. OU forecasts remain close to the naive benchmark because the estimated volatility process mean-reverts slowly.

3. EWMA improves forecast accuracy while remaining comparatively simple.

4. GARCH produces the lowest overall forecast errors in this experiment but also exhibits greater systematic overprediction.

5. GARCH's relative forecasting advantage is more pronounced during high-volatility periods.

6. Quarterly parameter re-estimation provides limited improvement relative to the choice of volatility model itself.

7. Conclusions can differ slightly depending on the loss function: quarterly GARCH improves MAE and RMSE, while fixed GARCH produces slightly lower QLIKE loss.

---

## Methodology Highlights

- Chronological train/test split
- No look-ahead information in forecast generation
- 20-day forward realized-volatility target
- Maximum-likelihood estimation of OU parameters
- Recursive multi-step GARCH forecasting
- Fixed-parameter and expanding-window estimation
- Multiple forecast loss functions
- Ex-post volatility-regime analysis

---

## Limitations

The 20-day forward realized-volatility targets overlap, meaning adjacent forecast errors are serially dependent. Performance metrics should therefore be interpreted descriptively rather than as independent observations.

Realized volatility is estimated using daily close-to-close squared returns rather than high-frequency intraday data.

The Gaussian OU process can theoretically generate negative volatility states and is therefore an imperfect representation of volatility dynamics.

The OU model is estimated using a backward-looking 40-day rolling volatility proxy, while EWMA and GARCH model return variance directly.

The GARCH specification assumes normally distributed innovations. Heavy-tailed distributions or asymmetric volatility models could be explored in future work.

---

## Technologies

Python, NumPy, Pandas, SciPy, Matplotlib, yfinance, arch

---

## Repository Structure

```text
volatility-forecasting-framework/
│
├── Volatility_Modeling_&_Out-of-Sample_Forecasting_Framework.ipynb
├── README.md
└── requirements.txt
