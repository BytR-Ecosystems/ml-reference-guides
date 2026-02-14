# 📊 classification_report Cheat Sheet

`sklearn.metrics.classification_report`

*AuDHD-friendly: visual hierarchy, emoji anchors, decision paths*

---

## ⚡ Quick Start

```python
from sklearn.metrics import classification_report

# For neural networks (softmax output → argmax to get class)
y_pred_classes = np.argmax(model.predict(X_test), axis=1)
y_true_classes = np.argmax(y_test_onehot, axis=1)  # if one-hot

print(classification_report(
    y_true_classes,
    y_pred_classes,
    target_names=['Bart', 'Lisa', 'Maggie']
))
```

> 💡 Both `y_true` and `y_pred` must be in the same format: either both integers or both strings.
> For neural networks with softmax output, use `np.argmax()` on **both** predictions AND one-hot labels.

---

## 🖨️ What the Output Looks Like

```
               precision    recall  f1-score   support

         Bart       0.92      0.88      0.90       150
         Lisa       0.85      0.90      0.87       130
       Maggie       0.78      0.82      0.80        90

     accuracy                           0.87       370
    macro avg       0.85      0.87      0.86       370
 weighted avg       0.87      0.87      0.87       370
```

---

## 🧠 What Each Metric Means

| Metric | Formula | Question It Answers | Fails When... |
|---|---|---|---|
| **Precision** | TP / (TP + FP) | Of all predicted Bart, how many were actually Bart? | Many false positives (crying wolf) |
| **Recall** | TP / (TP + FN) | Of all actual Bart images, how many did we catch? | Many false negatives (missing cases) |
| **F1-score** | 2 × (P × R) / (P + R) | Balanced single number. Harmonic mean of P and R. | When you need to prioritize P or R specifically |
| **Support** | count(y_true == class) | How many actual samples of this class in test set? | Low support = unreliable metrics |

---

## 🗺️ The Confusion Matrix (Mental Map)

Every classification outcome falls into one of four boxes:

|  | **PREDICTED Positive** | **PREDICTED Negative** |
|---|---|---|
| **ACTUAL Positive** | ✅ **TP** (True Positive) | ❌ **FN** (False Negative) |
| **ACTUAL Negative** | ❌ **FP** (False Positive) | ✅ **TN** (True Negative) |

> 🧠 **Memory aid:**
> - **True/False** = Was the prediction correct?
> - **Positive/Negative** = What did the model predict?
>
> | | |
> |---|---|
> | **TP** | Model said Bart, it WAS Bart → ✅ correct |
> | **FP** | Model said Bart, it was NOT Bart → ❌ false alarm |
> | **FN** | Model said NOT Bart, it WAS Bart → ❌ missed it |
> | **TN** | Model said NOT Bart, it was NOT Bart → ✅ correct |

> 🔍 **Reading the numbers:**
> - **Precision** = look DOWN the predicted column → "of all I flagged, how many were right?"
> - **Recall** = look ACROSS the actual row → "of all that existed, how many did I find?"
>
> Precision 0.92 for Bart: 92% of images the model called "Bart" really were Bart.
> Recall 0.88 for Bart: the model found 88% of all actual Bart images.

---

## 📐 The Three Averages

| Average Type | How It Works | When to Use |
|---|---|---|
| **accuracy** | Total correct / Total samples | Only if classes are balanced |
| **macro avg** | Simple average per class. Treats all classes equally. | When all classes matter equally |
| **weighted avg** | Average weighted by support (sample count). | When class imbalance exists |
| **micro avg** | Aggregate TP/FP/FN across all classes, then compute. Equals accuracy for single-label. | Multi-label classification |

> 🚨 **ACCURACY TRAP**
>
> Dataset: 950 cats, 50 dogs. Model predicts "cat" for EVERYTHING.
>
> | Metric | Value | Looks like... |
> |---|---|---|
> | Accuracy | 950/1000 = 95% | 🎉 Great! |
> | Recall for dogs | 0/50 = 0% | 💀 Useless |
>
> **Always check per-class metrics, not just accuracy.**
> `macro avg` exposes this: it would show ~50% because dog performance drags it down.

---

## ⚖️ Precision vs Recall: When to Prioritize What

| Scenario | Prioritize | Why |
|---|---|---|
| Spam filter | **Precision** | FP = real email in spam = angry user |
| Cancer screening | **Recall** | FN = missed cancer = deadly |
| Fraud detection | **Recall** | FN = missed fraud = financial loss |
| Search results ranking | **Precision** | FP = irrelevant result = bad UX |
| Balanced / general use | **F1-score** | No strong preference either way |
| Crisis detection (ND tool) | **Recall** | FN = missed crisis = harm to person |

> 💡 **Decision rule:**
> - Is a **FALSE POSITIVE** expensive/dangerous? → Prioritize **PRECISION**
> - Is a **FALSE NEGATIVE** expensive/dangerous? → Prioritize **RECALL**
> - Neither dominates? → Use **F1-SCORE**

---

## 📋 All Parameters

| Parameter | Type | Description | Default |
|---|---|---|---|
| `y_true` | array | Ground truth labels (integers or strings) | Required |
| `y_pred` | array | Predicted labels (integers or strings) | Required |
| `labels` | list | Which classes to include and their order | Auto-detected |
| `target_names` | list of str | Display names for each class | `None` (shows ints) |
| `sample_weight` | array | Weight per sample | `None` |
| `digits` | int | Decimal places in output | `2` |
| `output_dict` | bool | Return dict instead of string | `False` |
| `zero_division` | 0, 1, or `'warn'` | Value when division by zero | `'warn'` |

---

## 🧩 Common Usage Patterns

### Neural network with one-hot labels (Simpsons case)

```python
test_predict = model.predict(X_test)
print(classification_report(
    np.argmax(y_test_onehot, axis=1),   # true labels as ints
    np.argmax(test_predict, axis=1),     # predictions as ints
    target_names=le.classes_             # ['Bart','Lisa','Maggie']
))
```

### If y_test is already integers (sparse_categorical)

```python
test_predict = model.predict(X_test)
print(classification_report(
    y_test,                              # already integers
    np.argmax(test_predict, axis=1),
    target_names=le.classes_
))
```

### Get as dictionary (programmatic access)

```python
report = classification_report(
    y_true, y_pred,
    target_names=['Bart', 'Lisa', 'Maggie'],
    output_dict=True
)

# Access specific values
bart_f1 = report['Bart']['f1-score']
macro_f1 = report['macro avg']['f1-score']
overall_accuracy = report['accuracy']
```

### With sklearn classifiers (no argmax needed)

```python
from sklearn.ensemble import RandomForestClassifier

clf = RandomForestClassifier()
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)
print(classification_report(y_test, y_pred))
```

---

## 🔗 Companion: Visual Confusion Matrix

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

cm = confusion_matrix(y_true, y_pred)
disp = ConfusionMatrixDisplay(cm, display_labels=le.classes_)
disp.plot(cmap='Blues')
plt.show()
```

> 💡 Always pair `classification_report` with a confusion matrix plot.
> The report gives you the **numbers**, the matrix shows you the **pattern of errors** — which classes get confused with each other.

---

## 🚨 Common Mistakes

> 🛑 **One-hot arrays:** `classification_report` expects integer labels or strings, NOT `[[1,0,0],[0,1,0],...]`.
> Always `np.argmax()` one-hot arrays first.

> 🛑 **Argument order:** It's `classification_report(y_TRUE, y_PRED)` — true labels first, predictions second.
> Swapping them **inverts precision and recall**.

> 🛑 **Missing target_names:** Without it you get integer labels (0, 1, 2) which are harder to interpret.
> Pass `le.classes_` or a manual list.
