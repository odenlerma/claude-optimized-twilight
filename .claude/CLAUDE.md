# claude-optimized-twilight — plugin marketplace

This repo is a Claude Code **plugin marketplace** catalog. Docs: https://docs.claude.com/en/docs/claude-code/marketplaces

## Layout

- `.claude-plugin/marketplace.json` — the catalog. Required keys: `name`, `owner`, `plugins`.
- `plugins/<plugin-name>/` — one directory per plugin.
  - `.claude-plugin/plugin.json` — plugin manifest (`name`, `description`, `version`).
  - `skills/<skill-name>/SKILL.md` — model-invocable skills.
  - `commands/*.md` — slash commands.
  - `agents/*.md` — agent definitions.
  - `hooks/hooks.json` — event hooks.
  - `mcp/` — MCP server configs.

## Conventions

- **Plugin names** are kebab-case (lowercase letters, digits, hyphens). Claude.ai marketplace sync rejects other forms.
- **Source paths** in `marketplace.json` start with `./` and resolve from the marketplace root (not from `.claude-plugin/`). Never use `../`.
- **`version`** in `plugin.json` must be bumped on every release. If the field is unchanged, Claude Code treats the plugin as cached and skips updates for existing users.
- Do not set `version` in both `plugin.json` and the marketplace entry — `plugin.json` silently wins.
- Use `${CLAUDE_PLUGIN_ROOT}` (not relative paths) inside hooks/MCP configs to reference files in the plugin's install dir, since plugins are copied to a cache location.

## Reserved marketplace names (don't rename to any of these)

`claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `anthropic-agent-skills`, `knowledge-work-plugins`, `life-sciences`, `claude-for-legal`, `claude-for-financial-services`, `financial-services-plugins`.

## Workflow

Adding a plugin: prefer the `/add-plugin <name>` slash command in this repo, which scaffolds the directory, plugin.json, and marketplace entry in one step.

Editing a plugin: bump `version` in its `plugin.json` before committing if behavior changed.

Always run `/validate` before committing changes to `marketplace.json` or any `plugin.json`.

## Installing this marketplace

Local: `/plugin marketplace add /Users/audrueygana/Desktop/projects/claude-optimized-twilight`

After pushing to GitHub: `/plugin marketplace add <owner>/claude-optimized-twilight`
