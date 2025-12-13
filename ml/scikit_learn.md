# LogisticRegressionCV — Quick Reference

---

## Purpose
Logistic Regression with built-in cross-validation to find optimal regularization strength (`C`). Eliminates need for separate `GridSearchCV`.

---

## Basic Syntax
```python
from sklearn.linear_model import LogisticRegressionCV

model = LogisticRegressionCV(
    Cs=10,                    # int or list of floats
    cv=5,                     # int or cv splitter
    penalty='l2',             # 'l1', 'l2', 'elasticnet', None
    scoring='roc_auc',        # metric to optimize
    solver='lbfgs',           # optimization algorithm
    class_weight='balanced',  # handle imbalance
    max_iter=5000,            # convergence iterations
    random_state=42,
    n_jobs=-1
)

model.fit(X_train, y_train)
```

---

## Key Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `Cs` | int or list | If int: generates that many C values (log scale). If list: specific C values to try. Default=10 |
| `cv` | int or splitter | Number of folds, or `StratifiedKFold` object |
| `penalty` | str | `'l2'` (Ridge), `'l1'` (Lasso), `'elasticnet'`, `None` |
| `solver` | str | `'lbfgs'` (L2 only), `'saga'` (all penalties), `'liblinear'` (L1/L2) |
| `scoring` | str | Metric to optimize: `'roc_auc'`, `'f1'`, `'recall'`, `'accuracy'` |
| `l1_ratios` | list | Only for `elasticnet`. Values between 0-1 to try |
| `class_weight` | str/dict | `'balanced'` or custom weights |
| `refit` | bool | If True (default), refits on full train set with best C |

---

## Solver-Penalty Compatibility

| Solver | L1 | L2 | ElasticNet | None |
|--------|----|----|------------|------|
| `lbfgs` | ✗ | ✓ | ✗ | ✓ |
| `saga` | ✓ | ✓ | ✓ | ✓ |
| `liblinear` | ✓ | ✓ | ✗ | ✗ |

---

## Attributes (after fit)

```python
model.C_                # best C value found
model.Cs_               # all C values tested
model.scores_           # CV scores for each C (dict by class)
model.coef_             # coefficients of final model
model.l1_ratio_         # best l1_ratio (if elasticnet)
```

---

## Example: Custom C Range
```python
model = LogisticRegressionCV(
    Cs=np.logspace(-4, 4, 20),  # 20 values from 0.0001 to 10000
    cv=StratifiedKFold(n_splits=5, shuffle=True, random_state=42),
    scoring='roc_auc',
    penalty='l2',
    solver='lbfgs',
    class_weight='balanced',
    max_iter=5000,
    random_state=42,
    n_jobs=-1
)
```

---

## Example: ElasticNet (L1 + L2 mix)
```python
model = LogisticRegressionCV(
    Cs=np.logspace(-3, 3, 15),
    cv=5,
    penalty='elasticnet',
    solver='saga',              # required for elasticnet
    l1_ratios=[0.1, 0.5, 0.9],  # 0=Ridge, 1=Lasso
    scoring='roc_auc',
    class_weight='balanced',
    max_iter=5000,
    random_state=42
)
```

---

## Inspecting Results
```python
print(f"Best C: {model.C_[0]}")
print(f"Best l1_ratio: {model.l1_ratio_[0]}")  # elasticnet only

# CV scores per C value (for positive class)
scores_class_1 = model.scores_[1]  # shape: (n_folds, n_Cs)
mean_scores = scores_class_1.mean(axis=0)
```

---

## Notes for Damage Incident Classification
- Use `class_weight='balanced'` — keep it for imbalanced data
- Start with `penalty='l2'`, `solver='lbfgs'` — simplest
- If you want feature selection, try `penalty='l1'`, `solver='saga'`
- Use `scoring='roc_auc'` to match comparison metric
