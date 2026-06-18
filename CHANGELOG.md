# Changelog

All notable changes to the **Workflow Design Bible** are documented here.
This project follows [Semantic Versioning](https://semver.org/).

The version is mirrored in places that must stay in sync: `VERSION`, the
`version:` field in `skills/workflow-design-bible/SKILL.md` frontmatter,
`.claude-plugin/plugin.json`, the plugin entry in `.claude-plugin/marketplace.json`,
and the top entry here.

## [2.0.0] — 2026-06-19

Repackaged as an installable **Claude Code plugin** (with an in-repo marketplace).

### Added
- **`.claude-plugin/plugin.json`** — the plugin manifest, so the repo installs
  as a Claude Code plugin.
- **`.claude-plugin/marketplace.json`** — an in-repo marketplace listing the
  plugin, so users can add this repo as a marketplace and install in one step.

### Changed
- **BREAKING (install layout):** the skill moved from the repo root to
  `skills/workflow-design-bible/SKILL.md` (with its `templates/` alongside) so
  Claude Code's plugin loader auto-discovers it. The old
  `git clone … ~/.claude/skills/workflow-design-bible` install no longer works
  as-is; install via the plugin/marketplace flow (see README) instead.
- README install instructions rewritten around `/plugin marketplace add` +
  `/plugin install`.

## [1.0.0] — 2026-06-18

First public release.

### Added
- **`SKILL.md`** — the installable meta system prompt: an interview-driven
  generator that scaffolds a complete "constitution + documentation system"
  for a new autonomous, agent-run project.
- **Eight design philosophies** — the reusable DNA every generated project
  constitution must embody (CEO model, all-sub-agent + concurrency, four-layer
  architecture, single-source-of-truth pointers, global/local capability tiers,
  reflection as the last step, constitution-as-code self-check, standard shape).
- **`templates/CLAUDE.md.template`** — the fill-in constitution skeleton, now
  with a **Rules / Don'ts** split (hard prohibitions vs discouraged actions).
- **`templates/configuration.json.template`** — the brand single-source-of-truth.
- Interview protocol that **auto-surveys the host environment** for reusable
  global skills/agents/MCP instead of asking the user to enumerate them.
- Self-consistency (`doctor`) pattern, dry-run/test-safety rule, locked
  strategic-decisions table, and session-handoff loop baked into the templates.
