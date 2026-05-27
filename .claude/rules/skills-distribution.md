# Skills distribution rules (skills.sh)

Applies to **every** skill in repo — both `plugins/*/skills/` and standalone
`.agents/skills/`. Broader scope than `plugin-authoring.md` (which covers only
`plugins/`).

Repo lists on [skills.sh](https://www.skills.sh) via the open `skills` CLI
(`npx skills`, [vercel-labs/skills](https://github.com/vercel-labs/skills)). Listing is
**automatic and telemetry-driven** — no submission. Repo gets ranked once anyone runs
`npx skills add odenlerma/claude-optimized-twilight`. Skill stays invisible until CLI
can discover it. Rules below keep skills discoverable.

## 1. Valid SKILL.md frontmatter
Every skill ships `SKILL.md` with YAML frontmatter holding non-empty `name` and
`description`. Missing or empty either field → CLI skips skill → absent from skills.sh.

## 2. Name matches dir, stays unique
`name` matches skill's directory name. Unique across whole catalog (reinforces
plugin-authoring Rule 8). Collisions break discovery and shadow other skills.

## 3. Discoverable path only
Skills live in CLI-scanned paths:
- `.agents/skills/<name>/SKILL.md`
- `.claude/skills/<name>/SKILL.md`
- `plugins/<plugin>/skills/<name>/SKILL.md` (declared via `.claude-plugin` manifest)

Skills elsewhere fall to CLI recursive fallback — may not list cleanly. Avoid.

## 4. Group plugin skills in skills.sh.json
New or renamed **plugin** skill → add to matching group in root `skills.sh.json`.
Vendored third-party skills under `.agents/skills/` (`caveman`, `caveman-review`,
`skill-creator`, sourced per `skills-lock.json`) stay ungrouped — `notGrouped: "bottom"`
drops them in auto "Other skills". Do not promote vendored skills in groupings.

## 5. Verify before commit
Run `npx skills add . --list` from repo root. Lists every skill CLI exposes (no
install). Confirm each authored skill shows. `/validate` runs this check (Phase 1b).
