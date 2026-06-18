# Workflow Design Bible

**A constitution generator for autonomous, agent-run projects.**

`workflow-design-bible` is a reusable **meta system prompt** (and an installable
agent skill). Point it at a brand-new project idea — a content channel, an ebook
press, an SEO tool-site, a web product, a casual game, anything meant to *run
itself* — and it interviews you, then scaffolds a complete **"constitution +
documentation system"**: a lean root `CLAUDE.md`, a brand `configuration.json`,
and a `documentation/` folder of single-source-of-truth docs, plus the empty
`.claude/agents/` and `.claude/skills/` registries to grow into.

It encodes one opinionated architecture that has proven itself on real autonomous
projects:

> **The main agent is a CEO, not a worker.** It orchestrates; sub-agents do the
> work; skills document capabilities; a deterministic CLI executes them. Every
> pipeline ends by reflecting on itself and freezing "decided on the fly" into
> "frozen into a function."

— distilled into **eight design philosophies** every generated project inherits.

---

## Why this exists

LLM agents are great at *doing a task* and bad at *running a project*. Without a
durable structure they re-decide the same things every session, bloat their own
context, lose track of what's already built, and never improve their own process.

This Bible bakes in the structure: a CEO/sub-agent split, four clean capability
layers, single-source-of-truth pointers so the constitution never bloats, a
two-tier global/local capability model, a built-in self-consistency check, and a
reflection loop that compounds. You answer a short interview; you get a project
that knows how to run and improve itself.

## The eight design philosophies

1. **The main agent is a CEO, not a worker** — orchestrate, don't do manual labor.
2. **Everything is a sub-agent; concurrency is the default** — fan out at the seams.
3. **Four-layer architecture** — CEO → sub-agent prompt → skill → MCP/CLI; pointers down, details never up.
4. **Single source of truth + pointers** — the constitution stays lean forever.
5. **Global vs local capabilities** — reuse what's shared, build only what's unique.
6. **Reflection is always the last step** — two loops: per-cycle + cross-session.
7. **Constitution-as-code** — a `doctor` command keeps the docs' claims equal to filesystem reality.
8. **A standard project shape** — root constitution + `documentation/` from birth.

Full detail lives in [`SKILL.md`](./skills/workflow-design-bible/SKILL.md).

## How to use it

**Option A — install the plugin (recommended).** This repo is a self-contained
[Claude Code](https://claude.com/claude-code) plugin **and** marketplace. Add the
marketplace, then install the plugin — from inside Claude Code:

```text
/plugin marketplace add hly990/workflow-design-bible
/plugin install workflow-design-bible@workflow-design-bible
```

…or from the terminal with the `claude` CLI:

```bash
claude plugin marketplace add hly990/workflow-design-bible
claude plugin install workflow-design-bible@workflow-design-bible
```

Then just say *"start a new autonomous project"* (or describe the project) and the
skill runs the interview and generates the constitution skeleton.

**Option B — read the doc (no runtime).** Open
[`skills/workflow-design-bible/SKILL.md`](./skills/workflow-design-bible/SKILL.md)
(and the [`templates/`](./skills/workflow-design-bible/templates)) and paste it
into any capable LLM. Follow the interview protocol; fill in the templates by
hand. No runtime required.

> **Manual install fallback.** You can also clone the repo and copy the skill
> folder into your runtime's skills directory, e.g. for Claude Code:
> `cp -r skills/workflow-design-bible ~/.claude/skills/workflow-design-bible`,
> then reload skills. Prefer the plugin flow above.

> Filenames like `CLAUDE.md` are conventions — substitute whatever your runtime
> reads as top-level agent instructions (e.g. `AGENTS.md`). The ideas are
> runtime-agnostic.

## What's in this repo

| Path | What it is |
|---|---|
| [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) | The Claude Code plugin manifest. |
| [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json) | The in-repo marketplace listing this plugin. |
| [`skills/workflow-design-bible/SKILL.md`](./skills/workflow-design-bible/SKILL.md) | The meta system prompt: philosophies + interview protocol + generation rules. The main artifact. |
| [`skills/workflow-design-bible/templates/CLAUDE.md.template`](./skills/workflow-design-bible/templates/CLAUDE.md.template) | The fill-in constitution skeleton (Rules **and** Don'ts). |
| [`skills/workflow-design-bible/templates/configuration.json.template`](./skills/workflow-design-bible/templates/configuration.json.template) | The brand single-source-of-truth. |
| [`CHANGELOG.md`](./CHANGELOG.md) | Version history (semver). |
| [`VERSION`](./VERSION) | Current version, for tooling / online upgrades. |

## What it does *not* do

It runs no business code, writes no application logic, and deploys nothing. It
only does **interview → generate the documentation skeleton + registries**. The
actual CLI, sub-agent prompts, and capability skills are grown later by the
project's own CEO + maintainer agent.

## Versioning & upgrades

Semantic versioning. The version is mirrored in `VERSION`, the `version:` field of
`skills/workflow-design-bible/SKILL.md` frontmatter, `.claude-plugin/plugin.json`,
the plugin entry in `.claude-plugin/marketplace.json`, and `CHANGELOG.md`. To
upgrade an installed copy, run `claude plugin marketplace update workflow-design-bible`
then `claude plugin install workflow-design-bible@workflow-design-bible` (or
`/plugin` from inside Claude Code). Watch releases for new versions.

## Contributing

Issues and PRs welcome — especially new project archetypes, sharper interview
questions, and template improvements. Keep the core principle intact: the
constitution stays lean; detail sinks into pointers.

## License

[MIT](./LICENSE) © 2026 preangelleo
