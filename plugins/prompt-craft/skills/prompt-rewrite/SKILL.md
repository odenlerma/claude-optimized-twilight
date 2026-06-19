---
name: prompt-rewrite
description: >
  Diagnose a weak prompt and rewrite it to Anthropic's prompt-engineering best
  practices — clarity and explicit success criteria, context/motivation behind
  instructions, 3–5 examples in <example> tags when format matters, XML
  structure, role, tell-what-to-do framing, calibrated thinking/tool/agentic
  guidance — without over-prompting or redesigning the task. Outputs before/after
  plus what changed and why. Use when user says "improve this prompt", "rewrite
  this prompt", "make this prompt better", "fix my prompt", "optimize this
  prompt", pastes a vague/under-specified prompt to improve, or invokes
  `/prompt-craft`.
---

# Prompt rewrite

Take a weak prompt. Diagnose against best practices. Rewrite. Show before/after +
why. Preserve user intent — rewrite, never redesign the task.

## Hard rules

- Read `${CLAUDE_PLUGIN_ROOT}/rules/prompting-best-practices.md` first. Apply it.
- **Preserve intent.** Rewrite the prompt, not the task. Don't add goals the user
  didn't ask for.
- **Tell what to do, not what not to do.** Positive framing.
- **Add context/motivation** behind instructions — model generalizes from the why.
- **Examples only when they earn their place.** 3–5 in `<example>` tags when
  output format/tone/structure matters. Self-evident task → skip.
- **XML structure** for mixed content (instructions + context + input + examples).
- **Role** only when it sharpens output.
- **Make format explicit.**
- **Don't over-prompt.** No gratuitous CRITICAL / MUST / "always use X" — current
  models overtrigger on forceful language. Plain "Use X when…".
- **Calibrate scaffolding to the job.** Add thinking/effort, tool-use, agentic
  state, anti-overengineering, or anti-hallucination guidance only when the
  prompt's purpose needs it. Don't bolt on what doesn't apply.
- **No secrets, no inline API calls.** Guidance text only — never real keys,
  tokens, URLs, or runnable API calls (authoring Rules 1–2).
- **Caveman lite** in your own explanation text. Drop articles, filler, hedging.

## Flow

### 1. Capture target + intent

Get the prompt to rewrite (`$ARGUMENTS`, pasted text, or referenced context). Note
its goal, target model, and where it runs — system prompt, agent loop, one-shot
call. Ask only when genuinely blocking; otherwise infer the most useful reading
and proceed.

### 2. Diagnose

List concrete weaknesses against the checklist. Common ones:

- Vague intent / no explicit success criteria
- Missing context or motivation
- "What not to do" framing
- No examples where format matters (or examples that are thin/redundant)
- Unstructured — instructions, context, input all run together
- Over-prompted — CRITICAL/MUST, "always use", anti-laziness carryover
- Over-engineered — scaffolding the task doesn't need
- Hallucination-prone — asks for claims about code/files without grounding
- No role where one would sharpen output

Name only real weaknesses. Tight prompt → say so, minimal changes.

### 3. Rewrite

Apply the fixes. Keep scope to the user's intent. Reach for `<example>`/`<xml>`
tags, role, explicit format, and calibrated agentic/thinking guidance per the
checklist — only where they apply.

### 4. Output

Return:

```
<rewritten prompt in a code block>
```

Then **What changed + why** — short list, each change mapped to a best-practice
rule (e.g. "Added motivation (Rule 2)", "Examples in <example> tags (Rule 4)",
"Dropped MUST → plain instruction (Rule 9)"). If the original was already strong,
say what was kept and why changes were minimal.
