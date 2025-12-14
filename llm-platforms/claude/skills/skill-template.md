---
name: your-skill-name
description: |
  [CRITICAL: This is the ONLY trigger mechanism. Claude reads this to decide if the skill applies.]
  
  What it does: [One sentence - what capability this adds]
  
  When to use: [Explicit triggers - file types, keywords, task patterns]
  Example: "Use when user asks to create/edit .docx files, generate Word documents, 
  or work with tracked changes and comments."
---

# Your Skill Name

[Keep this under 500 lines. Move detailed content to references/]

## Quick Start

[Minimal working example - the 80% use case]

```python
# Example code that works
```

## Core Workflow

[Step-by-step for the primary task. Imperative form: "Do X", not "You should do X"]

1. Step one
2. Step two
3. Step three

## Critical Gotchas

[Things that will break if ignored - earned through trial and error]

- **Gotcha 1**: Why it matters and how to avoid
- **Gotcha 2**: The non-obvious failure mode

## Advanced Features

For specialized workflows, see:
- [FEATURE_A.md](references/FEATURE_A.md) - When dealing with X
- [FEATURE_B.md](references/FEATURE_B.md) - When dealing with Y

## Output Requirements

[Where files go, naming conventions, format requirements]

- Working directory: `/home/claude`
- Final outputs: `/mnt/user-data/outputs/`
- Always copy completed files to outputs for user access
