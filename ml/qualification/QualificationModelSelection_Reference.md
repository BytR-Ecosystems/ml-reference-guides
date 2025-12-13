# Qualification Model Selection: Quick Reference

Cross-validated comparison of multiple classifiers to qualify the best candidate before tuning.

---

## Basic Pattern

```python
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from xgboost import XGBClassifier
from sklearn.model_selection import cross_validate, StratifiedKFold

# Define models with imbalance handling
models = {
    'LogReg': LogisticRegression(class_weight='balanced', max_iter=5000, random_state=42),
    'SVM': SVC(kernel='rbf', class_weight='balanced', probability=True, random_state=42),
    'KNN_5': KNeighborsClassifier(n_neighbors=5, weights='distance'),
    'KNN_7': KNeighborsClassifier(n_neighbors=7, weights='distance'),
    'DecTree': DecisionTreeClassifier(max_depth=5, class_weight='balanced', random_state=42),
    'RF': RandomForestClassifier(n_estimators=200, class_weight='balanced', random_state=42, n_jobs=-1),
    'XGB': XGBClassifier(
        scale_pos_weight=(y_train == 0).sum() / (y_train == 1).sum(),  # ratio neg/pos
        n_estimators=200, max_depth=5, random_state=42, n_jobs=-1,
        eval_metric='logloss', verbosity=0
    ),
}

# Cross-validation: StratifiedKFold preserves class ratios in each fold
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scoring = ['roc_auc', 'average_precision', 'recall', 'precision', 'f1']

# Evaluate all models
results = []
for name, model in models.items():
    cv_scores = cross_validate(model, X_train_scaled, y_train, cv=cv, scoring=scoring, n_jobs=-1)
    
    results.append({
        'Model': name,
        'ROC_AUC': cv_scores['test_roc_auc'].mean(),
        'ROC_AUC_std': cv_scores['test_roc_auc'].std(),
        'PR_AUC': cv_scores['test_average_precision'].mean(),
        'PR_AUC_std': cv_scores['test_average_precision'].std(),
        'Recall': cv_scores['test_recall'].mean(),
        'Recall_std': cv_scores['test_recall'].std(),
        'Precision': cv_scores['test_precision'].mean(),
        'Precision_std': cv_scores['test_precision'].std(),
        'F1': cv_scores['test_f1'].mean(),
        'F1_std': cv_scores['test_f1'].std(),
    })

df_results = pd.DataFrame(results).set_index('Model')
```

---

## Metrics: What They Measure

| Metric | sklearn scoring | Measures | Use when |
|--------|-----------------|----------|----------|
| ROC-AUC | `'roc_auc'` | Probability ranking across all thresholds | Using `predict_proba()` for expected value/risk scoring |
| PR-AUC | `'average_precision'` | Precision-recall trade-off, focuses on positive class | Imbalanced data, positive class is rare |
| Recall | `'recall'` | % of actual positives caught | Missing positives is costly (fraud, damage, disease) |
| Precision | `'precision'` | % of predicted positives that are correct | False alarms are costly |
| F1 | `'f1'` | Harmonic mean of precision/recall | Need balance, using hard predictions |

### Choosing Your Primary Metric

```
Will you use predict_proba()?
    |
    +-- Yes --> ROC-AUC or PR-AUC
    |              |
    |              +-- Balanced classes? --> ROC-AUC
    |              +-- Imbalanced? --> PR-AUC
    |
    +-- No (hard predictions only)
               |
               +-- What's more costly?
                      |
                      +-- Missing positives --> Recall
                      +-- False alarms --> Precision
                      +-- Both matter equally --> F1
```

---

## Handling Imbalanced Data

### Option 1: class_weight='balanced' (Recommended)

Adjusts loss function so minority class misclassifications are penalized proportionally higher.

```python
LogisticRegression(class_weight='balanced')
SVC(class_weight='balanced')
DecisionTreeClassifier(class_weight='balanced')
RandomForestClassifier(class_weight='balanced')
```

**Advantages:**
- Preserves data integrity (no synthetic samples)
- Maintains probability calibration (critical for `predict_proba()`)
- No information loss

### Option 2: XGBoost scale_pos_weight

XGBoost uses a different parameter:

```python
XGBClassifier(scale_pos_weight=(y_train == 0).sum() / (y_train == 1).sum())
```

This is the ratio of negative to positive samples.

### Option 3: SMOTE/Undersampling

Not recommended when probability calibration matters. Synthetic samples or removed data distort probability estimates.

### KNN: No class_weight Support

KNN cannot handle imbalance natively. `weights='distance'` controls neighbor voting influence, not class imbalance.

```python
# This does NOT handle imbalance:
KNeighborsClassifier(weights='distance')  # Just weights by 1/distance
```

Exclude KNN from fair comparison when dealing with imbalanced data, or use with SMOTE (accepting calibration trade-off).

---

## Interpreting Results

### Example Output

| Model | ROC-AUC | PR-AUC | Recall | Precision | F1 |
|-------|---------|--------|--------|-----------|-----|
| LogReg | 0.6635 ± 0.0105 | 0.4184 ± 0.0208 | 0.5822 ± 0.0233 | 0.3722 ± 0.0152 | 0.4539 ± 0.0162 |
| RF | 0.6613 ± 0.0136 | 0.3886 ± 0.0298 | 0.0822 ± 0.0178 | 0.4785 ± 0.0977 | 0.1396 ± 0.0280 |

### What to Look For

**1. ROC-AUC vs Recall Mismatch**

RF shows competitive ROC-AUC (0.66) but catastrophic recall (0.08). This means:
- Probability rankings are decent overall
- But at default threshold (0.5), it fails to identify actual positives
- The model is too conservative despite `class_weight='balanced'`

**2. Standard Deviation**

Low std (± 0.01) = stable across folds. High std (± 0.05+) = sensitive to data splits.

**3. PR-AUC for Imbalanced Data**

PR-AUC is more informative than ROC-AUC when classes are imbalanced. A model can have high ROC-AUC but low PR-AUC if it struggles with the minority class.

---

## Decision Framework

```
Step 1: Check class distribution
         |
         +-- Balanced (40-60%) --> accuracy/F1 are fine
         +-- Moderate imbalance (20-40%) --> use class_weight, consider PR-AUC
         +-- Severe imbalance (<20%) --> definitely PR-AUC, may need threshold tuning
         
Step 2: Determine prediction type needed
         |
         +-- Probabilities (predict_proba) --> optimize ROC-AUC or PR-AUC
         +-- Hard labels (predict) --> optimize recall/precision/F1
         
Step 3: Compare models on primary metric
         |
         +-- But also check secondary metrics for red flags
         +-- High ROC-AUC + low recall = threshold problem
         +-- High precision + low recall = too conservative
         
Step 4: Check stability (std across folds)
         |
         +-- Prefer models with lower variance if close performance
```

---

## Common Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| RF ignores minority class | High ROC-AUC, near-zero recall | Try `class_weight='balanced_subsample'` or tune `min_samples_leaf` |
| KNN unfair comparison | Poor performance vs others | Exclude from comparison or use SMOTE |
| Optimizing wrong metric | Model performs well on metric but fails in production | Match metric to business objective |
| Ignoring std | Picked model with highest mean but high variance | Consider stability, especially with small datasets |
| Using accuracy | 90% accuracy on 90/10 split is useless | Never use accuracy for imbalanced data |

---

## Display Helper

```python
# Formatted display with mean ± std
df_display = pd.DataFrame({
    'ROC-AUC': df_results.apply(lambda x: f"{x['ROC_AUC']:.4f} ± {x['ROC_AUC_std']:.4f}", axis=1),
    'PR-AUC': df_results.apply(lambda x: f"{x['PR_AUC']:.4f} ± {x['PR_AUC_std']:.4f}", axis=1),
    'Recall': df_results.apply(lambda x: f"{x['Recall']:.4f} ± {x['Recall_std']:.4f}", axis=1),
    'Precision': df_results.apply(lambda x: f"{x['Precision']:.4f} ± {x['Precision_std']:.4f}", axis=1),
    'F1': df_results.apply(lambda x: f"{x['F1']:.4f} ± {x['F1_std']:.4f}", axis=1),
}).sort_values('ROC-AUC', ascending=False)

print("="*90)
print("MODEL COMPARISON - 5-fold CV")
print("="*90)
display(df_display)

# Best model selection
best_model_name = df_results['ROC_AUC'].idxmax()
print(f"\n✓ Best model: {best_model_name} (ROC-AUC: {df_results.loc[best_model_name, 'ROC_AUC']:.4f})")
```

---

## Key Takeaways

1. **StratifiedKFold** is mandatory for imbalanced classification
2. **class_weight='balanced'** preserves probability calibration, SMOTE does not
3. **ROC-AUC** for probability-based decisions, **PR-AUC** when imbalanced
4. **Always check recall** even if optimizing ROC-AUC: high AUC + zero recall = useless model
5. **KNN lacks class_weight**: exclude from imbalanced comparisons or accept trade-offs
6. **Look at std**: stable model > slightly better but volatile model
