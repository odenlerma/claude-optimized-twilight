# claude-optimized-twilight

Personal Claude Code **plugin marketplace**. Drop-in plugins — skills, slash commands, agents — you install once and use across projects. Built on Claude Code's marketplace system. Docs: https://docs.claude.com/en/docs/claude-code/marketplaces


## Install

### Claude Code (CLI, desktop, IDE)

After pushing to GitHub:

```
/plugin marketplace add odenlerma/claude-optimized-twilight
```

Then install a plugin:

```
/plugin install meeting-wiki@claude-optimized-twilight
```

### Claude Cowork

Marketplace must live on GitHub. Add it, then enable the plugin:

```
/plugin marketplace add odenlerma/claude-optimized-twilight
/plugin install meeting-wiki@claude-optimized-twilight
```

### Claude web (claude.ai/code)

Push repo to GitHub. Open plugin/marketplace settings, add the marketplace by GitHub slug `odenlerma/claude-optimized-twilight`, then install the plugin from the catalog.

### npx skills (skills only)

[`skills`](https://github.com/vercel-labs/skills) is the open agent-skills CLI. Pulls SKILLs from a repo:

```
npx skills add odenlerma/claude-optimized-twilight
```

This installs all discoverable skills — they auto-trigger by phrase:

- Plugin skills: `wiki-maintainer`, `jira-ticketize`, `qa-test-plan`, `qa-execute`, `qa-jira-report`, `cr-scope`, `cr-review`, `cr-jira-report`, `idea-clarify`, `idea-assess`, `idea-draft`, `idea-jira-create`
- Vendored utilities: `caveman`, `caveman-review`, `skill-creator`

Preview without installing: `npx skills add . --list`.

**Listing on skills.sh** happens automatically — the [skills.sh](https://www.skills.sh) leaderboard is telemetry-driven, so the repo gets indexed on first `npx skills add`. The page is grouped via [`skills.sh.json`](skills.sh.json) (plugin skills grouped; vendored skills fall under "Other skills").

**Caveat:** `npx skills` brings **skills only** — not `/wiki-*` slash commands or hooks. For full plugin (commands included), use the `/plugin marketplace add` path above.

## Plugins

### meeting-wiki

Turn meeting transcripts into an evolving, interlinked markdown wiki — people, projects, topics, decisions, action items, kept up to date by Claude. Never auto-commits; you review and commit.

| Command | Argument | Does |
|---|---|---|
| `/wiki-init` | — | Scaffold `wiki/` + `raw/`, templates, schema |
| `/wiki-ingest` | `<transcript-path>` | Ingest transcript, extract entities, update pages |
| `/wiki-query` | `<question>` | Answer from wiki with citations; optionally save as topic |
| `/wiki-status` | — | Counts, recent meetings, open todos, active projects |
| `/wiki-lint` | — | Health check (missing pages, orphans, stale, budget violations) — report only |
| `/wiki-ticketize` | `<transcript-path>` | Draft Jira tickets from action items (human approval at every gate) |

Skills: `wiki-maintainer` (all wiki ops), `jira-ticketize` (transcript → Jira drafts).

Usage: `/wiki-init` once, then `/wiki-ingest path/to/transcript.txt` per meeting. Ask `/wiki-query "what did we decide on auth?"`. Review the diff, commit yourself.

**Obsidian-ready.** Wiki output is plain markdown with `[[wikilinks]]`, YAML frontmatter properties, and `- [ ]` task checkboxes — Obsidian reads it natively. Optional: download Obsidian at https://obsidian.md/download and point a vault at the generated `wiki/` folder to browse pages, backlinks, and tasks. No Obsidian config files are generated; the plugin stays a git-backed markdown wiki.

### plugin-creator

Author Claude Code plugin marketplaces. Bundles the four workflow commands plus both authoring-rule docs, so any marketplace repo gets the same scaffolding, validation, and release flow. Self-contained — commands read rules from `${CLAUDE_PLUGIN_ROOT}/rules/`.

| Command | Argument | Does |
|---|---|---|
| `/add-plugin` | `<plugin-name>` | Scaffold plugin dir, `plugin.json`, catalog entry, README section |
| `/plugin-authoring` | — | Load authoring rules; enforce on all plugin work this session |
| `/validate` | `[plugin-name]` | Validate catalog (schema + skills.sh discoverability) + plugin-quality review |
| `/release` | `<plugin-name> <major\|minor\|patch>` | Bump plugin + marketplace versions, commit |

Bundled rules: `plugin-authoring.md` (10 rules — incl. Rule 10 hook authoring: `hooks/hooks.json` location, matchers, handler types, stdin-JSON input), `skills-distribution.md` (skills.sh discoverability).

Usage: install into a marketplace repo. `/add-plugin my-plugin` to scaffold, fill in components, `/validate` before commit, `/release my-plugin minor` to ship.

### feature-qa

**MANUAL-only** QA driven by a Jira ticket. Fetch requirements, study the codebase, build a test plan, run it by hand through a browser MCP with screenshot evidence, write a plain-language pass/fail report, post results to the ticket — only when everything passes, and only after you confirm.

| Command | Argument | Does |
|---|---|---|
| `/feature-qa` | `<jira-ticket-number> [notes]` | Full flow: build plan → run with evidence → report on pass |

Skills: `qa-test-plan` (ticket → plan: positive, negative, regression, what-if), `qa-execute` (run by hand, screenshot per scenario, pass/fail report), `qa-jira-report` (post result to ticket on pass, confirm-gated).

Usage: `/feature-qa PROJ-123`. Optional notes after ticket key add context — `/feature-qa PROJ-123 focus on mobile checkout, skip desktop`. Plan written to `qa-reports/PROJ-123/test-plan.md`, screenshots + report to `qa-reports/PROJ-123/`. Any failed scenario or logged issue → verdict FAIL, nothing posted. PASS → comment drafted, you confirm, then posted.

All external access detected dynamically by tool **description** (not name) and gated: no Jira MCP → paste ticket details / paste-ready comment; no browser MCP → stops (manual QA needs one for evidence).

### feature-code-review

Unbiased, **impact-driven** code review driven by a Jira ticket. Fetch the feature intent, scope the change (git diff vs base, widened to related code), review across seven impact dimensions, write a verdict report, post to the ticket — only on PASS, and only after you confirm. Skips per-line nits and generic by-the-book standards; focuses on usability, readability, and real impact.

| Command | Argument | Does |
|---|---|---|
| `/feature-code-review` | `<jira-ticket-number> [notes]` | Full flow: scope change → review by impact → report on pass |

Skills: `cr-scope` (ticket → intent + change inventory: diff + related context), `cr-review` (review by dimension — performance, security, responsiveness, error handling, UX, reusable components, structure; verdict PASS / CHANGES REQUESTED), `cr-jira-report` (post result to ticket on pass, confirm-gated).

Usage: `/feature-code-review PROJ-123`. Optional notes after ticket key add context — `/feature-code-review PROJ-123 focus on the auth refactor`. Scope written to `code-reviews/PROJ-123/scope.md`, review to `code-reviews/PROJ-123/review.md`. Any Blocker or Major finding → verdict CHANGES REQUESTED, nothing posted. PASS → comment drafted, you confirm, then posted.

All external access detected dynamically by tool **description** (not name) and gated: no Jira reader MCP → paste ticket details; no comment MCP → paste-ready comment.

### idea-to-ticket

Turn a raw idea into well-formed Jira ticket(s). Interrogate vague input (push back, name contradictions, no sycophancy), assess the codebase for collisions when one is present, draft caveman-lite title/description/acceptance-criteria, create in Jira only on your explicit approval. Developer assessment + test plan post as a **comment** (when a comment MCP exists), keeping the ticket description lean. The front half of the feature lifecycle — `feature-code-review` and `feature-qa` cover the rest.

| Command | Argument | Does |
|---|---|---|
| `/idea-to-ticket` | `<idea text> [notes]` | Full flow: interrogate idea → assess codebase → draft tickets → create on approval |

Skills: `idea-clarify` (idea → intent + acceptance criteria, interrogates vagueness), `idea-assess` (codebase only — collision scan + dev assessment + test plan), `idea-draft` (caveman-lite ticket(s), flat split on confirm), `idea-jira-create` (create on approval, post dev/test as comment, confirm-gated).

Usage: `/idea-to-ticket add CSV export to the reports page`. Optional notes add scope/constraints — `/idea-to-ticket add CSV export focus on the reports page only`. Drafts written to `ticket-drafts/<slug>/` (intent, assessment, tickets). Idea spans separate work → flat split proposed, you confirm. Nothing created until you approve.

Differs from `jira-ticketize` (meeting-wiki): that drafts tickets from a meeting transcript; this drafts from a free-text idea with codebase-collision awareness and dev/QA artifacts. All external access detected by tool **description** (not name) and gated: no create MCP → paste-ready ticket(s) + comment(s); no comment MCP → paste-ready comment.

## Dependencies

### meeting-wiki

- **git** — recommended, for versioning the wiki. Plugin never auto-commits; you control commits.
- **Transcript files** — `.txt`, `.md`, `.vtt`, `.srt` (Zoom, Granola, Otter, Google Meet).
- **Jira MCP server** — *optional*, only for `/wiki-ticketize` auto-create. Without it, plugin prints paste-ready ticket drafts. Plugin detects the MCP by tool description, supply it via your own config.
- No env vars, no API keys, no setup scripts.

### plugin-creator

- **`claude` CLI** — for `claude plugin validate` (schema validation in `/validate` and `/release`).
- **`npx` + `skills`** — *optional*, for skills.sh discoverability check in `/validate` Phase 1b. Missing → phase skips gracefully.
- **git** — for `/release` commit and staging.
- Commands assume a marketplace repo layout (`.claude-plugin/marketplace.json`, `plugins/`). No env vars, no API keys.

### feature-qa

- **Browser-automation MCP** — *required* for `qa-execute`. Manual QA drives the app and screenshots evidence through it (e.g. Playwright MCP). Detected by tool description; supply via your own config. Missing → `qa-execute` stops.
- **Jira MCP servers** — *optional*: reader (fetch ticket), comment-add (post result), attachment-upload (inline screenshots in posted comment). Missing reader → paste ticket details. Missing comment-add or attachment-upload → paste-ready comment + manual-attach list. Detected by tool description; wire via your own config.
- **git** — recommended, for versioning `qa-reports/` evidence artifacts.
- No env vars, no API keys, no inline secrets (authoring Rules 1–2).

### feature-code-review

- **git** — *required* for `cr-scope` diff. Reviews the feature branch diff vs a base branch; falls back to locating feature files if no diff/branch exists.
- **Jira MCP servers** — *optional*: reader (fetch ticket intent), comment-add (post result on pass). Missing reader → paste ticket details. Missing comment-add → paste-ready comment. Detected by tool description; wire via your own config.
- No browser MCP needed — review is static (code vs intent). No env vars, no API keys, no inline secrets (authoring Rules 1–2).

### idea-to-ticket

- **Jira MCP servers** — *optional*: create-issue (file the ticket), comment-add (post dev assessment + test plan), reader (duplicate-ticket check). Missing create-issue → paste-ready ticket(s) + comment(s). Missing comment-add → paste-ready comment. Missing reader → duplicate check skipped. Detected by tool description; wire via your own config.
- **git / source files** — codebase analysis (`idea-assess`) runs only when invoked inside a codebase; outside one, it skips and tickets carry title/desc/AC only.
- No browser MCP needed. No env vars, no API keys, no inline secrets (authoring Rules 1–2).

## Troubleshooting

- **Plugin update not picked up** — `version` in `plugin.json` unchanged, so Claude Code serves the cached copy. Bump version, or remove and re-add the marketplace.
- **`/wiki-*` commands missing after `npx skills`** — expected. `npx skills` installs skills only. Use `/plugin marketplace add` for slash commands.
- **`/plugin marketplace add` fails** — repo not pushed to GitHub, wrong `<owner>` slug, or local path isn't a git repo. Check the slug and that the repo is reachable.
- **Jira tickets only draft, never create** — no Jira MCP detected. Wire up a Jira MCP server in your config; otherwise drafts are paste-ready.
- **Wrong skill triggers / name collision** — another installed plugin shares a tool name. Names must be unique (authoring Rule 8); rename or disable the conflicting plugin.

## Contributing

Authoring rules: [`.claude/rules/plugin-authoring.md`](.claude/rules/plugin-authoring.md). Read before writing or editing any plugin.

Workflow commands:

- `/add-plugin <name>` — scaffold a new plugin (dir, `plugin.json`, catalog entry, README section).
- `/validate [plugin-name]` — validate catalog + run plugin-quality review.
- `/release <plugin-name> <major|minor|patch>` — bump versions and commit.

**PR requirement:** every PR touching `plugins/` or `marketplace.json` must paste the `/validate` output and the `/release` summary (old→new versions, commit SHA) in the PR description. Update this README for any plugin add or edit (Rule 7).
