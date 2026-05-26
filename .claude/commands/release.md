---
description: Bump a plugin's version (semver) and commit
argument-hint: <plugin-name> <major|minor|patch>
---

Bump the version of a plugin and commit. Arguments: `$ARGUMENTS` — expected format `<plugin-name> <major|minor|patch>`.

Steps:

1. Parse `$ARGUMENTS` into two tokens: plugin name and bump type. If the format is wrong or the bump type is not one of `major`/`minor`/`patch`, stop and tell the user.
2. Confirm `plugins/<plugin-name>/.claude-plugin/plugin.json` exists. If not, stop.
3. Read the current `version` from `plugin.json`. It must be semver `MAJOR.MINOR.PATCH`. If it isn't, stop and ask the user how to proceed.
4. Compute the new version:
   - `major`: bump MAJOR, reset MINOR and PATCH to `0`.
   - `minor`: bump MINOR, reset PATCH to `0`.
   - `patch`: bump PATCH.
5. Write the updated `plugin.json` back, preserving all other fields and formatting.
6. If `marketplace.json` has a `version` field set for this same plugin entry, warn the user — `plugin.json` silently wins, and having both is a footgun. Offer to remove the marketplace-entry version.
7. Bump `metadata.version` in `.claude-plugin/marketplace.json` using the same bump type (`major`/`minor`/`patch`). Read the current `metadata.version` (semver `MAJOR.MINOR.PATCH`; if missing or malformed, stop and ask). Write back preserving other fields and formatting.
8. Run `claude plugin validate ./plugins/<plugin-name>` and `claude plugin validate .`. Stop if either errors.
8a. Confirm `README.md` reflects this plugin (listed with current usage/dependencies). If missing or stale, warn user and offer to update before committing (authoring Rule 7).
9. Stage: `git add plugins/<plugin-name>/ .claude-plugin/marketplace.json`. Stages whole plugin dir — catches untracked commands/skills/agents/hooks/mcp on first release. Before commit, run `git diff --cached --stat` and show user. If staged set spans beyond version bump (new files, edits outside `plugin.json`), confirm with user before committing. Commit message: `<plugin-name>: bump to v<new-plugin-version> (marketplace v<new-marketplace-version>)`.
10. Report the old and new versions for both files, and the commit SHA.

**PR requirement:** every PR touching `plugins/` or `marketplace.json` must paste the `/validate` output and this `/release` summary (old→new versions for both files, commit SHA) in the PR description, so reviewers confirm the catalog validates.
