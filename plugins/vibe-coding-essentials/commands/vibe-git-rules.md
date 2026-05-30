---
description: Load reusable git and commit-hygiene ruleset and apply to source-history work this session
---

Load `${CLAUDE_PLUGIN_ROOT}/rules/git-hygiene.md` and apply its rules to all subsequent commit and history work this session.

Steps:

1. Read `${CLAUDE_PLUGIN_ROOT}/rules/git-hygiene.md`. Missing → tell user, stop.
2. Print rule titles only (numbered, one line each) so user sees what's in force.
3. Acknowledge one sentence: "Git-hygiene ruleset loaded. Applies to commits and history work rest of session."
4. From now on, committing or managing source history, conform to all rules. Request would violate a rule → surface conflict before acting.
