# ChatGPT Config Template - Quick Reference

YAML configuration for ChatGPT Custom Instructions and GPT builder.

---

## Basic Structure

```yaml
assistant:
  default_mode: short          # Active mode on session start
  style: technical             # technical | casual
  language: english            # Response language

modes:
  mode_name:
    description: "What this mode does"
    max_paragraphs: 1          # null for unlimited
    include_examples: false
    show_reasoning: false

output:
  no_fluff: true
  explain_variables: true

safety:
  restrict_high_risk: true
```

---

## Core Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `default_mode` | string | Mode activated on session start. Must match a key in `modes` |
| `style` | string | Response tone. Options: technical, casual |
| `language` | string | Output language. Default: english |
| `no_fluff` | bool | Remove rhetoric, metaphors, praise. Default: true |
| `explain_variables` | bool | Define abbreviations on first use. Default: true |

---

## Built-in Modes

| Mode | Purpose | Key Settings |
|------|---------|--------------|
| `short` | Concise answers | max_paragraphs=1, no examples, no reasoning |
| `brainstorm` | Generate options | divergent_thinking=true, no self-critique |
| `code_evaluator` | Review code | security+performance+maintainability checks |
| `learning_aid` | Teach concepts | analogies, scaffolding, understanding checks |

---

## Mode Parameters

| Parameter | Type | Modes | Description |
|-----------|------|-------|-------------|
| `max_paragraphs` | int/null | all | Response length limit. null=unlimited |
| `include_examples` | bool | all | Add concrete examples |
| `show_reasoning` | bool | all | Explain thought process |
| `divergent_thinking` | bool | brainstorm | Generate multiple alternatives |
| `critique_own_ideas` | bool | brainstorm | Self-evaluate suggestions |
| `check_security` | bool | code_evaluator | Flag security issues |
| `check_performance` | bool | code_evaluator | Flag performance issues |
| `check_maintainability` | bool | code_evaluator | Flag code quality issues |
| `severity_levels` | list | code_evaluator | Issue categories to report |
| `use_analogies` | bool | learning_aid | Explain via comparisons |
| `check_understanding` | bool | learning_aid | Ask verification questions |
| `scaffold_complexity` | bool | learning_aid | Build from simple to complex |

---

## Safety Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `restrict_high_risk` | bool | Limit output for security/OT topics |
| `human_in_loop_required` | bool | Require confirmation for impactful actions |
| `refuse_unsafe_actions` | bool | Block harmful step-by-step instructions |

---

## Example: Adding Custom Mode

```yaml
custom_modes:
  debugger:
    description: "Systematic bug hunting"
    max_paragraphs: null
    include_examples: true
    show_reasoning: true
    hypothesis_driven: true
    check_assumptions: true
    trace_execution: true
```

---

## Example: Platform-Specific Override

```yaml
# ChatGPT custom instructions format
system_prompt: |
  You are a technical assistant. Default mode: {{default_mode}}.
  Style: {{style}}. Language: {{language}}.
  {% if output.no_fluff %}No fluff, no praise, no filler.{% endif %}
  {% for mode in modes %}...{% endfor %}
```

---

## Common Pitfalls

| Issue | Symptom | Fix |
|-------|---------|-----|
| Mode not activating | Assistant ignores mode settings | Verify default_mode matches a key in modes |
| Conflicting parameters | Unexpected behavior | Check mode inheritance, child overrides parent |
| Safety bypass | High-risk content generated | Ensure safety block is not overridden in custom modes |

---

## Key Takeaways

1. Define modes once, switch dynamically per session or prompt
2. Safety parameters apply globally, cannot be overridden per mode
3. Extend via `custom_modes` without modifying base template
