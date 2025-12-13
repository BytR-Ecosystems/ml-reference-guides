# Qualification Model Selection — Quick Reference

Cross-validated comparison of multiple classifiers. Copy-paste patterns for qualifying the best model candidate before hyperparameter tuning.

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

## Metrics Reference

| Metric | sklearn scoring | Function | Primary use case |
|--------|-----------------|----------|------------------|
| ROC-AUC | `'roc_auc'` | Probability ranking across all thresholds | Probability-based decisions, balanced classes |
| PR-AUC | `'average_precision'` | Precision-recall trade-off | Imbalanced data, focus on positive class |
| Recall | `'recall'` | True Positive Rate | Missing positives is costly |
| Precision | `'precision'` | Positive Predictive Value | False alarms are costly |
| F1 | `'f1'` | Harmonic mean of precision/recall | Balance of precision and recall |

---

## Handling Imbalance — Syntax

### class_weight='balanced' (Recommended)

```python
# Scikit-learn models
LogisticRegression(class_weight='balanced')
SVC(class_weight='balanced')
DecisionTreeClassifier(class_weight='balanced')
RandomForestClassifier(class_weight='balanced')
```

### XGBoost: scale_pos_weight

```python
XGBClassifier(scale_pos_weight=(y_train == 0).sum() / (y_train == 1).sum())
# Ratio of negative to positive samples
```

### KNN: No Native Support

```python
# KNN does NOT support class_weight
KNeighborsClassifier(weights='distance')  # Only controls neighbor voting, not class balance
```

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

## Model-Specific Parameters

### Random Forest for Imbalance

```python
# If class_weight='balanced' still produces zero recall:
RandomForestClassifier(
    class_weight='balanced_subsample',  # Balances each tree's bootstrap sample
    min_samples_leaf=10,                # Prevent extreme leaf splits
    n_estimators=200, 
    random_state=42, 
    n_jobs=-1
)
```

### SVM with Probabilities

```python
# Always include probability=True if using ROC-AUC
SVC(kernel='rbf', class_weight='balanced', probability=True, random_state=42)
```

---

## Common Pitfalls — Quick Fixes

| Issue | Symptom | Fix |
|-------|---------|-----|
| RF ignores minority class | High ROC-AUC, near-zero recall | Use `class_weight='balanced_subsample'` |
| KNN unfair comparison | Poor performance vs others | Exclude from comparison or use SMOTE |
| Missing probabilities | Cannot compute ROC-AUC | Add `probability=True` to SVC |
| High variance | Large std across folds | Increase CV folds or collect more data |
| Wrong optimization | Good metric, bad production results | Match scoring to business objective |

---

## Key Takeaways

1. **StratifiedKFold** mandatory for imbalanced data
2. **class_weight='balanced'** preserves calibration
3. **Check recall** even when optimizing ROC-AUC
4. **KNN cannot handle imbalance** natively
5. **Look at std** for stability assessment
