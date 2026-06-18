# Changelog

All notable changes to the **Workflow Design Bible** are documented here.
This project follows [Semantic Versioning](https://semver.org/).

The skill version is mirrored in three places that must stay in sync:
`VERSION`, the `version:` field in `SKILL.md` frontmatter, and the top entry here.

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
