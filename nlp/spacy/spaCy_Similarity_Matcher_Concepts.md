# spaCy Similarity & Matchers Concepts

Theory, decision frameworks, and interpretation for spaCy's matching and similarity features.

---

## How Similarity Works

### Word Vectors

spaCy models with vectors (`_md`, `_lg`) map each word to a high-dimensional vector (typically 300 dimensions). Words used in similar contexts get similar vectors.

| Level | Vector source | Use case |
|-------|---------------|----------|
| Token | Single word vector | "hond" vs "kat" |
| Span | Average of token vectors in span | "zwarte kat" vs "rode hond" |
| Doc | Average of all token vectors | Full sentence comparison |

### Score Interpretation

| Score range | Interpretation |
|-------------|---------------|
| 0.8 - 1.0 | Very similar (near-synonyms, same concept) |
| 0.5 - 0.8 | Related (same domain, related concepts) |
| 0.2 - 0.5 | Weak relation |
| < 0.2 | Unrelated |

> Scores are context-dependent. "computer" and "laptop" score ~0.73 because they appear in similar contexts in training data, not because of dictionary definitions.

### Limitations

| Limitation | Example |
|------------|---------|
| Averaging dilutes meaning | Long sentences lose specificity |
| No word order awareness | "hond bijt man" vs "man bijt hond" get similar scores |
| OOV (out of vocabulary) words | Unknown words get zero vectors, dragging down averages |
| `_sm` models have no vectors | Returns 0.0 or unreliable scores with a warning |

### When to use which level

| Comparison | Best for |
|------------|----------|
| Token vs Token | Single concept similarity ("laptop" vs "computer") |
| Doc vs Doc | Intent matching, paraphrase detection |
| Span vs Span | Comparing extracted phrases or entities |
| Token vs Doc | Checking if a concept appears in a sentence (less reliable) |

---

## Matcher vs PhraseMatcher

### Decision framework

| Question | Matcher | PhraseMatcher |
|----------|---------|---------------|
| Need exact text lookup? | No | Yes |
| Need attribute patterns (POS, lemma)? | Yes | Limited (single attr) |
| Need operators (optional, repeating)? | Yes | No |
| Large term lists (100+)? | Slow | Fast |
| Case-insensitive search? | Via `LOWER` attribute | Via `attr="LOWER"` |

### When to use Matcher

- Finding grammatical patterns: verb + noun combinations
- Matching by lemma: catch all conjugations of a verb
- Complex patterns with optional or repeating elements
- Combining multiple token attributes in one pattern

### When to use PhraseMatcher

- Looking up known entities or terms from a list
- Exact string matching at scale
- Simple lookups where pattern flexibility is not needed

---

## Matcher Pattern Design

### Common mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Assuming token order matches sentence order | Dutch has flexible word order | Check actual token order with dependency parse first |
| Adjacent pattern for non-adjacent tokens | "laptop meegebracht" has tokens between them in "heb je een laptop meegebracht" | Use `{"OP": "*"}` to skip tokens, or use dependency tree |
| Matching on TEXT instead of LEMMA | Misses conjugations: "gestrooid" vs "strooien" | Use `{"LEMMA": "strooien"}` to catch all forms |
| Forgetting OP for optional elements | "500 euro" pattern fails on "euro" alone | Add `{"OP": "?"}` on the digit token |

### Pattern vs Dependency tree

| Approach | Best for |
|----------|----------|
| Matcher patterns | Sequential token patterns, known surface forms |
| Dependency tree | Grammatical relations (subject, object), flexible word order |
| Combined | Match a verb, then walk its dependency tree for arguments |

> The combined approach is most robust for Dutch. Match the verb with Matcher, extract arguments with `token.children`.

---

## NER Reliability

### Model size matters

The same text can produce different NER labels depending on model size and version.

| Issue | Cause | Mitigation |
|-------|-------|------------|
| Wrong entity type | Small model, insufficient training data | Use `_lg` model |
| Missing entities | Tokenization differences from whitespace | Clean input text |
| Different results across versions | Model weights change between releases | Pin model version |
| "Wie" tagged as PERSON | Statistical artifact in small model | Not a code bug |

### Known weak spots in Dutch models

| Entity type | Reliability |
|-------------|-------------|
| PERSON (common names) | Good |
| ORG (well-known) | Good |
| GPE (countries, major cities) | Good |
| MONEY (amount + currency) | Inconsistent, often tagged as DATE or QUANTITY |
| QUANTITY (with units) | Moderate |

### Improving NER results

| Approach | Effort | Impact |
|----------|--------|--------|
| Use `_lg` model | Low | Moderate improvement |
| Clean whitespace in input | Low | Fixes tokenization issues |
| Add EntityRuler for known patterns | Medium | High for specific patterns |
| Fine-tune on domain data | High | Best results |

---

## Dependencies for Dutch

### Dutch word order challenges

Dutch has a relatively flexible word order compared to English. This affects how you extract grammatical relations.

| Sentence structure | Example | ROOT position |
|-------------------|---------|---------------|
| SVO (declarative) | "Ik koop een laptop" | Verb is 2nd |
| V2 with inversion | "Gisteren kocht ik een laptop" | Verb is 2nd, subject moves |
| Subordinate clause | "... dat ik een laptop koop" | Verb is final |
| Passive | "De laptop werd gekocht" | Past participle is ROOT |

### Extracting grammatical roles

| Role | dep_ label | What it answers |
|------|-----------|-----------------|
| Subject (onderwerp) | `nsubj` | Who/what does the action? |
| Direct object (lijdend voorwerp) | `obj` | Who/what receives the action? |
| Indirect object (meewerkend voorwerp) | `iobj` | To/for whom? |
| Passive subject | `nsubj:pass` | Who/what is acted upon? |
| Oblique | `obl` | Prepositional complements |

### Subtree vs children

| Method | Returns | Use when |
|--------|---------|----------|
| `token.children` | Direct dependents only | Finding grammatical role (obj, nsubj) |
| `token.subtree` | All descendants | Getting full phrase with modifiers |
| `left_edge` / `right_edge` | Span boundaries of subtree | Extracting a clean text span |

---

## Similarity + Matcher: Design Pattern

### The intent extraction pattern

A common NLP pattern combines matching, dependency parsing, and similarity in three steps.

| Step | Tool | Purpose |
|------|------|---------|
| 1. Find the verb | Matcher (LEMMA) | Identify the action |
| 2. Extract arguments | Dependency tree (children) | Get subject, object |
| 3. Compare to concepts | Similarity | Classify the object into a known category |

This pattern is useful for building simple intent recognition without training a classifier.

### Practical example: banking intent

| Input | Step 1 (verb) | Step 2 (object) | Step 3 (similarity) |
|-------|---------------|-----------------|---------------------|
| "Ik wil 500 euro overschrijven naar Koen" | overschrijven | 500 euro, Koen | money transfer intent |
| "Heb je een laptop meegebracht?" | meegebracht | laptop | device ~ computer (0.73) |

### Threshold guidelines

| Threshold | Use case |
|-----------|----------|
| > 0.8 | Strict matching, near-synonyms only |
| > 0.6 | General concept matching |
| > 0.4 | Loose association, topic detection |

> Always validate thresholds on your specific domain. These are starting points, not rules.

---

## Whitespace Sensitivity

### Why formatting matters

spaCy tokenizes based on whitespace and punctuation rules. Extra spaces change tokenization, which changes everything downstream.

| Input | Tokenization | NER result |
|-------|-------------|------------|
| `"Wie isoleert,"` | `["Wie", "isoleert", ","]` | Correct |
| `"Wie isoleert ,"` | `["Wie", "isoleert", ","]` | Different char offsets |
| `" Wie isoleert"` | `["", "Wie", "isoleert"]` | May change entity boundaries |

### Rules

1. No leading/trailing whitespace in text
2. No spaces before punctuation
3. No spaces inside hyphenated words ("secretaris-generaal", not "secretaris -generaal")
4. Consistent line breaks in multiline strings
