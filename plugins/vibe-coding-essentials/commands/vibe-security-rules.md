---
description: Load reusable software-security ruleset and apply to code written or edited this session
---

Load `${CLAUDE_PLUGIN_ROOT}/rules/security.md` and apply its rules to all subsequent code work this session.

Steps:

1. Read `${CLAUDE_PLUGIN_ROOT}/rules/security.md`. Missing → tell user, stop.
2. Print rule titles only (numbered, one line each) so user sees what's in force.
3. Acknowledge one sentence: "Security ruleset loaded. Applies to code I write or edit rest of session."
4. From now on, writing or editing code, conform to all rules. Request would violate a rule → surface conflict before acting.
