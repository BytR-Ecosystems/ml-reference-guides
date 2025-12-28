# Data Preparation & Outlier Management Guide

## Core Principle: There Is No Golden Rule

**The handling of outliers depends entirely on:**
1. Your prediction objective (what outcome you're modeling)
2. The business context (are extreme values meaningful?)
3. The model architecture (robust vs. sensitive algorithms)
4. Whether outliers represent errors or genuine phenomena

---

## Decision Framework for Outlier Handling

### Step 1: Understand the Nature of Outliers

| Type | Description | Typical Action |
|------|-------------|----------------|
| **Data Errors** | Measurement mistakes, entry errors, system glitches | Remove or correct |
| **True Extremes** | Rare but valid observations (e.g., VIP customers, unusual events) | Often retain |
| **Domain Violations** | Values outside physically possible ranges | Remove |
| **Influential Edge Cases** | Observations that carry important predictive signal | Retain with care |

### Step 2: Ask the Right Questions

```
┌─────────────────────────────────────────────────────────────┐
│  Is the outlier physically/logically possible?              │
│  ├─ NO  → Remove or investigate data pipeline               │
│  └─ YES → Continue...                                       │
│                                                             │
│  Does the outlier represent your target population?         │
│  ├─ NO  → Consider removal (out of scope)                   │
│  └─ YES → Continue...                                       │
│                                                             │
│  Is predicting these extreme cases important for your goal? │
│  ├─ YES → RETAIN (use robust methods)                       │
│  └─ NO  → Consider capping, transformation, or removal      │
└─────────────────────────────────────────────────────────────┘
```

---

## Case Study: Hotel Guest Profit Prediction

### Context
Predicting guest profitability to inform marketing and selection decisions.

### Key Features with Extreme Values

| Feature | P99 | Max | Ratio (Max/P99) |
|---------|-----|-----|-----------------|
| `profit_am` | 29,662 | 100,577 | 3.4x |
| `profit_last_am` | 7,454 | 150,537 | 20.2x |
| `claims_am` | 2,632 | 90,587 | 34.4x |
| `shop_am` | 6,834 | 12,098 | 1.8x |
| `damage_am` | 2,496 | 14,866 | 6.0x |
| `nights_booked` | 162 | 375 | 2.3x |

### Decision: Retain Outliers

**Reasoning:**
- High-spending guests are **exactly who we want to identify**
- Removing them would bias the model toward predicting "average" guests
- These are rare but **plausible** observations, not errors
- The business value lies in finding these exceptional cases

### Mitigation Strategies Used

Instead of removing outliers, use robust modeling approaches:

1. **Loss Functions**
   - MAE instead of MSE (less sensitive to extremes)
   - Quantile loss for specific percentile predictions
   - Huber loss as a compromise

2. **Transformations** (where appropriate)
   - Log-transform for highly skewed features
   - Box-Cox or Yeo-Johnson transformations

3. **Robust Algorithms**
   - Tree-based models (XGBoost, LightGBM, Random Forest)
   - Quantile regression

---

## Common Outlier Handling Techniques

### 1. Detection Methods

```python
# IQR Method
Q1, Q3 = df[col].quantile([0.25, 0.75])
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Z-Score Method
from scipy import stats
z_scores = np.abs(stats.zscore(df[col]))
outliers = z_scores > 3

# Percentile Method
p01, p99 = df[col].quantile([0.01, 0.99])
```

### 2. Treatment Options

| Method | When to Use | Pros | Cons |
|--------|-------------|------|------|
| **Removal** | Clear data errors | Clean dataset | Loss of information |
| **Winsorization** | Reduce influence, keep pattern | Preserves data points | Arbitrary threshold |
| **Capping** | Hard business limits | Interpretable | Information loss |
| **Log Transform** | Multiplicative relationships | Normalizes distribution | Changes interpretation |
| **Keep As-Is** | Outliers are informative | Full information | May need robust methods |

### 3. Winsorization Example

```python
from scipy.stats import mstats

# Winsorize at 1st and 99th percentile
df['feature_winsorized'] = mstats.winsorize(df['feature'], limits=[0.01, 0.01])
```

---

## Validation: Did Outlier Handling Help?

### Always Test Your Assumptions

```python
# Compare model performance with different outlier treatments
results = {
    'raw': train_and_evaluate(X, y_raw),
    'winsorized': train_and_evaluate(X, y_winsorized),
    'log_transformed': train_and_evaluate(X, np.log1p(y_raw)),
    'outliers_removed': train_and_evaluate(X_clean, y_clean)
}
```

### Case Study Result

| Target Version | MAE |
|----------------|-----|
| Raw (with outliers) | 386 |
| Winsorized (P1-P99) | 385 |
| **Difference** | **+1** |

**Conclusion:** Winsorization provided no meaningful improvement, confirming that extreme values should be retained.

---

## Red Flags: When Outliers SHOULD Be Removed

1. **Values violate domain constraints**
   - Negative ages, prices above $999,999,999
   
2. **Clear data pipeline issues**
   - Default/placeholder values (e.g., -1, 9999, "NULL")
   
3. **Impossible temporal patterns**
   - Future dates in historical data
   
4. **Inconsistent with related fields**
   - 0 nights booked but positive spending

---

## Summary: Context-Driven Decision Making

| Scenario | Recommended Approach |
|----------|---------------------|
| **Predicting rare high-value events** | Keep outliers, use robust methods |
| **Classification with imbalanced classes** | Outliers may define minority class—retain |
| **Anomaly detection** | Outliers ARE the target—definitely retain |
| **Average behavior modeling** | Consider capping or transformation |
| **High sensitivity to extreme errors** | Validate data quality, consider removal |

---

## Key Takeaways

> **"The best outlier strategy is the one that serves your specific prediction goal."**

1. **No universal rule exists** — context determines everything
2. **Ask "why" before "how"** — understand the outlier's origin
3. **Validate empirically** — test different approaches on your metric
4. **Document your reasoning** — future you will thank present you
5. **Prefer information preservation** — when in doubt, use robust methods rather than deletion

---

*Reference guide created for ML Project: Smurf Hotel Guest Profitability Prediction*  
*Last updated: December 2025*
