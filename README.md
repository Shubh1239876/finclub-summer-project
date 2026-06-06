# CIR Interest Rate Modelling
### Finance Club, IIT Roorkee — Open Projects 2026

Implementation, calibration, and extension of the **Cox-Ingersoll-Ross (CIR)** stochastic short-rate model on real yield curve data. The core challenge: use only the 3-month yield as input and reconstruct the entire yield curve from first principles.

---

## What this does

The notebook works through five stages:

**A. Data preprocessing** — loads three CSVs of daily zero-coupon bond yields, strips whitespace from column names, detects and clips outliers using a rolling z-score window, and validates the cleaned data.

**B. CIR model** — implements the closed-form bond pricing formula from scratch, including the `B(τ)` and `ln A(τ)` components. Calibrates the three parameters (κ, θ, σ) by minimising cross-sectional MSE across all maturities and all training days simultaneously.

**C. Prediction** — for each day in the test set, feeds *only* the 3M yield into the calibrated model and reconstructs yields at 6M, 9M, 1Y, and 2Y. Evaluates against held-out actuals.

**D. Extension** — recalibrates the model on the most recent 500 trading days instead of the full history. This adapts θ and κ to the post-2022 rate-hike regime, improving the pooled R² from 0.893 to 0.914.

**E. Critical analysis** — covers where and why the model fails, what the Feller condition means in practice, why the 2Y tenor is the hardest to fit, and what a two-factor or jump-diffusion extension would add.

---

## Results

| Model | Pooled R² | 6M | 9M | 1Y | 2Y |
|-------|-----------|----|----|----|----|
| Base CIR (1,976-day window) | **0.8932** | 0.994 | 0.968 | 0.910 | 0.390 |
| Extended CIR (500-day window) | **0.9136** | 0.989 | 0.964 | 0.918 | 0.566 |

Evaluation threshold: pooled R² > 0.85. Both models pass.

Calibrated parameters (base model): κ = 0.166, θ = 2.44%, σ ≈ 0. Half-life of rate shocks: ~4.2 years. Feller condition satisfied.

---

## Dataset

Three files (not included in this repo — upload them to Colab before running):

| File | Rows | Contents |
|------|------|----------|
| `train_data.csv` | 1,976 | Daily yields, 9 tenors (3M–30Y), May 2016–Apr 2024 |
| `test_data.csv` | 495 | Daily yields, 5 tenors (3M–2Y), Apr 2024–Apr 2026 |
| `test_data_3M.csv` | 495 | 3M yield only — the sole input allowed at prediction time |

Maturity tenors: 3M, 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, 30Y (training); 3M–2Y (test).

---

## Repo structure

```
├── CIR_Interest_Rate_Modelling.ipynb   # main notebook
└── README.md
```

The notebook is self-contained. All code, math explanations, plots, and analysis live inside it.

---

## Running it

1. Open `CIR_Interest_Rate_Modelling.ipynb` in Google Colab
2. Upload the three CSV files to the Colab session
3. Runtime → Run all

Dependencies are all standard: `numpy`, `pandas`, `scipy`, `scikit-learn`, `matplotlib`. No installs needed on Colab.

---

## Key design decisions

**Cross-sectional calibration over time-series MLE** — fitting κ, θ, σ to minimise errors across all maturities simultaneously gives economically meaningful parameters and directly optimises for the yield-curve reconstruction task. Time-series MLE on the 3M rate alone produces an unstable negative κ because the short rate is near-unit-root over an 8-year sample.

**500-day recalibration window** — the training data spans a low-rate era (avg 3M ≈ 1.7%) while the test period is a high-rate regime (3M = 3–5%). A static θ calibrated on the full history pulls predictions downward. Recalibrating on the most recent 500 days lets θ adapt to the current regime. The sensitivity analysis confirms this choice is robust — anything above ~200 days clears the 0.85 threshold.

**Why 2Y is hardest** — CIR is a single-factor model: every maturity is a deterministic function of one scalar, the current short rate. The 3M–2Y spread is an independent degree of freedom (yield curve slope), driven by expectations about where rates will be in two years rather than where they are today. A two-factor model (Longstaff-Schwartz 1992) would handle this.

---


