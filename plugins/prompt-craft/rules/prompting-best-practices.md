# Prompting best practices

Checklist `prompt-rewrite` applies. Distilled from Anthropic's prompt-engineering
guide for current Claude models (Fable 5, Opus 4.8/4.7/4.6, Sonnet 4.6, Haiku 4.5).
Apply what fits the target prompt's job — skip rules that don't apply. Goal:
sharper prompt, same intent. Never redesign the task.

## 1. Clear and direct
State desired output, format, constraints explicitly. Want "above and beyond"?
Say so — don't rely on the model inferring it from vague text. Golden rule: a
colleague with no context should be able to follow the prompt. Order matters →
numbered steps or bullets.

## 2. Context and motivation
Explain *why* behind an instruction. "No ellipses, because a text-to-speech engine
reads this aloud and mispronounces them" beats "NEVER use ellipses." Model
generalizes from the reason.

## 3. Tell what to do, not what not to do
"Respond in flowing prose paragraphs" beats "don't use markdown." Positive
framing steers better. Reserve negatives for hard prohibitions.

## 4. Examples (few-shot)
3–5 examples when output format, tone, or structure matters. Make them relevant
(mirror the real case), diverse (cover edge cases, vary so no unintended pattern
sticks), structured (wrap each in `<example>`, the set in `<examples>`). Skip
examples when the task is self-evident — don't pad.

## 5. XML structure
Mixed content (instructions + context + input + examples) → wrap each kind in its
own descriptive tag (`<instructions>`, `<context>`, `<input>`). Consistent tag
names. Nest for hierarchy (`<documents>` → `<document index="n">`).

## 6. Role
Assign a role in the system prompt when it sharpens tone/behavior ("You are a
helpful coding assistant specializing in Python"). One sentence is enough. Skip
when it adds nothing.

## 7. Long context
20k+ token inputs → put longform docs at the **top**, query/instructions/examples
at the bottom (improves quality up to ~30%). Wrap docs with `<document>` →
`<source>` + `<document_content>`. For long-doc tasks, ask the model to pull
relevant quotes into `<quotes>` first, then answer.

## 8. Control output format
Match prompt style to desired output (remove markdown from prompt → less markdown
out). Steer format with positive instructions or XML output tags. For
prose-not-bullets, say so explicitly and give the reason.

## 9. Don't over-prompt
Current models follow instructions precisely and overtrigger on forceful
language. Replace "CRITICAL: You MUST use this tool when…" with "Use this tool
when…". Dial back anti-laziness / "if in doubt, use X" prompting carried over
from older models — it now causes overtriggering. Drop blanket "always use [tool]"
defaults; target them ("use [tool] when it improves understanding of the problem").

## 10. Tool use, explicitly
For action, say "Change this function" not "can you suggest changes" (the latter
gets suggestions, not edits). For parallel-eligible independent calls, instruct
parallel execution. Add a `<default_to_action>` or `<do_not_act_before_instructions>`
block only when the prompt's autonomy level needs steering.

## 11. Thinking / effort
Current models use adaptive thinking + an `effort` parameter — not `budget_tokens`.
Don't prescribe step-by-step plans the model can derive ("think thoroughly" beats a
hand-written CoT). Constrain reasoning only when overthinking is a real problem;
otherwise let effort/adaptive thinking handle depth. When thinking is off, prefer
"consider"/"evaluate"/"reason through" over "think."

## 12. Agentic / long-horizon
Multi-window or long autonomous tasks → tell the model context compacts (don't
stop early), persist state (git, `tests.json`, `progress.txt`), work
incrementally, verify its own work. Add subagent guidance only when over/under-use
shows. Add reversibility/safety confirmation guidance for destructive or
shared-system actions.

## 13. Anti-overengineering
For coding prompts prone to scope creep, add: only changes directly requested or
clearly necessary; no speculative abstractions, defensive code for impossible
cases, or docs on untouched code; minimum complexity for the current task.

## 14. Anti-hallucination
Add `<investigate_before_answering>`: never speculate about un-opened code; read
referenced files before answering; ground claims in what was actually inspected.

## 15. Model identity / strings
App needs the model to self-identify or emit a model string → state it: "The
current model is Claude Opus 4.8 … exact string `claude-opus-4-8`." Default new
LLM-powered apps to the latest capable Claude model.

## Output of a rewrite
1. Rewritten prompt in a code block.
2. "What changed + why" — each change mapped to the rule above.
Keep scope to the user's intent. No real API calls, keys, tokens, or secrets in
prompt text — guidance only.
