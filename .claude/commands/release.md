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
7. Run `claude plugin validate ./plugins/<plugin-name>`. Stop if it errors.
8. Stage the change with `git add plugins/<plugin-name>/.claude-plugin/plugin.json` and commit with message: `<plugin-name>: bump to v<new-version>`.
9. Report the old and new version, and the commit SHA.
