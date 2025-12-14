# Logistic Regression — Regularization & Scoring Concepts

---

## 1. The C Parameter and Logarithmic Scale

### What is C?
`C` is the **inverse regularization strength**:
- **High C** (e.g., 10000) → Low penalty → Model can use large coefficients → Risk of overfitting
- **Low C** (e.g., 0.0001) → High penalty → Coefficients shrunk aggressively → Risk of underfitting

### Why Logarithmic Scale?
C values span orders of magnitude. Linear spacing wastes samples.

```python
# Linear: uneven coverage
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]  # All in same order of magnitude

# Logarithmic: even coverage across magnitudes
np.logspace(-4, 4, 20)  # 10^-4 to 10^4 = 0.0001 to 10000
# Produces: 0.0001, 0.0003, 0.001, 0.003, 0.01, 0.03, 0.1, 0.3, 1, 3, 10, 30, 100...
```

### Syntax
```python
np.logspace(start, stop, num)
# start: exponent of 10 for first value (10^start)
# stop: exponent of 10 for last value (10^stop)
# num: number of samples to generate

np.logspace(-4, 4, 20)  # 20 values from 10^-4 (0.0001) to 10^4 (10000)
np.logspace(-3, 3, 15)  # 15 values from 10^-3 (0.001) to 10^3 (1000)
```

---

## 2. Scoring — What the CV Loop Optimizes

### The Concept
`scoring` tells cross-validation: **"Which metric should I maximize when comparing different C values?"**

### How It Works
1. For each C value in your grid:
   - Train on K-1 folds
   - Predict on held-out fold
   - Compute scoring metric
   - Repeat for all folds
   - Average the scores
2. Select C with highest average score
3. Refit model on full training set with best C

### Different Scoring = Different Optimal C

| Scoring | Optimizes for | Best when |
|---------|---------------|-----------|
| `'roc_auc'` | Probability ranking ability | Using `predict_proba()` for expected value |
| `'recall'` | Catching all positive cases | Missing positives is costly (fraud, disease) |
| `'precision'` | Avoiding false positives | False alarms are costly (spam filter) |
| `'f1'` | Balance of precision/recall | Equal cost for FP and FN |
| `'accuracy'` | Overall correct predictions | Balanced classes only |

### For Your Use Case
You're using `predict_proba()` for expected value calculation → **`scoring='roc_auc'`** is correct.
ROC-AUC measures how well the model ranks probabilities, not hard classifications.

---

## 3. Regularization Penalties — The Core Concept

### Why Regularization?
Without constraints, a model can assign huge coefficients to fit training data perfectly → **overfitting**.

Regularization adds a penalty term to the loss function:

```
Total Loss = Prediction Error + λ × Penalty Term
```

Where `C = 1/λ` in sklearn (**inverse relationship**)

---

## 4. L2 Penalty (Ridge) — `penalty='l2'`

### Mathematical Form
```
Penalty = Σ(β²)  →  Sum of squared coefficients
```

### Effect on Coefficients
- Shrinks all coefficients toward zero
- **Never exactly zero** — all features retained
- Larger coefficients penalized more heavily (quadratic penalty)

### Geometric Interpretation
Constraint region is a **circle** (2D) or **sphere** (nD).
Optimal solution typically lands on the curved surface → non-zero coefficients.

```
        β₂
        │    ╭───╮
        │   ╱     ╲
        │  │   ●   │  ← Optimal point on circle edge
        │   ╲     ╱      (both β₁ and β₂ non-zero)
        │    ╰───╯
        └──────────── β₁
```

### When to Use L2
- You believe **all features contribute** somewhat
- Features are **correlated** (multicollinearity) — L2 spreads weight across them
- You want **stable** coefficient estimates
- Default choice when unsure

### Solver Compatibility
`'lbfgs'` (default), `'saga'`, `'liblinear'`

---

## 5. L1 Penalty (Lasso) — `penalty='l1'`

### Mathematical Form
```
Penalty = Σ|β|  →  Sum of absolute coefficients
```

### Effect on Coefficients
- Shrinks coefficients toward zero
- **Pushes some to exactly zero** — automatic feature selection
- Linear penalty — constant pressure regardless of size

### Geometric Interpretation
Constraint region is a **diamond** (2D) or **polytope** (nD).
Optimal solution often lands on a **corner** → some coefficients exactly zero.

```
        β₂
        │      ◇
        │     ╱ ╲
        │    ╱   ╲
        │   ●     ╲  ← Optimal point on corner
        │    ╲   ╱      (β₂ = 0, only β₁ non-zero)
        │     ╲ ╱
        │      ◇
        └──────────── β₁
```

### When to Use L1
- You suspect **many features are irrelevant**
- You want a **sparse model** (fewer features)
- Feature selection is important
- Interpretability matters (fewer non-zero coefficients to explain)

### Solver Compatibility
`'saga'`, `'liblinear'` — **not** `'lbfgs'`

---

## 6. ElasticNet — `penalty='elasticnet'`

### Mathematical Form
```
Penalty = l1_ratio × Σ|β| + (1 - l1_ratio) × Σ(β²)
```

### The l1_ratio Parameter
- `l1_ratio = 1.0` → Pure Lasso (L1 only)
- `l1_ratio = 0.0` → Pure Ridge (L2 only)
- `l1_ratio = 0.5` → 50/50 mix

### Effect on Coefficients
- **Partial sparsity** — some coefficients go to zero, but not as aggressively as pure L1
- **Groups correlated features** — unlike L1 which picks one arbitrarily

### When to Use ElasticNet
- You want **some feature selection** (L1 benefit)
- But also **stability with correlated features** (L2 benefit)
- Dataset has groups of correlated predictors

### Solver Compatibility
**Only `'saga'`** — must set `solver='saga'`

### Syntax
```python
LogisticRegressionCV(
    penalty='elasticnet',
    solver='saga',
    l1_ratios=[0.1, 0.5, 0.7, 0.9],  # Values to try
    ...
)
```

---

## 7. Comparison Summary

| Aspect | L2 (Ridge) | L1 (Lasso) | ElasticNet |
|--------|------------|------------|------------|
| **Penalty** | Σ(β²) | Σ\|β\| | Mix of both |
| **Sparsity** | No | Yes | Partial |
| **Feature selection** | Keeps all | Removes weak | Removes some |
| **Correlated features** | Spreads weight | Picks one | Groups them |
| **Stability** | High | Lower | Medium |
| **Solver** | lbfgs, saga | saga, liblinear | saga only |

---

## 8. Practical Decision Tree

```
Start here
    │
    ▼
Do you need feature selection?
    │
    ├── No → Use L2 (Ridge)
    │         solver='lbfgs'
    │
    └── Yes
         │
         ▼
    Are features correlated?
         │
         ├── No → Use L1 (Lasso)
         │         solver='saga'
         │
         └── Yes → Use ElasticNet
                   solver='saga'
                   l1_ratios=[0.1, 0.5, 0.9]
```

---

## 9. For Your Damage Classification Model

**Recommendation:** Start with L2

1. Your model comparison showed LogReg already performs well
2. You have ~65 features — not excessively high
3. L2 is more stable, less sensitive to hyperparameter choice
4. For `predict_proba()` use case, probability calibration matters more than sparsity

**If you're curious about feature importance:**
Run a separate L1 model and inspect which coefficients go to zero — tells you which features the model considers irrelevant.

```python
# After fitting L1 model
coefs = pd.Series(model.coef_[0], index=X_train.columns)
print("Zero coefficients (irrelevant features):")
print(coefs[coefs == 0].index.tolist())

print("\nNon-zero coefficients (relevant features):")
print(coefs[coefs != 0].sort_values(key=abs, ascending=False))
```
