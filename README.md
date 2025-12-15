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

### LLM Platforms

#### ChatGPT

| Topic | Reference | Concepts | Template |
|-------|-----------|----------|----------|
| Custom Instructions Config | [Reference](llm-platforms/chatgpt/config/ChatGPT_Config_Reference.md) | [Concepts](llm-platforms/chatgpt/config/ChatGPT_Config_Concepts.md) | [YAML](llm-platforms/chatgpt/config/chatgpt_config_template.yaml) |

Extended example: [tech_assistant_config.yaml](llm-platforms/chatgpt/config/examples/tech_assistant_config.yaml)

#### Claude

| Topic | File |
|-------|------|
| Reference Guide Skill | [reference-guide-SKILL.md](llm-platforms/claude/skills/reference-guide-SKILL.md) |

---

## Structure

```
ml-reference-guides/
├── llm-platforms/
│   ├── chatgpt/
│   │   └── config/
│   │       ├── examples/
│   │       ├── chatgpt_config_template.yaml
│   │       ├── ChatGPT_Config_Reference.md
│   │       └── ChatGPT_Config_Concepts.md
│   └── claude/
│       └── skills/
│           └── reference-guide-SKILL.md
├── ml/
│   └── scikit/
│       └── supervised_learning/
│           ├── qualification/
│           └── quantile_regression/
├── pytorch/                    # Planned
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