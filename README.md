# Yield Curve Prediction with the CIR Model

Predicting the full zero-coupon yield curve (6-month to 30-year) from the
3-month rate alone, using the Cox–Ingersoll–Ross short-rate model and two
extensions. Everything is in one notebook: [`cir24114011.ipynb`](cir24114011.ipynb).

## What the notebook does

1. **Data engineering** — cleans daily yield data (train: 2016-05-19 → 2024-04-26, 1,976 rows; test: 2024-04-29 → 2026-04-29, 495 rows), handles non-trading-day gaps with a variable time step, and runs ADF and KPSS stationarity tests.
2. **CIR framework** — the SDE, the Feller condition, and the closed-form bond price, with `A(τ)` and `B(τ)` computed in the log domain for numerical stability.
3. **Calibration, three ways** — WLS/OLS on the discretised SDE, maximum likelihood on the exact non-central chi-squared transition density, and panel least squares on the cross-section of yields (differential evolution followed by Nelder–Mead).
4. **Extensions** — a rate-adaptive CIR++ deterministic shift, and a jump-diffusion CIR (Duffie–Pan–Singleton, 2000) calibrated on the post-2022 high-rate regime.
5. **Diagnostics** — rolling-window R², residual autocorrelation and Q-Q plots, and a train-versus-test regime comparison.

## Results (out of sample)

| Model | R² (variance-weighted) | RMSE (bps) |
|---|---|---|
| Jump-diffusion CIR | **0.9426** | **16.05** |
| Base CIR (panel LS) | 0.8959 | 21.61 |
| Dynamic CIR++ | 0.6224 | 41.15 |

Per-maturity R² for the best model: 0.9933 (6M), 0.9772 (9M), 0.9451 (1Y), 0.7107 (2Y).

Things that did not work, or limit the result:

- **CIR++ made things worse.** The spread regression fitted in training did not carry over to the test period, where the 3M–2Y spread is inverted.
- **Time-series calibration alone is not enough.** Parameters fitted to the 3M series only (WLS/OLS, MLE) do not pin down the shape of the term structure; the MLE fit scores R² = 0.2592 out of sample.
- **The 2Y maturity is capped.** The 3M and 2Y rates have correlation 0.908 in the test period, so no single-factor model driven by the 3M rate can exceed R² ≈ 0.82 there.
- **5Y to 30Y predictions are not validated.** The test set has no actual yields for those maturities.

## Running it

```bash
pip install numpy pandas scipy statsmodels scikit-learn matplotlib
jupyter notebook cir24114011.ipynb
```

The notebook reads `train_data.csv`, `test_data.csv` and `test_data_3M.csv`
from the working directory. These data files are not included in this
repository. The committed notebook keeps its executed outputs, so the
results can be read without re-running it.

## References

- Cox, Ingersoll, Ross (1985), *A theory of the term structure of interest rates*, Econometrica 53(2).
- Duffie, Pan, Singleton (2000), *Transform analysis and asset pricing for affine jump-diffusions*, Econometrica 68(6).
- Brigo, Mercurio (2006), *Interest Rate Models — Theory and Practice*, 2nd ed., Springer.
