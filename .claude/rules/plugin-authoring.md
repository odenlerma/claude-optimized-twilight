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
