---
description: Diagnose a weak prompt and rewrite it to Anthropic's prompt-engineering best practices — clarity, context, examples, XML structure, calibrated guidance — without over-prompting or redesigning the task
argument-hint: <prompt text to improve>
---

Rewrite the prompt in `$ARGUMENTS` to prompt-engineering best practices.

`$ARGUMENTS` = the prompt to improve. Empty → ask the user to paste it, or read it
from referenced context. Optional trailing note (target model, where it runs) adds
scope.

Run the **`prompt-rewrite`** skill:

1. Capture the target prompt + intent (goal, target model, system prompt / agent /
   one-shot).
2. Diagnose weaknesses against `${CLAUDE_PLUGIN_ROOT}/rules/prompting-best-practices.md`.
3. Rewrite — preserve intent, apply only fixes that fit the prompt's job, no
   over-prompting.
4. Output the rewritten prompt + a "what changed + why" list mapped to the rules.

Preserve the user's intent — rewrite the prompt, not the task.
