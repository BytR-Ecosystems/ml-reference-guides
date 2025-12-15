# ML Reference Guides

Quick reference docs for ML practitioners. Two files per topic: Reference (syntax) and Concepts (theory).

---

## Guides

### scikit-learn

| Topic | Reference | Concepts |
|-------|-----------|----------|
| Logistic Regression | [Reference](ml/scikit/supervised_learning/qualification/LogisticRegressionCV_Reference.md) | [Concepts](ml/scikit/supervised_learning/qualification/LogisticRegression_Concepts.md) |
| Model Selection (Classification) | [Reference](ml/scikit/supervised_learning/qualification/QualificationModelSelection_Reference.md) | [Concepts](ml/scikit/supervised_learning/qualification/QualificationModelSelection_Concepts.md) |
| Quantile Regression | [Reference](ml/scikit/supervised_learning/quantile_regression/Quantile_Regression_Reference.md) | [Concepts](ml/scikit/supervised_learning/quantile_regression/Quantile_Regression_Concepts.md) |

---

## Structure

```
ml-reference-guides/
├── ml/
│   └── scikit/
│       └── supervised_learning/
│           ├── qualification/
│           └── quantile_regression/
├── pytorch/                    # Planned
├── llm/                        # Planned
├── math/                       # Planned
├── ABOUT.md
├── LICENSE
└── README.md
```

---

## Format

| File Type | Contains | Use When |
|-----------|----------|----------|
| `*_Reference.md` | Parameters, syntax, runnable code | You know what you need, want copy-paste |
| `*_Concepts.md` | Theory, decision frameworks, interpretation | You need to understand why |

---

## More Info

[ABOUT.md](ABOUT.md): Project background, design philosophy, contributing guidelines.

---

GPL-2.0 | [ByteRider](https://github.com/BytR-Ecosystems)