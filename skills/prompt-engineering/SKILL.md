---
name: prompt-engineering
description: Comprehensive skill for improving and creating high-quality Claude prompts using evidence-based techniques. Current for Claude 4.x (Sonnet/Opus 4.6). Use whenever asked to improve, create, troubleshoot, or evaluate prompts — including diagnosing inconsistent outputs, updating legacy prompts with over-eager language, applying effort/thinking parameters, or designing agentic skill workflows. Even if the user just says "this prompt isn't working" or "help me write a prompt for X" — use this skill.
---

# Prompt Engineering Skill

Help users create well-structured prompts that produce reliable, accurate outputs. Use when asked to improve prompts, create new prompts, troubleshoot prompt issues, or provide prompt engineering guidance.

## The Prompt Spine

Every high-quality prompt follows this structure:

```
[ROLE] You are [specific role with domain expertise]...
[OBJECTIVE] Your task is [clear, measurable outcome]...
[CONTEXT] Background, constraints, definitions, stakeholder considerations...
[INSTRUCTIONS] Step-by-step process; DO NOT list...
[OUTPUT FORMAT] Exact structure required...
[QUALITY BAR] Standards the output must meet...
[INPUT] """[Material to process]"""
```

## Essential Patterns

| Pattern | When to Use | Key Element |
|---------|-------------|-------------|
| **Scaffolding** | Complex reasoning | "Think in three passes: summarize, generate options, select best" |
| **Sequential Program** | Multi-step processes | "Follow these steps exactly in order..." |
| **Template Lock** | Consistent output structure | Provide exact headers and format to follow |
| **Exemplar** | Quality/style matching | Include example of desired output quality |
| **Self-Verification** | Reduce hallucinations | "Before final answer, identify uncertain claims, re-check reasoning" |

## Anti-Patterns to Flag

- **Dense paragraphs** → Use headers, bullets, numbered steps
- **Underspecified tasks** ("improve this") → Add specific criteria and constraints
- **Goal mixing** ("summarize and critique and rewrite") → Separate into distinct steps
- **Scope overload** → Break into modular prompts
- **Vague quality standards** ("make it good") → Define specific quality criteria

## Prompt Improvement Process

When asked to improve a prompt:
1. **Diagnose** against Prompt Spine—what's missing?
2. **Flag anti-patterns**—dense text, vague goals, mixed objectives
3. **Select patterns** that address weaknesses
4. **Reconstruct** using Prompt Spine structure
5. **Note key changes** made

When asked to create a new prompt:
1. **Clarify requirements** (ask if needed)
2. **Apply Prompt Spine** structure
3. **Add relevant patterns** (exemplars, verification)
4. **Set quality bar** appropriate to use case

## Quick Diagnostics

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Inconsistent outputs | Vague instructions | Add explicit output format template |
| Hallucinations | No verification | Include self-checking steps |
| Wrong tone | Unstated audience | Specify audience and tone |
| Too verbose | No length constraint | Set specific length limits |
| Off-topic | Unclear objective | State measurable outcome |

## Claude 4.x Behavior Notes

These apply to Claude Opus 4.x and Sonnet 4.x (including 4.6).

### Over-Eagerness Reversal

Older guidance said "push harder with emphatic language." Claude 4.x reverses this: aggressive capitalized directives now *reduce* quality.

**Avoid:**
```
CRITICAL: You MUST follow these steps EXACTLY. NEVER deviate.
```

**Prefer:**
```
Follow these steps in order. Accuracy matters more than speed.
```

### Extended Thinking / Effort Control

Claude 4.x supports an `effort` parameter (`low` / `medium` / `high`) that controls reasoning depth and latency. Use this as a primary lever before restructuring prompts.

- `low` — fast, minimal reasoning; appropriate for classification, formatting, extraction
- `medium` — default; good for most analysis and writing tasks
- `high` — activates extended thinking; best for complex multi-step reasoning, agentic workflows, or high-stakes outputs

### Anti-Markdown Block

Claude 4.x defaults to heavy formatting even when prose is more appropriate. For clean narrative output, include:

```
Respond in clear prose paragraphs. Do not use bullet points, numbered lists, bold headers, or markdown formatting unless the output is explicitly a structured document or list.
```

### Prefill Deprecation

Prefilled assistant turns are deprecated in Claude 4.6+. Replace prefill patterns with explicit output format instructions in the `[OUTPUT FORMAT]` section of the Prompt Spine.

### Parallel Tool Call Optimization

For agentic skills that invoke multiple tools, instruct Claude explicitly to parallelize independent steps:

```
Where steps do not depend on each other's outputs, execute them in parallel rather than sequentially.
```
