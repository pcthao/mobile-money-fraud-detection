# Fraud Detection for Mobile-Money Transactions

Detecting account-takeover fraud in 3.4M simulated transactions (PaySim), comparing a rules baseline against logistic regression and XGBoost, with leak-free features, a time-based evaluation, and SHAP reason codes.

## Highlights
- Found two label leaks that gave falsely perfect scores, and excluded them
- At the rules' false-positive rate, raised recall from 96.0% to 99.6% on unseen days
- Explained individual alerts with SHAP reason codes

## Data
Generated with [PaySim](https://github.com/EdgarLopezPhD/PaySim); see [`DATA_GENERATION.md`](DATA_GENERATION.md) for version, seed, and settings. 3.4M transactions over 30 days, 1,368 fraud cases (0.04%). Every fraud follows one pattern: a victim's balance is **transferred** to a mule account, which **cashes it out**.

## Approach
1. **Scoping:** fraud occurs only in TRANSFER and CASH_OUT, so analysis uses those 1.0M transactions (0.14% fraud).
2. **Leakage removed:**
   - *Drained balances:* every fraud empties the sender's account exactly, and no legitimate transaction does (100% precision and recall from one rule). Balance columns are excluded.
   - *Mule IDs:* "CC" account IDs are simulator-assigned mules. IDs are used only for grouping, never as features.
3. **Time-based split:** train on days 0–23, test on days 24–29 (252 fraud cases). The test period has far less legitimate activity (2.4% vs. 0.11% fraud), so models are compared on **recall and false-positive rate**, which don't depend on the fraud rate.
4. **Rules baseline** (thresholds set on train only): CASH_OUT over $508K and TRANSFER over $2.6M. Test: 96% recall at 0.62% FPR. Amount alone can't separate fraud transfers from large legitimate ones.
5. **Leak-free features**, using only transactions *before* the current one: transaction amount and type, sender's prior transaction count, amount relative to the sender's average, and **how many times the recipient has received money before**.
6. **Models:** logistic regression (log-transformed, standardized inputs, class weighting) and XGBoost (raw features, `scale_pos_weight`).

## Results
Test recall at a fixed false-positive rate:

| Method | FPR ≤ 0.62% | FPR ≤ 0.2% |
|---|---|---|
| Rules | 96.0% | n/a |
| Logistic regression, 4 features | 96.0% | 76.6% |
| XGBoost, 4 features | 96.0% | 83.0% |
| **Logistic regression, 5 features** | **99.6%** | **99.6%** |
| **XGBoost, 5 features** | **99.6%** | **99.2%** |

- Recipient history made the difference, moving both models from matching the rules to beating them.
- With five features, the models tie, so the simpler, more explainable logistic regression is the practical choice.
- Unlike rules, models can be tuned to any false-alarm budget.

## Caveats
- **Synthetic data:** results don't represent real-world performance.
- **Mules are always brand new in PaySim,** which makes history features unusually clean. Real mule accounts are often older.

## Future work
- Choose the alert threshold by expected cost on a held-out validation period

## Repository
```
├── README.md
├── DATA_GENERATION.md
├── 01_eda_and_baseline.ipynb     # scoping, leak checks, time split, rules baseline
├── 02_features.ipynb             # history features
├── 03_models.ipynb               # logistic regression, XGBoost, ROC comparison, SHAP 
└── data/                         # generated PaySim output (not committed; see DATA_GENERATION.md)
```

**Tools:** Python, pandas, scikit-learn, XGBoost, SHAP, Matplotlib
