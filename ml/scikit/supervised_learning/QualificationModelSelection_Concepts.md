# Qualification Model Selection — Concepts

Understanding model comparison, metrics, and what results actually mean.

---

## Purpose of Qualification

**Qualification** = Finding the best model family before tuning hyperparameters.

Why not skip to tuning?
- Tuning is expensive (hundreds of parameter combinations)
- Wrong model family wastes time (no amount of tuning makes KNN handle imbalance)
- Need baseline to measure tuning improvement

**Process:**
1. **Qualify** → Compare model families with reasonable defaults
2. **Tune** → Optimize hyperparameters for the winner
3. **Validate** → Test on hold-out set

---

## Metrics — What They Actually Measure

### ROC-AUC: Probability Ranking

**What it is:**  
Area under the ROC curve (True Positive Rate vs False Positive Rate at all thresholds).

**What it measures:**  
How well the model ranks predictions. A score of 0.8 means: "Given one random positive and one random negative sample, there's an 80% chance the model assigns a higher probability to the positive."

**Use when:**
- You'll use `predict_proba()` for risk scoring or ranking
- Classes are relatively balanced
- You care about performance across all possible thresholds

**Limitation:**  
Can be misleading with severe imbalance. A model can have high ROC-AUC while failing to identify the minority class.

### PR-AUC: Precision-Recall Balance

**What it is:**  
Area under the Precision-Recall curve across all thresholds.

**What it measures:**  
Trade-off between precision (avoiding false alarms) and recall (catching all positives). More sensitive to performance on the minority class.

**Use when:**
- Classes are imbalanced (<20% minority)
- The positive class is what matters (fraud, defects, disease)
- False negatives and false positives both have costs

**Why it's better for imbalance:**  
PR-AUC doesn't give credit for correctly rejecting tons of negatives. ROC-AUC does.

### Recall: Don't Miss Positives

**What it is:**  
True Positives / (True Positives + False Negatives)

**What it measures:**  
Of all actual positives, what % did we catch?

**Use when:**
- Missing a positive is extremely costly
- Example: Disease screening, fraud detection
- Accept more false alarms to avoid missing cases

**Trade-off:**  
Higher recall usually means lower precision (more false alarms).

### Precision: Avoid False Alarms

**What it is:**  
True Positives / (True Positives + False Positives)

**What it measures:**  
Of all predicted positives, what % were actually positive?

**Use when:**
- False alarms are costly
- Example: Surgery recommendations, expensive interventions
- Resources are limited (can't investigate every alert)

**Trade-off:**  
Higher precision usually means lower recall (miss more true cases).

### F1: Harmonic Balance

**What it is:**  
2 × (Precision × Recall) / (Precision + Recall)

**What it measures:**  
Balanced combination of precision and recall. Harmonic mean punishes extreme imbalance (0.9 precision + 0.1 recall = 0.18 F1, not 0.5).

**Use when:**
- Both false positives and false negatives matter
- Need a single number that reflects balance
- Using hard predictions (not probabilities)

**Limitation:**  
Doesn't tell you *why* performance is good/bad. Low F1 could be low precision, low recall, or both.

---

## Metric Selection Framework

```
What type of prediction?
    |
    +-- Probabilities (predict_proba)
    |       |
    |       +-- Class distribution?
    |               |
    |               +-- Balanced (40-60%) → ROC-AUC
    |               +-- Imbalanced (<20%) → PR-AUC
    |
    +-- Hard labels (predict)
            |
            +-- Which error is more costly?
                    |
                    +-- Missing positives → Recall
                    +-- False alarms → Precision
                    +-- Both equal → F1
```

---

## Handling Imbalance — Why It Matters

### The Problem

With imbalanced data (e.g., 10% positive class), a naive model can get 90% accuracy by predicting "negative" for everything. Useless.

### class_weight='balanced': The Why

Most sklearn classifiers optimize a loss function. By default, all misclassifications have equal cost.

`class_weight='balanced'` adjusts the loss:
- Misclassifying a minority sample → higher penalty
- Misclassifying a majority sample → lower penalty
- Penalty proportional to class frequency

**Formula:**  
`weight = n_samples / (n_classes × n_samples_class)`

For 90% negative, 10% positive:
- Negative weight = 1000 / (2 × 900) = 0.56
- Positive weight = 1000 / (2 × 100) = 5.0

The model now "cares" 9× more about getting positives right.

### Why class_weight > SMOTE

**class_weight='balanced':**
- ✓ Preserves original data
- ✓ Maintains probability calibration
- ✓ No information loss
- ✓ Works with `predict_proba()`

**SMOTE (Synthetic Minority Oversampling):**
- ✗ Creates synthetic samples (interpolations)
- ✗ Distorts probability calibration
- ✗ Can introduce noise
- ✗ Probabilities no longer reflect reality

Use SMOTE only when class_weight isn't available or fails completely.

### XGBoost: scale_pos_weight

XGBoost uses a different parameter but same concept:

```python
scale_pos_weight = count(negative) / count(positive)
```

This is the ratio, not the weight formula. For 900 negative, 100 positive:
```python
scale_pos_weight = 900 / 100 = 9.0
```

### KNN: No Solution

KNN classifies by majority voting among k neighbors. There's no loss function to weight.

`weights='distance'` controls neighbor influence (closer = more vote), NOT class balance.

**Options:**
1. Exclude KNN from comparison (recommended)
2. Use SMOTE (accept probability distortion)
3. Use specialized KNN variants (e.g., BorderlineSMOTE + KNN)

---

## Interpreting Results

### Example Comparison

| Model | ROC-AUC | PR-AUC | Recall | Precision | F1 |
|-------|---------|--------|--------|-----------|-----|
| LogReg | 0.6635 ± 0.0105 | 0.4184 ± 0.0208 | 0.5822 ± 0.0233 | 0.3722 ± 0.0152 | 0.4539 ± 0.0162 |
| RF | 0.6613 ± 0.0136 | 0.3886 ± 0.0298 | 0.0822 ± 0.0178 | 0.4785 ± 0.0977 | 0.1396 ± 0.0280 |
| SVM | 0.6701 ± 0.0118 | 0.4025 ± 0.0243 | 0.5123 ± 0.0312 | 0.3915 ± 0.0189 | 0.4438 ± 0.0201 |

### What to Look For

**1. Metric Mismatches Reveal Problems**

Random Forest:
- ROC-AUC: 0.66 (competitive)
- Recall: 0.08 (catastrophic)

**Interpretation:**  
RF can rank predictions decently (ROC-AUC), but at the default threshold it predicts "negative" for 92% of actual positives. The model is too conservative despite `class_weight='balanced'`.

**Action:**  
Try `class_weight='balanced_subsample'` or tune `min_samples_leaf`. RF often struggles with imbalance.

**2. Standard Deviation = Stability**

| Model | Mean | Std | Interpretation |
|-------|------|-----|----------------|
| LogReg | 0.6635 | ± 0.0105 | Very stable across folds |
| RF | 0.6613 | ± 0.0136 | Less stable, more fold-dependent |

Low std (< 0.02): Reliable performance estimate  
High std (> 0.05): Model is sensitive to data variations, may not generalize

**3. PR-AUC vs ROC-AUC for Imbalance**

With 10% positive class:
- ROC-AUC 0.85 might sound great
- But PR-AUC 0.30 reveals it struggles with the minority class

ROC-AUC weights all thresholds equally. PR-AUC focuses on positive predictions.

---

## Decision Framework

### Step 1: Assess Class Imbalance

```python
class_ratio = y_train.sum() / len(y_train)

if class_ratio > 0.4:
    # Balanced
    primary_metric = 'ROC-AUC' or 'F1'
elif class_ratio > 0.2:
    # Moderate imbalance
    primary_metric = 'PR-AUC'
    use_class_weight = True
else:
    # Severe imbalance
    primary_metric = 'PR-AUC'
    use_class_weight = True
    consider_threshold_tuning = True
```

### Step 2: Match Metric to Use Case

| Use case | Primary metric | Why |
|----------|---------------|-----|
| Loan default risk scoring | ROC-AUC | Need probabilities for risk tiers, not binary decision |
| Fraud detection dashboard | PR-AUC | Imbalanced, need good precision at usable recall levels |
| Medical screening (early stage) | Recall | Can't miss cases, false alarms acceptable for follow-up |
| Medical diagnosis (final) | Precision | Invasive treatment, need confidence in positive prediction |
| Email spam filter | F1 | Balance: missing spam is bad, blocking real email is bad |

### Step 3: Compare on Primary + Check Red Flags

**Primary:** Select model with highest primary metric  
**Secondary:** Check for red flags in other metrics

Red flags:
- High ROC-AUC + low recall → threshold problem
- High precision + low recall → too conservative
- High recall + low precision → too aggressive
- High variance → unstable, may not generalize

### Step 4: Stability Consideration

If two models are within 0.02 on primary metric, prefer:
1. Lower standard deviation (more stable)
2. Simpler model (easier to tune, deploy, explain)
3. Faster training (iteration speed matters)

---

## Common Pitfalls

### 1. Optimizing the Wrong Metric

**Mistake:**  
Using ROC-AUC because it's familiar, then deploying with hard predictions.

**Why it's wrong:**  
ROC-AUC measures probability ranking. If you use `predict()` (threshold=0.5), the ranking doesn't matter—only the classification at that threshold.

**Fix:**  
If using hard predictions, optimize recall/precision/F1. If using probabilities, optimize ROC-AUC/PR-AUC.

### 2. Using Accuracy with Imbalance

**Mistake:**  
90% accuracy sounds great!

**Reality:**  
With 90% majority class, predicting "negative" for everything gets 90% accuracy while catching zero positives.

**Fix:**  
Never use accuracy for imbalanced data. Use PR-AUC, recall, or F1.

### 3. Ignoring Standard Deviation

**Mistake:**  
Picking model with 0.71 mean ROC-AUC over 0.70, ignoring std of ±0.08 vs ±0.01.

**Reality:**  
The "better" model's true performance is somewhere between 0.63-0.79. The "worse" model reliably delivers 0.69-0.71.

**Fix:**  
With high variance, either get more data or prefer stable models.

### 4. Unfair KNN Comparison

**Mistake:**  
Including KNN with `weights='distance'` in imbalanced comparison where others use `class_weight='balanced'`.

**Reality:**  
KNN can't handle imbalance natively. It's being evaluated under harder conditions.

**Fix:**  
Either exclude KNN or apply SMOTE to all models (accepting probability distortion).

### 5. RF Default Settings with Imbalance

**Mistake:**  
Trusting `class_weight='balanced'` will make RF work.

**Reality:**  
RF often still produces near-zero recall with imbalanced data. The ensemble averaging and tree structure make it particularly challenging.

**Fix:**  
Try `class_weight='balanced_subsample'`, increase `min_samples_leaf`, or consider different model families.

---

## What Happens After Qualification

1. **Winner Selected:** Based on primary metric + stability + red flag check
2. **Hyperparameter Tuning:** GridSearchCV or Bayesian optimization on the winner
3. **Threshold Tuning:** If using `predict_proba()`, optimize threshold for production metric
4. **Final Validation:** Test on completely unseen data
5. **Monitoring:** Track metric drift in production

Qualification eliminates bad candidates early. Tuning squeezes performance from the winner. Both are necessary.
