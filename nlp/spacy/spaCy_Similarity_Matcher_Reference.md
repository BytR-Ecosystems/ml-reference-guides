# spaCy Similarity & Matchers Reference

`spacy`, `spacy.matcher.Matcher`, `spacy.matcher.PhraseMatcher`

---

## Setup

```python
import spacy

nlp = spacy.load("nl_core_news_md")  # md/lg for vectors
```

```bash
python -m spacy download nl_core_news_md
```

---

## Dutch Models

| Model | Vectors | NER Quality | Size |
|-------|---------|-------------|------|
| `nl_core_news_sm` | No | Basic | ~17 MB |
| `nl_core_news_md` | Yes | Better | ~50 MB |
| `nl_core_news_lg` | Yes | Best | ~560 MB |

> Use `_md` or `_lg` for similarity. `_sm` has no vectors and returns unreliable scores.

```python
# Check model version
print(nlp.meta["version"])
```

---

## Similarity

Works on tokens, spans, and docs. Returns cosine similarity on word vectors.

### Token vs Token

```python
tokens = nlp("hond kat")
print(tokens[0].similarity(tokens[1]))  # ~0.73
```

### Doc vs Doc (comparing sentences)

```python
doc1 = nlp("Ik wil een laptop kopen")
doc2 = nlp("Ik zoek een computer te koop")
print(doc1.similarity(doc2))
```

### Span vs Span

```python
doc = nlp("De zwarte kat zat op de rode mat")
span1 = doc[1:3]   # "zwarte kat"
span2 = doc[5:7]   # "rode mat"
print(span1.similarity(span2))
```

### Comparing individual words via Doc objects

```python
computerToken = nlp("computer")
laptopToken = nlp("laptop")
print(computerToken.similarity(laptopToken))  # ~0.73
```

### Formatting output

```python
print(f"{computerToken} <-> {laptopToken} {computerToken.similarity(laptopToken):.2f}")
```

---

## Matcher (token pattern matching)

Matches sequences of tokens based on attributes like lemma, POS, text.

### Setup

```python
from spacy.matcher import Matcher

matcher = Matcher(nlp.vocab)
pattern = [{"LEMMA": "meebrengen"}]
matcher.add("REGEL_NAAM", [pattern])

matches = matcher(doc)
for match_id, start, end in matches:
    span = doc[start:end]
    print(span.text)
```

### Token Attributes

| Key | Type | Example |
|-----|------|---------|
| `TEXT` | exact text | `{"TEXT": "hond"}` |
| `LOWER` | lowercase | `{"LOWER": "brussel"}` |
| `LEMMA` | lemma form | `{"LEMMA": "strooien"}` |
| `POS` | coarse POS tag | `{"POS": "VERB"}` |
| `DEP` | dependency label | `{"DEP": "obj"}` |
| `ENT_TYPE` | entity label | `{"ENT_TYPE": "PERSON"}` |
| `IS_DIGIT` | boolean | `{"IS_DIGIT": True}` |
| `IS_PUNCT` | boolean | `{"IS_PUNCT": False}` |
| `LENGTH` | integer | `{"LENGTH": 4}` |

### Operators

| Op | Meaning | Example |
|----|---------|---------|
| `!` | 0 times (negation) | Token must NOT match |
| `?` | 0 or 1 times | Optional token |
| `+` | 1 or more | Greedy match |
| `*` | 0 or more | Skip tokens |

```python
# "500 euro" or just "euro"
pattern = [{"IS_DIGIT": True, "OP": "?"}, {"LOWER": "euro"}]
```

### Multiple patterns per rule

```python
matcher.add("GELD", [
    [{"IS_DIGIT": True}, {"LOWER": "euro"}],
    [{"IS_DIGIT": True}, {"LOWER": "dollar"}],
])
```

### Getting rule name from match

```python
for match_id, start, end in matches:
    string_id = nlp.vocab.strings[match_id]  # rule name as string
    span = doc[start:end]
    print(string_id, span.text)
```

---

## PhraseMatcher (exact phrase matching)

Faster than Matcher for exact text lookups.

### Setup

```python
from spacy.matcher import PhraseMatcher

matcher = PhraseMatcher(nlp.vocab)
terms = ["Elon Musk", "Bill Gates"]
patterns = [nlp.make_doc(t) for t in terms]
matcher.add("PERSONEN", patterns)

matches = matcher(doc)
for match_id, start, end in matches:
    span = doc[start:end]
    print(span.text)
```

### Attribute matching

| attr | Behaviour |
|------|-----------|
| (default) | Exact text match |
| `"LOWER"` | Case-insensitive |
| `"LEMMA"` | Lemma-based |

```python
matcher = PhraseMatcher(nlp.vocab, attr="LOWER")
```

---

## NER (Named Entity Recognition)

```python
doc = nlp("Kofi Annan werd in 1997 secretaris-generaal van de Verenigde Naties.")

for ent in doc.ents:
    print(f"{ent.text} | {ent.label_} ({spacy.explain(ent.label_)})")
```

### Common Dutch NER labels

| Label | Meaning |
|-------|---------|
| `PERSON` | Person |
| `ORG` | Organisation |
| `GPE` | Country/city/region |
| `DATE` | Date |
| `MONEY` | Money amount |
| `QUANTITY` | Quantity |
| `CARDINAL` | Number |

### NER validation pattern

```python
persons = [ent for ent in doc.ents if ent.label_ == "PERSON"]
money = [ent for ent in doc.ents if ent.label_ in ("MONEY", "QUANTITY")]

if persons:
    print(f"✓ Persoon herkend: {', '.join(p.text for p in persons)}")
else:
    print("✗ Geen persoon herkend")
```

---

## Dependencies (syntactic relations)

```python
for token in doc:
    print(f"{token.text:15} {token.lemma_:15} {token.pos_:8} {token.dep_:10}")
```

### Key dependency labels

| Label | Meaning |
|-------|---------|
| `nsubj` | Subject (onderwerp) |
| `obj` | Direct object (lijdend voorwerp) |
| `iobj` | Indirect object (meewerkend voorwerp) |
| `ROOT` | Main verb (hoofdwerkwoord) |
| `advmod` | Adverbial modifier (bijwoordelijke bepaling) |
| `amod` | Adjective modifier (bijvoeglijk naamwoord) |
| `det` | Determiner (lidwoord) |
| `obl` | Oblique nominal |
| `aux` | Auxiliary verb (hulpwerkwoord) |
| `aux:pass` | Passive auxiliary |
| `punct` | Punctuation |

### Children of a token

```python
for token in doc:
    if token.dep_ == "ROOT":
        for child in token.children:
            explanation = spacy.explain(child.dep_) or child.dep_
            print(f"  └─ {child.text} ({child.dep_}: {explanation})")
```

### Full noun phrase via subtree

```python
for child in token.children:
    if child.dep_ == "obj":
        span = doc[child.left_edge.i : child.right_edge.i + 1]
        print(f"Full phrase: {span.text}")  # "een laptop"
```

### Containing sentence

```python
span = doc[start:end]
print(span.sent)
```

---

## Combined Patterns

### Matcher + Dependency Tree (extract direct object from matched verb)

```python
from spacy.matcher import Matcher

nlp = spacy.load("nl_core_news_md")
doc = nlp("heb je een laptop meegebracht?")

matcher = Matcher(nlp.vocab)
matcher.add("WERKWOORD", [[{"LEMMA": "meebrengen"}]])

matches = matcher(doc)
for match_id, start, end in matches:
    token = doc[start:end][0]

    for child in token.children:
        if child.dep_ == "obj":
            print(f"Werkwoord: {token.text} -> Lijdend voorwerp: {child.text}")
```

### Matcher + Similarity (match verb, extract object, compare to reference)

```python
import spacy
from spacy.matcher import Matcher

nlp = spacy.load("nl_core_news_md")
computerToken = nlp("computer")
voorwerp = ""

doc = nlp("heb je een laptop meegebracht?")

matcher = Matcher(nlp.vocab)
matcher.add("WERKWOORD", [[{"LEMMA": "meebrengen"}]])

matches = matcher(doc)
for match_id, start, end in matches:
    token = doc[start:end][0]
    for child in token.children:
        if child.dep_ == "obj":
            voorwerp = child.text

voorwerpToken = nlp(voorwerp)
print(f"{computerToken} <-> {voorwerpToken} {computerToken.similarity(voorwerpToken):.2f}")
# computer <-> laptop 0.73
```

---

## Debug Helpers

```python
# Model info
print(nlp.meta["version"])
print(nlp.meta["name"])

# spacy.explain fallback for unknown labels
explanation = spacy.explain(token.dep_) or token.dep_

# Full token dump
for token in doc:
    print(f"{token.text:15} {token.lemma_:15} {token.pos_:8} {token.dep_:10} {token.ent_type_:10}")
```
