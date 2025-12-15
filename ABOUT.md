# About This Project

## Why This Exists

Most ML resources have the same problems: too academic, too verbose, buried in prose when you need syntax, or so simplified they skip the why.

This collection shares what actually works: guides built from real project experience, designed for practitioners who want to understand concepts deeply without wading through fluff.

This is a living repository, actively maintained and expanded.

---

## Scope

| Domain | Coverage |
|--------|----------|
| Classical ML | scikit-learn, statistical foundations |
| Deep Learning | PyTorch, training pipelines, optimization |
| LLMs & Inference | Mistral (EU-based, GDPR-friendly), vLLM, quantization, deployment |
| MLOps | Reproducibility, experiment tracking, production patterns |

---

## Design Philosophy

### Neurodivergent-Friendly Structure

These guides are structured for how many of us actually process information:

| Principle | Implementation |
|-----------|----------------|
| Visual structure over walls of text | Tables, decision trees, clear headers |
| Concept and syntax separated | Two files per topic, choose what you need |
| No hidden assumptions | If something matters, it's explicit |
| Decision frameworks | When to use what, not just how |

### Two-File Format

Every topic gets two complementary guides:

**Reference files** (`*_Reference.md`):
- Copy-paste ready
- Parameters, compatibility tables, common patterns
- For when you know what you want and need the syntax

**Concepts files** (`*_Concepts.md`):
- The why behind the what
- Mathematical intuition, geometric interpretations
- Decision trees for choosing approaches
- For when you need to actually understand

Start with Concepts if you're learning. Use Reference once you get it.

---

## Roadmap

| Topic | Status |
|-------|--------|
| scikit-learn (classification) | ✓ Available |
| scikit-learn (quantile regression) | ✓ Available |
| PyTorch (training loops, debugging) | Planned |
| Mistral (local deployment, fine-tuning) | Planned |
| Inference optimization (vLLM, quantization) | Planned |
| Linear algebra for ML | Planned |

---

## Contributing

Found something unclear? Have a guide that helped you? PRs welcome.

### Format Requirements

| Rule | Rationale |
|------|-----------|
| Direct language, no filler | Respect reader's time |
| Tables over prose for comparisons | Faster to scan |
| Code examples that run | No pseudo-code frustration |
| Decision frameworks over vague advice | Actionable guidance |
| No em dashes | Accessibility |
| Horizontal rules between sections | Visual separation |

### Quality Checklist

Before submitting:

1. All code blocks are complete and runnable
2. Every concept has "Use when" and "Limitation"
3. Tables replace comparison prose
4. Decision trees for "when to use" questions
5. Reference file has no theory paragraphs
6. Concepts file explains why, not just how

---

## License

GPL-2.0. Use it, share it, improve it.

---

*A [ByteRider](https://github.com/BytR-Ecosystems) project.*
