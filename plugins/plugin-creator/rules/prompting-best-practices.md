# Prompting best practices

Applies when authoring prompt text — `SKILL.md`, `commands/*.md`, `agents/*.md`.
Distilled from Anthropic's prompt-engineering guide, scoped to the parts that
matter for writing plugin prompts. Works alongside caveman-lite
([plugin-authoring.md](plugin-authoring.md) Rule 6): concise prose that still
carries context, examples, and structure. Brevity ≠ vagueness.

## 1. Clear intent + explicit success criteria
State what the prompt must produce, in what format, with what constraints. A
reader with no context should be able to follow it. Order matters → numbered steps.

## 2. Context and motivation
Give the *why* behind a hard instruction — model generalizes from the reason
better than from the bare rule.

## 3. Tell what to do, not what not to do
Positive framing steers better. Reserve negatives for true prohibitions.

## 4. Examples when format matters
Output format/tone/structure matters → show 3–5 examples in `<example>` tags
(set in `<examples>`). Relevant, diverse, structured. Skip when self-evident.

## 5. XML structure for mixed content
Instructions + context + input + examples in one prompt → wrap each kind in its
own descriptive tag. Consistent names. Nest for hierarchy.

## 6. Role when it sharpens output
Assign a role only when it changes tone/behavior usefully. One sentence. Skip
otherwise.

## 7. Don't over-prompt
No gratuitous CRITICAL / MUST / "always use X" — current models follow precisely
and overtrigger on forceful language. Plain "Use X when…". Drop anti-laziness and
"if in doubt, use [tool]" carryover from older models.

## 8. Calibrate scaffolding to the job
Add thinking/effort, tool-use, agentic-state, anti-overengineering, or
anti-hallucination guidance only when the prompt's purpose needs it. Don't bolt on
boilerplate the skill/command doesn't require.
