---
description: Scaffold a new plugin in this marketplace
argument-hint: <plugin-name>
---

Scaffold a new plugin named `$ARGUMENTS` in this marketplace.

Steps:

1. Validate `$ARGUMENTS` is kebab-case (lowercase letters, digits, hyphens only — no spaces, no uppercase). If not, stop and tell the user the rule.
2. Confirm `plugins/$ARGUMENTS/` does not already exist. If it does, stop and tell the user.
3. Ask the user (with `AskUserQuestion`) for:
   - A one-line description of the plugin.
   - Which component types it will include (multi-select: skills, commands, agents, hooks, mcpServers).
4. Create the directory structure:
   - `plugins/$ARGUMENTS/.claude-plugin/`
   - One subdirectory per component type they chose (e.g. `plugins/$ARGUMENTS/skills/`). Add a `.gitkeep` to each so empty dirs are tracked.
5. Write `plugins/$ARGUMENTS/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "$ARGUMENTS",
     "description": "<the description from step 3>",
     "version": "0.1.0"
   }
   ```
6. Add an entry to the `plugins` array in `.claude-plugin/marketplace.json`:
   ```json
   {
     "name": "$ARGUMENTS",
     "source": "./plugins/$ARGUMENTS",
     "description": "<same description>"
   }
   ```
   Preserve the existing JSON formatting.
7. Run `claude plugin validate .` from the repo root. If it errors, fix and re-run.
8. Report what was created (file tree + next steps for filling in component files).

Do not commit — leave the changes staged so the user can review first.
