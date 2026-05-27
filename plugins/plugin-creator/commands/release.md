---
description: Bump a plugin's version (semver) and commit
argument-hint: <plugin-name> <major|minor|patch>
---

Bump plugin version and commit. Arguments: `$ARGUMENTS` — format `<plugin-name> <major|minor|patch>`.

Steps:

1. Parse `$ARGUMENTS` into two tokens: plugin name, bump type. If format wrong or bump type not `major`/`minor`/`patch`, stop and tell user.
2. Confirm `plugins/<plugin-name>/.claude-plugin/plugin.json` exists. If not, stop.
3. Read current `version` from `plugin.json`. Must be semver `MAJOR.MINOR.PATCH`. If not, stop and ask user how to proceed.
4. Compute new version:
   - `major`: bump MAJOR, reset MINOR and PATCH to `0`.
   - `minor`: bump MINOR, reset PATCH to `0`.
   - `patch`: bump PATCH.
5. Write updated `plugin.json` back, preserve all other fields and formatting.
6. If `marketplace.json` has `version` field set for same plugin entry, warn user — `plugin.json` silently wins, both is footgun. Offer to remove marketplace-entry version.
7. Bump `metadata.version` in `.claude-plugin/marketplace.json` with same bump type. Read current `metadata.version` (semver `MAJOR.MINOR.PATCH`; if missing or malformed, stop and ask). Write back, preserve other fields and formatting.
8. Run `claude plugin validate ./plugins/<plugin-name>` and `claude plugin validate .`. Stop if either errors.
8a. Confirm `README.md` reflects this plugin (listed, current usage/dependencies). If missing or stale, update `README.md` now in caveman lite (authoring Rule 7). Also run `git status` — uncommitted README edits belong in this release commit, not orphaned. README ships in same commit as plugin.
8b. Confirm root `skills.sh.json` reflects this plugin's skills — every new or renamed plugin skill grouped (skills-distribution Rule 4). If stale, update now. skills.sh.json ships in same commit as plugin.
9. Stage: `git add plugins/<plugin-name>/ .claude-plugin/marketplace.json README.md skills.sh.json`. Stages whole plugin dir — catches untracked commands/skills/agents/hooks/mcp on first release. README.md + skills.sh.json staged so plugin docs and skills.sh grouping ship in same commit (Rules 7, skills-distribution 4). Before commit, run `git diff --cached --stat` and show user. If staged set spans beyond version bump (new files, edits outside `plugin.json`/`README.md`/`skills.sh.json`), confirm with user before committing. Commit message: `<plugin-name>: bump to v<new-plugin-version> (marketplace v<new-marketplace-version>)`.
10. Report old and new versions for both files, and commit SHA.

**PR requirement:** every PR touching `plugins/` or `marketplace.json` must paste `/validate` output and this `/release` summary (old→new versions for both files, commit SHA) in PR description, so reviewers confirm catalog validates.
