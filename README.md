# Loop Design Bible

**Design self-running agent *loops*, not just prompts — and scaffold the constitution that runs them.**

`workflow-design-bible` is a reusable **meta system prompt** (and an installable
[Claude Code](https://claude.com/claude-code) plugin) built on one idea:

> **The moment you want an agent to *run something* instead of *answer something*,
> you have stopped writing prompts and started designing loops.** A prompt produces
> one output. A loop produces a system that keeps producing outputs — and gets
> better at it.

So this repo is two things at once. It is a **Bible for loop design** — a curriculum
that teaches the eight-part architecture every agent loop is made of — and it is a
**constitution generator**: point it at a brand-new self-running project (a content
channel, an ebook press, an SEO tool-site, a web product, a casual game, anything
meant to run itself) and it interviews you, then scaffolds a complete "constitution +
documentation system" whose pipeline spine *is* a chain of the loops you designed.

---

## The eight parts of every agent loop

Stop sharpening one prompt. Design the **iteration** — these eight parts, in order:

| # | Part | Answers | Skip it and you get |
|---|---|---|---|
| ① | **Trigger** | What starts an iteration? | You are the trigger; no real loop |
| ② | **Context Injection** | What does it know? | Context bloat, drift, rising cost |
| ③ | **Agent Role** | Who runs it? | A do-everything agent that can't scale |
| ④ | **Tools** | What can it do? | Fragile re-derived actions, wasted tokens |
| ⑤ | **Verification** | How do you know it worked? | Silent quality drift into slop |
| ⑥ | **State** | What does it remember? | Groundhog Day; never improves |
| ⑦ | **Retry** | What on failure? | A hiccup kills the run (or hangs forever) |
| ⑧ | **Escalation** | When ask a human? | Constant pinging, or fabrication past blockers |

```
① TRIGGER → ② CONTEXT INJECTION → ③ AGENT ROLE → ④ TOOLS
                                                    │
   ⑥ STATE ◀──[pass]── ⑤ VERIFICATION ──[fail]──▶ ⑦ RETRY ──▶ ⑧ ESCALATION
```

**Verification and State are load-bearing.** Verification is the only part that can
say *no* (it turns repetition into improvement instead of decay). State is the only
part that survives the context window (it turns "ran a thousand times" into "learned
a thousand times"). Get those two right and the loop forgives weakness elsewhere.

## Why this exists

LLM agents are great at *doing a task* and bad at *running a project*. Without a
durable loop they re-decide the same things every session, bloat their own context,
drift into slop because nothing ever says *no*, stall on the first failure, and never
improve. **None of those are prompt problems — you cannot word your way out of them.**
They are loop-architecture problems, solved by designing the eight parts above. This
Bible teaches that design, then bakes it into a project scaffold so a new autonomous
project is born knowing how to run and improve itself.

## Read the Bible

Six docs, four templates, three worked examples — a full curriculum at the repo root.

**Curriculum (`docs/`)**

| Doc | What it covers |
|---|---|
| [`01-loop-thinking.md`](./docs/01-loop-thinking.md) | The paradigm shift: design the loop, not the prompt. |
| [`02-loop-anatomy.md`](./docs/02-loop-anatomy.md) | The eight parts, each with its design questions and failure mode. |
| [`03-verification-and-state.md`](./docs/03-verification-and-state.md) | The two load-bearing parts, in depth. |
| [`04-retry-and-escalation.md`](./docs/04-retry-and-escalation.md) | Making failure safe: bounded retry, sharp escalation, anti-fabrication. |
| [`05-from-workflow-to-loop.md`](./docs/05-from-workflow-to-loop.md) | The bridge: the CEO philosophies and the eight parts are one machine. |
| [`06-agent-loop-patterns.md`](./docs/06-agent-loop-patterns.md) | A catalog of proven loop shapes to steal. |

**Design tools (`templates/`)**

| Template | Use it to |
|---|---|
| [`loop-blueprint.md`](./templates/loop-blueprint.md) | Design one loop — the eight parts with blanks to fill. |
| [`verification-checklist.md`](./templates/verification-checklist.md) | Design and run the verification gate (part ⑤). |
| [`memory-state-template.md`](./templates/memory-state-template.md) | Design what persists between iterations (part ⑥). |
| [`skill-template.md`](./templates/skill-template.md) | Freeze a recurring action into a reusable capability (part ④). |

**Worked designs (`examples/`)** — three fully-filled blueprints:
[`coding-agent-loop.md`](./examples/coding-agent-loop.md) ·
[`content-production-loop.md`](./examples/content-production-loop.md) ·
[`customer-acquisition-loop.md`](./examples/customer-acquisition-loop.md).

## The constitution generator (the scaffold side)

When you start a real project, the skill turns loop thinking into structure. It
encodes one opinionated architecture proven on real autonomous projects:

> **The main agent is a CEO, not a worker.** It orchestrates; sub-agents do the
> work; skills document capabilities; a deterministic CLI executes them. Every
> pipeline ends by reflecting on itself and freezing "decided on the fly" into
> "frozen into a function."

— distilled into **eight design philosophies** every generated project inherits. They
are the same machine as the eight loop parts, seen from the architecture angle (the
mapping is [`05-from-workflow-to-loop.md`](./docs/05-from-workflow-to-loop.md)):

1. **The main agent is a CEO, not a worker** — orchestrate, don't do manual labor.
2. **Everything is a sub-agent; concurrency is the default** — fan out at the seams.
3. **Four-layer architecture** — CEO → sub-agent prompt → skill → MCP/CLI; pointers down, details never up.
4. **Single source of truth + pointers** — the constitution stays lean forever.
5. **Global vs local capabilities** — reuse what's shared, build only what's unique.
6. **Reflection is always the last step** — the outer loop that rewrites the inner loop.
7. **Constitution-as-code** — a `doctor` command keeps the docs' claims equal to filesystem reality.
8. **A standard project shape** — root constitution + `documentation/` from birth.

The full skill — philosophies, interview protocol, and generation rules — lives in
[`SKILL.md`](./skills/workflow-design-bible/SKILL.md). Its pipeline spine is filled
with loops you design using the templates above.

## How to use it

**Option A — install the plugin (recommended).** This repo is a self-contained
Claude Code plugin **and** marketplace. Add the marketplace, then install the plugin
— from inside Claude Code:

```text
/plugin marketplace add hly990/workflow-design-bible
/plugin install workflow-design-bible@workflow-design-bible
```

…or from the terminal with the `claude` CLI:

```bash
claude plugin marketplace add hly990/workflow-design-bible
claude plugin install workflow-design-bible@workflow-design-bible
```

Then say *"design a loop for X"* (to design a single self-running loop) or *"start a
new autonomous project"* (to run the interview and scaffold the constitution).

**Option B — read the docs (no runtime).** Open the [`docs/`](./docs/01-loop-thinking.md),
fill in a [`templates/loop-blueprint.md`](./templates/loop-blueprint.md) by hand, and
paste [`SKILL.md`](./skills/workflow-design-bible/SKILL.md) into any capable LLM to
run the interview manually. No runtime required.

> **Manual install fallback.** You can also clone the repo and copy the skill folder
> into your runtime's skills directory, e.g. for Claude Code:
> `cp -r skills/workflow-design-bible ~/.claude/skills/workflow-design-bible`, then
> reload skills. Prefer the plugin flow above.

> Filenames like `CLAUDE.md` are conventions — substitute whatever your runtime reads
> as top-level agent instructions (e.g. `AGENTS.md`). The ideas are runtime-agnostic.

## What's in this repo

| Path | What it is |
|---|---|
| [`docs/`](./docs/01-loop-thinking.md) | The Loop Design Bible curriculum (six docs, 01–06). |
| [`templates/loop-blueprint.md`](./templates/loop-blueprint.md) | Design one loop — the eight parts with blanks. |
| [`templates/verification-checklist.md`](./templates/verification-checklist.md) | Design the verification gate (part ⑤). |
| [`templates/memory-state-template.md`](./templates/memory-state-template.md) | Design the state layer (part ⑥). |
| [`templates/skill-template.md`](./templates/skill-template.md) | Freeze a tool into a capability (part ④). |
| [`examples/`](./examples/coding-agent-loop.md) | Three worked, fully-filled loop designs. |
| [`skills/workflow-design-bible/SKILL.md`](./skills/workflow-design-bible/SKILL.md) | The meta system prompt: loop lens + philosophies + interview + generation rules. |
| [`skills/workflow-design-bible/templates/CLAUDE.md.template`](./skills/workflow-design-bible/templates/CLAUDE.md.template) | The fill-in constitution skeleton (Rules **and** Don'ts). |
| [`skills/workflow-design-bible/templates/configuration.json.template`](./skills/workflow-design-bible/templates/configuration.json.template) | The brand single-source-of-truth. |
| [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) | The Claude Code plugin manifest. |
| [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json) | The in-repo marketplace listing this plugin. |
| [`CHANGELOG.md`](./CHANGELOG.md) | Version history (semver). |
| [`VERSION`](./VERSION) | Current version, for tooling / online upgrades. |

## What it does *not* do

It runs no business code, writes no application logic, and deploys nothing. It teaches
loop design and does **interview → generate the documentation skeleton + registries**.
The actual CLI, sub-agent prompts, and capability skills are grown later by the
project's own CEO + maintainer agent — each one a loop you design with the blueprint.

## Versioning & upgrades

Semantic versioning. The version is mirrored in `VERSION`, the `version:` field of
`skills/workflow-design-bible/SKILL.md` frontmatter, `.claude-plugin/plugin.json`,
the plugin entry in `.claude-plugin/marketplace.json`, and `CHANGELOG.md`. To upgrade
an installed copy, run `claude plugin marketplace update workflow-design-bible` then
`claude plugin install workflow-design-bible@workflow-design-bible` (or `/plugin` from
inside Claude Code). Watch releases for new versions.

## Contributing

Issues and PRs welcome — especially new loop patterns, sharper verification gates, new
worked examples, and template improvements. Keep the core principle intact: design the
loop, not just the prompt; the constitution stays lean; detail sinks into pointers.

## License

[MIT](./LICENSE) © 2026 preangelleo
