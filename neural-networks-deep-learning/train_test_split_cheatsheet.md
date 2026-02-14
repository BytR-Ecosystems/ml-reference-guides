# 🔀 train_test_split Cheat Sheet

`sklearn.model_selection.train_test_split`

---

## ✂️ Function Signature

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    shuffle=True,
    stratify=y
)
```

---

## 📋 Parameters

| Parameter | Type | Description | Default |
|---|---|---|---|
| `*arrays` | X, y (or just X) | Input data and labels | Required |
| `test_size` | float or int | Fraction (`0.2`) or count (`100`) | `0.25` if train_size=None |
| `train_size` | float or int | Complement of test_size | Auto from test_size |
| `random_state` | int | Seed for reproducibility | `None` (random each run) |
| `shuffle` | bool | Shuffle before splitting | `True` |
| `stratify` | array-like | Preserves class proportions | `None` |

---

## 📤 Returns

| Return | Description |
|---|---|
| `X_train, X_test` | Feature splits |
| `y_train, y_test` | Label splits (only if y provided) |

> ⚠️ **Order matters:** returns follow input order.
> `X, y` → `X_train, X_test, y_train, y_test` (NOT grouped by train/test).

---

## 🧩 Common Patterns

### Basic split (80/20)

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42)
```

### Stratified split (preserves class balance)

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)
```

### Three-way split: Train / Validation / Test

```python
# First: split off test set
X_temp, X_test, y_temp, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)

# Second: split remaining into train + validation
X_train, X_val, y_train, y_val = train_test_split(
    X_temp, y_temp, test_size=0.25, random_state=42,
    stratify=y_temp)

# Result: 60% train, 20% val, 20% test
```

### Splitting without labels (unsupervised)

```python
X_train, X_test = train_test_split(
    X, test_size=0.2, random_state=42)
```

---

## 🎯 Stratify: When & Why

| Without stratify | With `stratify=y` |
|---|---|
| Random split may skew class ratios | Preserves original class distribution |
| Fine for large, balanced datasets | Essential for imbalanced classes |
| Train: 70%A, 30%B (was 60/40) | Train: 60%A, 40%B (matches original) |

> 💡 **Always use `stratify=y` for classification tasks**, especially with imbalanced datasets.
> Prevents test set from having zero samples of a minority class.

---

## 🚨 Common Pitfalls

| Pitfall | Fix |
|---|---|
| No `random_state` set | Different split every run. Always set it. |
| Data leakage via scaling | Fit scaler on `X_train` ONLY, transform both. |
| Time-series data shuffled | Set `shuffle=False` or use `TimeSeriesSplit`. |
| Stratify with continuous target | Stratify only works with class labels, not regression. |
| Splitting after augmentation | Always split FIRST, augment training set only. |

> 🛑 **#1 MISTAKE:** Fitting a scaler/encoder on the full dataset before splitting.
> This leaks test info into training. **Always split first, fit on train only.**

---

## ✅ Correct Preprocessing Workflow

```python
# 1. Split FIRST
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)

# 2. Fit preprocessing on TRAIN only
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# ❌ NEVER call fit() or fit_transform() on test data
```

---

## 🧭 Quick Decision Guide

| Situation | Action |
|---|---|
| Classification task? | → Use `stratify=y` |
| Time-series data? | → Use `shuffle=False` or `TimeSeriesSplit` |
| Very small dataset (<500 samples)? | → Use cross-validation (`KFold`, `StratifiedKFold`) instead |
| Need reproducibility? | → Always set `random_state` |
| Imbalanced classes (<10% minority)? | → Stratify is mandatory, consider SMOTE on training set after split |
