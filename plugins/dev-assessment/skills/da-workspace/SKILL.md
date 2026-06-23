---
name: da-workspace
description: >
  Discover which repos/projects in the workspace are in scope for a ticket —
  not just the current repo. Many projects are polyrepo, not monorepo. Scans the
  workspace root (parent of cwd, or cwd when it directly holds project dirs) for
  repos by `.git` and project markers (package.json, pom.xml, go.mod,
  pyproject.toml, build.gradle, Cargo.toml, composer.json, etc.), presents the
  candidate list, and confirms/trims the in-scope set with the human via
  AskUserQuestion. Use after `da-intake`, when user says "which repos", "scope
  the workspace", "across the projects", or invokes `/dev-assessment`. Runs
  before `da-assess`.
---

# DA workspace

Find the repos/projects in scope for the ticket. Workspace, not just one repo. Confirm with the human.

## Hard rules

- **Workspace, not just cwd.** Projects may be polyrepo. Look across sibling repos, not only the current one.
- **Confirm scope with the human.** Present candidates, let the human trim. Do not silently assess every repo found.
- Read-only. Enumerate dirs and markers only. Do not modify any repo.
- No candidates beyond cwd → use cwd only, note it. Do not invent repos.

## Flow

### 1. Load intent

Read `dev-assessments/<TICKET>/intent.md`. Missing → run `da-intake` first, then return.

### 2. Locate workspace root

Determine the workspace root:

- cwd directly holds multiple project dirs → workspace root = cwd.
- Else → workspace root = parent dir of cwd (sibling repos live alongside the current one).

State the chosen root.

### 3. Enumerate candidate repos

Under the workspace root, list candidate repos/projects — a dir qualifies when it holds:

- `.git`, or
- a project marker: `package.json`, `pom.xml`, `build.gradle`, `go.mod`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, `composer.json`, `Gemfile`, `*.csproj`/`*.sln`.

For each candidate capture: path, primary language/stack (from the marker), one-line guess at its role (from name + README first line if present).

### 4. Confirm in-scope set

Present the candidate list, one line per repo: `path — stack — role guess`. Factor in ticket intent + notes to pre-mark likely-relevant repos.

Confirm via `AskUserQuestion`: `Which repos are in scope for <TICKET>?` (multi-select the relevant ones, or "all", or "current repo only").

- Human trims → use the confirmed set.
- Only cwd qualifies / human picks current only → in-scope = cwd. Note the assessment is single-repo.

### 5. Write scope

Write `dev-assessments/<TICKET>/scope.md`:

```markdown
# Workspace scope — <TICKET>

Workspace root: <path>

## In scope
- `<repo path>` — <stack> — <role>

## Out of scope (found, excluded)
- (none) | - `<repo path>` — <reason / human excluded>
```

Tell user the path. Next: run `da-assess`.
