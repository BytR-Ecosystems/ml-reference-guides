# ML Reference Guides

Practical, no-bullshit reference guides for machine learning and AI. From classical ML to deep learning, model deployment to inference optimization.

## Why This Exists

There are plenty of ML resources out there. Most suffer from the same problems: too academic, too verbose, buried in prose when you just need the syntax, or so simplified they skip the *why*.

**ByteRider** created this collection to share what actually works: guides built from real project experience, designed for practitioners who want to understand concepts deeply without wading through fluff.

This is a **living repository**, actively maintained and expanded as we work through new tools, frameworks, and techniques.

### Scope

- **Classical ML**: scikit-learn, statistical foundations
- **Deep Learning**: PyTorch, training pipelines, optimization
- **LLMs & Inference**: Mistral (EU-based, GDPR-friendly), vLLM, quantization, deployment
- **MLOps**: reproducibility, experiment tracking, production patterns

### Neurodivergent-Friendly by Design

These guides are structured for how many of us actually process information:

- **Visual structure over walls of text**: tables, decision trees, clear headers
- **Concept and syntax separated**: understand the theory OR grab the code, your choice
- **No hidden assumptions**: if something matters, it's explicit
- **Decision frameworks**: when to use what, not just how

## Current Guides

### Classification (scikit-learn)

#### Qualification

| Guide | Description |
|-------|-------------|
| [LogisticRegressionCV Reference](ml/scikit/supervised_learning/qualification/LogisticRegressionCV_Reference.md) | API-style quick reference: parameters, solver compatibility, code snippets |
| [Logistic Regression Concepts](ml/scikit/supervised_learning/qualification/LogisticRegression_Concepts.md) | Deep dive into regularization (L1/L2/ElasticNet), C parameter, scoring metrics, geometric intuition |
| [Qualification Model Selection Reference](ml/scikit/supervised_learning/qualification/QualificationModelSelection_Reference.md) | Copy-paste patterns for multi-model comparison, metrics, imbalance handling |
| [Qualification Model Selection Concepts](ml/scikit/supervised_learning/qualification/QualificationModelSelection_Concepts.md) | Understanding metrics, interpreting results, why class_weight > SMOTE, decision frameworks |

### Regression (scikit-learn)

#### Quantile Regression

| Guide | Description |
|-------|-------------|
| [Quantile Regression Reference](ml/scikit/supervised_learning/quantile_regression/Quantile_Regression_Reference.md) | API-style quick reference: parameters, estimators, code snippets |
| [Quantile Regression Concepts](ml/scikit/supervised_learning/quantile_regression/Quantile_Regression_Concepts.md) | Understanding quantile loss, when to use vs OLS, interpretability, practical applications |

### Coming Soon

- **PyTorch**: Training loops, custom datasets, debugging common issues
- **Mistral**: Local deployment, fine-tuning, EU compliance considerations
- **Inference Optimization**: vLLM, quantization (GPTQ, AWQ, GGUF), batching strategies
- **Linear Algebra for ML**: Vectors, matrices, transformations (the stuff that actually matters)

## Structure

```
ml-reference-guides/
├── ml/
│   └── scikit/
│       └── supervised_learning/
│           ├── qualification/
│           │   ├── LogisticRegressionCV_Reference.md
│           │   ├── LogisticRegression_Concepts.md
│           │   ├── QualificationModelSelection_Reference.md
│           │   └── QualificationModelSelection_Concepts.md
│           └── quantile_regression/
│               ├── Quantile_Regression_Reference.md
│               └── Quantile_Regression_Concepts.md
├── pytorch/                    # Coming
├── llm/                        # Coming (Mistral, inference, deployment)
├── math/                       # Coming (linear algebra, statistics)
├── LICENSE
└── README.md
```

## Usage

Each topic has two types of guides:

1. **Reference** (`*_Reference.md`): Copy-paste ready. Parameters, compatibility tables, common patterns. For when you know what you want and need the syntax.

2. **Concepts** (`*_Concepts.md`): The why behind the what. Mathematical intuition, geometric interpretations, decision trees for choosing approaches. For when you need to actually understand.

Start with Concepts if you're learning. Use Reference once you get it.

## Contributing

Found something unclear? Have a guide that helped you? PRs welcome.

Keep the format:
- Direct language, no filler
- Tables over prose where it makes sense
- Code examples that actually run
- Decision frameworks over vague advice

## License

GPL-2.0. Use it, share it, improve it.

---

*A [ByteRider](https://github.com/BytR-Ecosystems) project.*
