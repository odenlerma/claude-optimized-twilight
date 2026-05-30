---
description: Load all five vibe-coding rulesets (security, clean-code, git, testing, project-setup) and apply them this session
---

Load every ruleset under `${CLAUDE_PLUGIN_ROOT}/rules/` and apply them to all subsequent work this session. One command, all guardrails on.

Steps:

1. Read all five files: `${CLAUDE_PLUGIN_ROOT}/rules/security.md`, `clean-code.md`, `git-hygiene.md`, `testing.md`, `project-setup.md`. Any missing → note which, continue with the rest.
2. For each ruleset, print a heading (ruleset name) then its rule titles only (numbered, one line each).
3. Acknowledge one sentence: "All five vibe-coding rulesets loaded. They apply to code, tests, commits, and setup rest of session."
4. From now on, conform to every loaded rule. Request would violate a rule → surface conflict before acting.

To load a single ruleset instead, use `/vibe-security-rules`, `/vibe-clean-code-rules`, `/vibe-git-rules`, `/vibe-testing-rules`, or `/vibe-setup-rules`.
