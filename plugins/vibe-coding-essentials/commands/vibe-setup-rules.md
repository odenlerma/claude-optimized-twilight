---
description: Load reusable project-setup and vibe-coding workflow ruleset and apply this session
---

Load `${CLAUDE_PLUGIN_ROOT}/rules/project-setup.md` and apply its rules to all subsequent setup and workflow work this session.

Steps:

1. Read `${CLAUDE_PLUGIN_ROOT}/rules/project-setup.md`. Missing → tell user, stop.
2. Print rule titles only (numbered, one line each) so user sees what's in force.
3. Acknowledge one sentence: "Project-setup ruleset loaded. Applies to setup and workflow rest of session."
4. From now on, scaffolding or running the project, conform to all rules. Request would violate a rule → surface conflict before acting.
