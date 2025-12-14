# Quantile Regression - Concepts

Predict any percentile of the target distribution, not just the mean, enabling asymmetric risk management.

---

## Why This Matters

Standard regression (OLS, gradient boosting) minimizes squared error, producing mean predictions. The mean is optimal only when overprediction and underprediction are equally costly. In risk-sensitive domains (insurance, fraud, safety), underprediction of extreme cases causes disproportionate harm. Quantile regression lets you choose where on the conditional distribution to predict, trading accuracy for appropriate conservatism.

---

## Pinball Loss (Quantile Loss)

### What It Is

The asymmetric loss function that quantile regression minimizes. For quantile τ:

```
L_τ(y, ŷ) = τ * max(y - ŷ, 0) + (1 - τ) * max(ŷ - y, 0)
```

Underprediction (y > ŷ) is penalized by factor τ. Overprediction (ŷ > y) is penalized by factor (1 - τ).

### What It Measures

The weighted cost of prediction errors, where the weight depends on direction. At τ=0.90, underprediction costs 9x more than overprediction.

### Use When

Your business loss is asymmetric: missing a high-damage guest costs more than overestimating a low-damage guest.

### Limitation

Pinball loss does not directly minimize MAE or RMSE. Higher quantiles will always have higher absolute error than median regression.

---

## Conditional Quantile vs. Unconditional Percentile

### What It Is

Quantile regression estimates the τ-th percentile of y given X (conditional). This differs from filtering to the top τ% of outcomes (unconditional).

### What It Measures

The value below which τ% of outcomes fall, for each specific combination of input features. A Q90 prediction varies by observation based on X.

### Use When

You need predictions that account for feature-specific risk, not a single threshold for all cases.

### Limitation

Requires sufficient data across the feature space. Sparse regions produce unreliable quantile estimates.

---

## Bias_High Metric

### What It Is

Mean prediction error for high-value cases only (typically top 10% of actuals):

```
Bias_High = mean(ŷ - y) for observations where y >= P90(y)
```

### What It Measures

Systematic over or underprediction for extreme cases. Negative values indicate conservative overestimation.

### Use When

Your primary concern is performance on tail cases, not average performance across all observations.

### Limitation

Sensitive to threshold choice. P90 is common but arbitrary. Also ignores error magnitude distribution within the high group.

---

## Decision Framework

```
Is underprediction costlier than overprediction?
    |
    +-- No --> Use Q50 (median regression)
    |
    +-- Yes --> What is your cost ratio (under:over)?
                |
                +-- 2:1 to 3:1 --> Start with Q75, check Bias_High
                |
                +-- 4:1 to 9:1 --> Start with Q90, verify coverage
                |
                +-- >9:1 --> Use Q95, accept high MAE
                            |
                            Is Bias_High still positive?
                            |
                            +-- Yes --> Move to higher quantile
                            +-- No --> Current quantile achieves conservatism
```

---

## Interpreting Results

| Metric | Value | Meaning |
|--------|-------|---------|
| MAE | 276 (Q50) vs 516 (Q90) | Higher quantiles increase absolute error |
| Bias_High | +463 (Q50) | Model underpredicts high cases by 463 on average |
| Bias_High | -69 (Q90) | Model overpredicts high cases by 69 on average |
| Coverage | 0.87 for Q90 | 87% of actuals fall below predictions (expect ~90%) |
| Std% | 24.7 (Q90) | Relative standard deviation of errors |

---

## Common Pitfalls

### 1. Confusing Quantile Regression with Filtering

**Mistake:** Believing that Q90 regression is equivalent to filtering to the top 10% of outcomes and training on that subset.

**Why it's wrong:** Filtering discards 90% of your data and models only extreme cases. Quantile regression uses all data to model the 90th percentile of the conditional distribution. These produce fundamentally different predictions and generalization behavior.

**Fix:** Understand that quantile regression models the full distribution. The quantile parameter controls the loss function asymmetry, not data selection.

---

### 2. Ignoring Coverage Calibration

**Mistake:** Assuming a Q90 model automatically has 90% of actual values below predictions without verification.

**Why it's wrong:** Model miscalibration, distribution shift, or poor feature engineering can cause actual coverage to diverge from the target quantile. A "Q90" model with 70% coverage is not conservative.

**Fix:** Always compute empirical coverage on holdout data: `coverage = mean(y_actual <= y_pred)`. If coverage deviates more than 5 percentage points from target, investigate calibration.

---

### 3. Selecting Quantile Without Defining Cost Asymmetry

**Mistake:** Choosing Q90 "because it's conservative" without quantifying the actual cost ratio between underprediction and overprediction.

**Why it's wrong:** The optimal quantile depends on your loss function. If underprediction costs 2x overprediction, Q75 may suffice. If it costs 20x, even Q95 might be insufficient. Arbitrary selection leads to either unnecessary accuracy loss or inadequate risk coverage.

**Fix:** Estimate the business cost ratio first. Use: `optimal_quantile ≈ cost_under / (cost_under + cost_over)`. For 9:1 cost ratio, optimal τ ≈ 0.90.

---

## Key Takeaways

1. Quantile regression trades accuracy (MAE) for asymmetric risk control by penalizing underprediction more heavily
2. The optimal quantile is determined by your cost ratio, not arbitrary conservatism
3. Always validate coverage calibration: Q90 should have ~90% of actuals below predictions
4. Bias_High turning negative indicates the model has achieved conservatism for extreme cases
