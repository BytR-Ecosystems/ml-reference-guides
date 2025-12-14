---
name: reference-guide
description: Create ML/AI reference documentation in a consistent, neurodivergent-friendly format. Use when user asks to create a reference guide, quick reference, cheat sheet, or documentation for ML concepts, libraries, or workflows. Produces two complementary files: a Reference file (syntax, copy-paste) and a Concepts file (theory, decisions).
---

# Reference Guide Creation

Create two files for every topic:

1. `[Topic]_Reference.md`: Syntax, parameters, runnable examples
2. `[Topic]_Concepts.md`: Theory, decision frameworks, interpretation

## Formatting Rules

Apply these rules to all content:

- No em dashes (`—`): use colons, commas, or periods
- Horizontal rules (`---`) between major sections
- Tables for comparisons, parameters, options
- ASCII decision trees for conditional logic
- Code blocks must be complete and runnable
- Bold key terms with colon: `**Term:** explanation`

## Reference File Template

```markdown
# [Topic] - Quick Reference

[One-line purpose]

---

## Basic Syntax

[Complete runnable code with inline comments]

---

## Key Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `param` | type | What it does. Default=X |

---

## [Compatibility Matrix if applicable]

| Option | A | B | C |
|--------|---|---|---|
| X | ✓ | ✗ | ✓ |

---

## Example: [Variation]

[Runnable code]

---

## Common Pitfalls

| Issue | Symptom | Fix |
|-------|---------|-----|
| Problem | What you see | What to do |

---

## Key Takeaways

1. First point
2. Second point
```

## Concepts File Template

```markdown
# [Topic] - Concepts

[One-line summary]

---

## Why This Matters

[Context: what problem this solves]

---

## [Concept Name]

### What It Is
[Definition]

### What It Measures
[Behavior]

### Use When
[Conditions]

### Limitation
[Trade-offs]

---

## Decision Framework

[ASCII decision tree]

---

## Interpreting Results

| Metric | Value | Meaning |
|--------|-------|---------|
| X | 0.65 | Interpretation |

---

## Common Pitfalls

### 1. [Pitfall Name]

**Mistake:** What people do wrong

**Why it's wrong:** The problem

**Fix:** What to do

---

## Key Takeaways

1. First point
2. Second point
```

## Decision Tree Format

```
Question?
    |
    +-- Yes -> Result
    |
    +-- No -> Next question?
              |
              +-- A -> Action A
              +-- B -> Action B
```

## Quality Checks

Before delivering:

1. No em dashes in output
2. All code blocks runnable
3. Every concept has "Use when" and "Limitation"
4. Tables replace comparison prose
5. Decision trees for "when to use" questions
6. Reference file has no theory paragraphs
7. Concepts file explains why, not just how
