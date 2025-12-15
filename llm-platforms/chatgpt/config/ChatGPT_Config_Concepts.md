# ChatGPT Config Template - Concepts

Standardized YAML configuration for consistent ChatGPT behavior via Custom Instructions or GPT builder.

---

## Why This Matters

ChatGPT supports structured configuration via Custom Instructions (for personal use) or the GPT builder (for shared assistants). YAML provides a clean, readable format to define behavior rules that ChatGPT interprets from the system prompt.

Other platforms (Claude, Gemini, local models) use different configuration methods. This guide is ChatGPT-specific.

Without explicit configuration, you get inconsistent outputs: verbose when you need concise, creative when you need factual, casual when you need technical. A config template externalizes these preferences into a reusable, shareable format. Define once, apply everywhere.

---

## Modes

### What It Is

A mode is a named collection of behavior parameters. Instead of repeating instructions per prompt, you activate a mode and the assistant adopts all associated behaviors.

### What It Controls

Output length, reasoning visibility, example inclusion, domain-specific checks, interaction style.

### Use When

You have recurring interaction patterns. Teaching, debugging, brainstorming, and code review each require different assistant behaviors.

### Limitation

Modes are not enforced by the LLM. They are instructions the model may follow imperfectly. Complex or conflicting mode definitions reduce compliance.

---

## Brainstorm Mode

### What It Is

Divergent thinking mode. Generates multiple options without premature filtering.

### What It Controls

Enables idea generation, disables self-critique, encourages quantity over quality initially.

### Use When

You need options, not answers. Early-stage problem solving, exploring solution spaces, generating alternatives.

### Limitation

Produces noise. Requires human filtering. Not suitable when you need a single actionable recommendation.

---

## Code Evaluator Mode

### What It Is

Structured code review mode. Systematic analysis across security, performance, and maintainability dimensions.

### What It Controls

Enables domain-specific checks, categorizes findings by severity, focuses on actionable feedback.

### Use When

Reviewing code before merge, auditing existing codebases, learning from code examples.

### Limitation

Static analysis only. Cannot execute code, test edge cases, or verify runtime behavior. Misses context-dependent issues.

---

## Learning Aid Mode

### What It Is

Teaching mode. Builds understanding through scaffolding, analogies, and comprehension checks.

### What It Controls

Enables step-by-step explanation, uses analogies, asks verification questions, adapts complexity.

### Use When

Learning new concepts, understanding fundamentals, preparing to teach others.

### Limitation

Slower than direct answers. Frustrating when you already understand and just need the syntax. Switch to short mode when scaffolding is unnecessary.

---

## Decision Framework

```
What do you need from the assistant?
    |
    +-- Quick answer, no explanation
    |       |
    |       +-- Use: short mode
    |
    +-- Generate ideas/options
    |       |
    |       +-- Use: brainstorm mode
    |
    +-- Review code for issues
    |       |
    |       +-- Use: code_evaluator mode
    |
    +-- Understand a concept deeply
            |
            +-- Use: learning_aid mode
```

```
Is the built-in mode sufficient?
    |
    +-- Yes -> Use as-is
    |
    +-- No -> What's missing?
              |
              +-- Domain-specific checks -> Add to custom_modes
              +-- Different output format -> Override output block
              +-- Different safety rules -> DO NOT override safety block
```

---

## Mode Behavior Matrix

| Behavior | short | brainstorm | code_evaluator | learning_aid |
|----------|-------|------------|----------------|--------------|
| Concise output | ✓ | ✗ | ✗ | ✗ |
| Show reasoning | ✗ | ✓ | ✓ | ✓ |
| Include examples | ✗ | ✓ | ✓ | ✓ |
| Domain checks | ✗ | ✗ | ✓ | ✗ |
| Comprehension checks | ✗ | ✗ | ✗ | ✓ |
| Divergent output | ✗ | ✓ | ✗ | ✗ |

---

## Common Pitfalls

### 1. Overloading a Single Mode

**Mistake:** Adding all desired behaviors to one mode instead of creating focused modes.

**Why it's wrong:** Conflicting instructions reduce compliance. A mode that is "concise but includes examples and shows reasoning" is internally contradictory.

**Fix:** Create separate modes for distinct interaction patterns. Switch modes rather than combining them.

---

### 2. Overriding Safety in Custom Modes

**Mistake:** Disabling `restrict_high_risk` or `refuse_unsafe_actions` in a custom mode for convenience.

**Why it's wrong:** Safety parameters exist to prevent harmful outputs. Overriding them per-mode creates exploitable loopholes.

**Fix:** Safety block is global and immutable. If you need high-risk content for legitimate purposes (security research, red teaming), use a separate config with explicit justification, not a mode override.

---

### 3. Expecting Enforcement

**Mistake:** Treating the config as a contract the LLM will perfectly follow.

**Why it's wrong:** LLMs are probabilistic. Complex instructions, long conversations, or conflicting prompts cause drift. The config is a strong suggestion, not a guarantee.

**Fix:** Keep modes simple. Verify compliance on critical outputs. Use explicit mode reminders in long conversations.

---

## Key Takeaways

1. Modes externalize recurring prompt patterns into reusable, named configurations
2. Match mode to task: short for answers, brainstorm for options, code_evaluator for review, learning_aid for understanding
3. Safety parameters are global. Never override them in custom modes
4. Keep modes focused. One purpose per mode, switch rather than combine
