---
description: Validate the marketplace catalog and every plugin in it
---

Validate the marketplace and every plugin.

Steps:

1. Run `claude plugin validate .` from the repo root to check `marketplace.json` (schema, duplicate plugin names, source-path traversal, version mismatches against each referenced `plugin.json`).
2. For each subdirectory under `plugins/` that contains a `.claude-plugin/plugin.json`, run `claude plugin validate ./plugins/<name>`. This is the only way to validate skill/agent/command frontmatter and `hooks/hooks.json` syntax for that plugin.
3. Report a clean summary:
   - List each validation target and whether it passed.
   - List all errors (blocking) and warnings (non-blocking) under the file they came from.
4. If there are errors, recommend fixes. Do not auto-fix unless the user asks.

If `plugins/` is empty or contains no plugin manifests, skip step 2 and note that only the marketplace catalog was validated.
