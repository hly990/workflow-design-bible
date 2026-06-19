# Changelog

All notable changes to the **Workflow Design Bible** are documented here.
This project follows [Semantic Versioning](https://semver.org/).

The version is mirrored in places that must stay in sync: `VERSION`, the
`version:` field in `skills/workflow-design-bible/SKILL.md` frontmatter,
`.claude-plugin/plugin.json`, the plugin entry in `.claude-plugin/marketplace.json`,
and the top entry here.

## [3.0.0] — 2026-06-20

Upgraded from a workflow design guide into the **Loop Design Bible**: the repo now
teaches you to **design self-running agent loops, not just write prompts**, on top of
the existing constitution generator.

### Added
- **The eight-part loop architecture** — Trigger · Context Injection · Agent Role ·
  Tools · Verification · State · Retry · Escalation — as the spine of the whole Bible.
- **`docs/01–06`** — a six-part curriculum: loop thinking, loop anatomy,
  verification & state (the two load-bearing parts), retry & escalation, a
  workflow→loop bridge mapping the eight philosophies to the eight parts, and an
  agent-loop pattern catalog.
- **`templates/loop-blueprint.md`** — design one loop (the eight parts with blanks).
- **`templates/verification-checklist.md`** — design/run the verification gate (part ⑤).
- **`templates/memory-state-template.md`** — design what persists between iterations (part ⑥).
- **`templates/skill-template.md`** — freeze a recurring action into a capability (part ④).
- **`examples/coding-agent-loop.md`, `content-production-loop.md`,
  `customer-acquisition-loop.md`** — three fully-filled, worked loop designs.

### Changed
- **README.md** reframed around "design the loop, not the prompt," with a navigation
  table for the new docs/templates/examples.
- **`SKILL.md`** gains a new **§0 "The loop lens"** (the eight parts + the Bible map)
  and ties Round 3 of the interview to the loop blueprint; the constitution's pipeline
  spine is now explicitly a chain of loops. Frontmatter/title repositioned to "Loop
  Design Bible." Display name updated in the plugin + marketplace manifests.



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
