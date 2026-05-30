# Plugin authoring rules

Applies when creating or editing anything under `plugins/`.

## 1. No inline API calls
Never call external APIs from skill/command prompt text. Wrap the API as an MCP server the developer supplies via their own config. Skill text instructs the developer how to wire the MCP — it never contains the API call itself.

## 2. No inline env vars or secrets
Never hardcode API keys, tokens, URLs, account IDs, or any sensitive value in skill/command text. Preferred path: route the secret through an MCP server's config. Fallback (only if MCP unsuitable): skill ships a `.env.example` template at the plugin root, the SKILL.md body documents copy-and-fill setup steps, and the runtime reference is `${CLAUDE_PLUGIN_ROOT}/.env`. Never commit a real `.env`.

## 3. Do not code MCP servers
This marketplace ships plugins (skills, commands, agents, hooks) — not MCP server implementations. When a plugin needs an MCP, write the prompt the developer pastes into their own model to build the MCP. Display that prompt to the developer; do not implement the MCP here.

## 4. Plugins must be reusable
Write for general scenarios, not the author's setup. No hardcoded paths, org names, team names, or personal assumptions. Parameterize via `$ARGUMENTS`, MCP config the developer supplies, or values the user provides at runtime. A plugin only the author can use is a bug.

## 5. Break skills down
Analyze the plugin's intent before writing. Any step that is independent and reusable on its own becomes a separate skill. Many small focused skills beat one monolithic skill — they compose, they get triggered precisely, and other developers can adopt them piecemeal.

## 6. Caveman lite for prompt text
Write SKILL.md, command, and agent prompt text in caveman mode lite. Drop articles (a/an/the), filler (just/really/basically/simply), and hedging. Keep technical terms exact. Code blocks unchanged. Goal: concise prompts that get to the point.

## 7. Update README on every plugin change
Adding or editing any plugin under `plugins/` requires updating `README.md` in same change — install line, usage, dependencies. New plugin: add section. Removed plugin: drop section. Write README in caveman lite (Rule 6). Stale README = bug.

## 8. Names must be unique
Every plugin, skill, command, and agent name unique across whole marketplace. No two tools share a name. Duplicate names cause trigger collisions and shadow users' other installed plugins. Prefer descriptive, namespaced names (`wiki-ingest`, not `ingest`).

## 9. Skills must be skills.sh-discoverable
Every skill needs valid `SKILL.md` frontmatter (`name` + `description`) and a CLI-discoverable path, and plugin skills must be grouped in root `skills.sh.json`. See [`.claude/rules/skills-distribution.md`](skills-distribution.md). Verify with `npx skills add . --list`.

## 10. Hooks live in hooks/hooks.json
Plugin hooks go in `plugins/<plugin>/hooks/hooks.json`. Never user/project `settings.json` or `.claude/hooks/` — those are out-of-plugin scope. Optional top-level `description`, then `hooks` object.

**Shape — 3 levels:** event → matcher group → handler.

```json
{
  "description": "what these hooks do",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PLUGIN_ROOT}/scripts/check.sh", "args": [] }
        ]
      }
    ]
  }
}
```

**Events (when fire):** `SessionStart`/`SessionEnd` (per session); `UserPromptSubmit`, `Stop` (per turn); `PreToolUse`, `PostToolUse`, `PostToolUseFailure` (per tool call); `SubagentStart`/`SubagentStop`, `PreCompact`/`PostCompact`, `Notification`. Full list → docs.

**Matchers** (filter when hook fires): `*`/`""`/omit = all; letters+digits+`_`+`|` = exact or `|`-list (`Edit|Write`); other chars = regex. Tool events match `tool_name`. MCP tools are `mcp__<server>__<tool>` — `mcp__memory__.*` matches all of a server's tools. Narrow tool+args with handler `if`: `"Bash(git *)"`, `"Edit(*.ts)"`.

**Handler types:** `command` (shell; stdin JSON → exit code/stdout), `http` (POST), `mcp_tool` (call connected MCP), `prompt` (LLM yes/no), `agent` (subagent verifier).

**Command handler:** `command` required. With `args` = exec form (no shell, each arg literal — use for path placeholders). Without `args` = shell form (pipes, `&&`). `async`/`asyncRewake` run background. `timeout` in seconds.

**Path placeholders:** `${CLAUDE_PLUGIN_ROOT}` (bundled scripts — use this), `${CLAUDE_PLUGIN_DATA}` (persistent data), `${CLAUDE_PROJECT_DIR}` (project root). Never relative paths.

**Input:** command hook gets JSON on **stdin** — `session_id`, `cwd`, `hook_event_name`. Tool events add `tool_name`, `tool_input` (+`tool_response` on PostToolUse). Read file path via `jq -r '.tool_input.file_path'`. No `$CLAUDE_TOOL_*` env vars exist.

**Output + exit codes:** exit 0 = ok (stdout parsed for JSON); exit 2 = block (stderr → Claude); other = non-blocking error. Or exit 0 + JSON: `continue:false` stops; PreToolUse `hookSpecificOutput.permissionDecision` (`allow`/`deny`/`ask`); `additionalContext` injects context. Use exit codes OR JSON — not both.

**Security:** validate stdin, quote `"$VAR"`, no secrets (Rule 2), absolute paths via `${CLAUDE_PLUGIN_ROOT}`, skip `.env`/`.git`.

Full reference: https://code.claude.com/docs/en/hooks
