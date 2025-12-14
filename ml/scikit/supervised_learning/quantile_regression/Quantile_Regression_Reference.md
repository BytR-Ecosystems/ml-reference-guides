# Quantile Regression - Quick Reference

Predict conditional quantiles instead of the conditional mean for asymmetric loss scenarios.

---

## Basic Syntax

```python
from sklearn.linear_model import QuantileRegressor
from sklearn.model_selection import cross_val_predict
import numpy as np

# Train a single quantile model
model = QuantileRegressor(quantile=0.90, alpha=0.0, solver='highs')
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# Train multiple quantiles for comparison
quantiles = [0.50, 0.75, 0.85, 0.90, 0.95]
predictions = {}

for q in quantiles:
    qr = QuantileRegressor(quantile=q, alpha=0.0, solver='highs')
    qr.fit(X_train, y_train)
    predictions[f'Q{int(q*100)}'] = qr.predict(X_test)
```

---

## Key Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `quantile` | float | Target quantile to predict. Range: (0, 1). Default=0.5 (median) |
| `alpha` | float | L1 regularization strength. Default=1.0. Set to 0.0 for no regularization |
| `solver` | str | Optimization algorithm. Default='highs'. Options: 'highs', 'highs-ds', 'highs-ipm', 'interior-point' |
| `fit_intercept` | bool | Whether to fit intercept term. Default=True |

---

## Quantile Selection Matrix

| Quantile | Use Case | Trade-off |
|----------|----------|-----------|
| Q50 | Minimize absolute error | Symmetric loss, underestimates extremes |
| Q75 | Moderate conservatism | Higher MAE, reduces underprediction |
| Q85 | Near-neutral bias | Balanced for moderate risk aversion |
| Q90 | Conservative estimates | Overestimates high cases, acceptable MAE increase |
| Q95 | Strong conservatism | High MAE, use only when underprediction is very costly |

---

## Example: Comparing Quantiles with Metrics

```python
from sklearn.linear_model import QuantileRegressor
from sklearn.model_selection import cross_val_predict
import numpy as np
import pandas as pd

def evaluate_quantile(y_true, y_pred, high_threshold_percentile=90):
    """Calculate MAE and bias for high-value cases."""
    mae = np.mean(np.abs(y_true - y_pred))
    
    # Bias for high-value cases (above threshold)
    threshold = np.percentile(y_true, high_threshold_percentile)
    high_mask = y_true >= threshold
    bias_high = np.mean(y_pred[high_mask] - y_true[high_mask])
    
    return {'MAE': mae, 'Bias_High': bias_high}

# Compare quantiles
quantiles = [0.50, 0.75, 0.85, 0.90, 0.95]
results = []

for q in quantiles:
    model = QuantileRegressor(quantile=q, alpha=0.0, solver='highs')
    y_pred_cv = cross_val_predict(model, X, y, cv=5)
    metrics = evaluate_quantile(y, y_pred_cv)
    metrics['Quantile'] = f'Q{int(q*100)}'
    results.append(metrics)

comparison_df = pd.DataFrame(results)
print(comparison_df)
```

---

## Example: Coverage Calibration Check

```python
def check_coverage(y_true, y_pred, target_quantile):
    """Verify model calibration: actual coverage should match target quantile."""
    actual_coverage = np.mean(y_true <= y_pred)
    expected_coverage = target_quantile
    calibration_gap = actual_coverage - expected_coverage
    
    return {
        'Target': target_quantile,
        'Actual': actual_coverage,
        'Gap': calibration_gap,
        'Calibrated': abs(calibration_gap) < 0.05
    }

# Example usage
coverage = check_coverage(y_test, y_pred_q90, target_quantile=0.90)
print(f"Coverage: {coverage['Actual']:.2%} (target: {coverage['Target']:.0%})")
```

---

## Common Pitfalls

| Issue | Symptom | Fix |
|-------|---------|-----|
| Solver warnings with alpha=0 | Convergence errors or slow fitting | Use solver='highs' explicitly |
| Crossing quantiles | Q75 predictions > Q90 predictions for some samples | Train joint model or apply isotonic regression post-hoc |
| Miscalibrated coverage | Q90 model covers only 75% of actuals | Check feature distribution shift, add regularization |

---

## Key Takeaways

1. Set `alpha=0.0` for unregularized quantile regression, use `solver='highs'` for stability
2. Higher quantiles increase MAE but reduce underprediction of extreme values
3. Always validate with coverage check: Q90 should have ~90% of actuals below predictions
