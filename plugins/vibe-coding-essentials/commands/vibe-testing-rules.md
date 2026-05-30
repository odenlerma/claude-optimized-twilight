---
description: Load reusable testing-discipline ruleset and apply to tests written or edited this session
---

Load `${CLAUDE_PLUGIN_ROOT}/rules/testing.md` and apply its rules to all subsequent test work this session.

Steps:

1. Read `${CLAUDE_PLUGIN_ROOT}/rules/testing.md`. Missing → tell user, stop.
2. Print rule titles only (numbered, one line each) so user sees what's in force.
3. Acknowledge one sentence: "Testing ruleset loaded. Applies to tests I write or edit rest of session."
4. From now on, writing or editing tests, conform to all rules. Request would violate a rule → surface conflict before acting.
