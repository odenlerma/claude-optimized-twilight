# claude-optimized-twilight

Personal Claude Code **plugin marketplace**. Drop-in plugins — skills, slash commands, agents — you install once and use across projects. Built on Claude Code's marketplace system. Docs: https://docs.claude.com/en/docs/claude-code/marketplaces

Repo: https://github.com/odenlerma/claude-optimized-twilight

## Install

### Claude Code (CLI, desktop, IDE)

Local clone:

```
/plugin marketplace add /Users/audrueygana/Desktop/projects/claude-optimized-twilight
```

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

- Plugin skills: `wiki-maintainer`, `jira-ticketize`
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

Bundled rules: `plugin-authoring.md` (9 rules), `skills-distribution.md` (skills.sh discoverability).

Usage: install into a marketplace repo. `/add-plugin my-plugin` to scaffold, fill in components, `/validate` before commit, `/release my-plugin minor` to ship.

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
