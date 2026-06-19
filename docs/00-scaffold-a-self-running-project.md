---
name: scaffold-self-running-project
description: A complete, self-contained skill that takes a raw project idea and scaffolds a self-running, agent-run autonomous project — designed loop-first. It teaches the eight-part loop architecture (Trigger · Context Injection · Agent Role · Tools · Verification · State · Retry · Escalation), runs a short intake interview, then emits a CEO-orchestrated constitution (a lean CLAUDE.md), a brand configuration.json, and a documentation/ skeleton. Everything needed is inline — paste this one file into any capable LLM and run it. Triggers - "scaffold a self-running project", "design the loops for a new autonomous project", "set up an agent-run project from scratch", "turn this idea into a project that runs itself", "generate a CLAUDE.md / AGENTS.md constitution", "bootstrap a self-running pipeline".
version: 1.0.0
license: MIT
---

# Scaffold a self-running project — a loop-first, self-contained skill

> **What this skill is.** A single, self-contained operating procedure that turns a
> project idea into a **self-running autonomous project**. You design the project as
> a set of **loops** (the eight parts below), then assemble those loops into a
> **CEO-orchestrated constitution** — a lean `CLAUDE.md` plus a `documentation/`
> skeleton — that an agent can run, session after session, with minimal babysitting.
>
> **What this skill is NOT.** It writes no business code, ships nothing, deploys
> nothing. It does **idea → loop design → constitution + docs skeleton**. The real
> CLI, sub-agent prompts, and capability skills are grown *later* by the project's own
> CEO + maintainer agent — each one a loop you design with the blueprint in §5.
>
> **It is fully self-contained.** The three templates you need (constitution,
> brand config, loop blueprint) are embedded inline in §5 — paste this one file into
> any capable LLM and follow it; no other files required. The links to
> [`01-loop-thinking.md`](./01-loop-thinking.md) … [`06-agent-loop-patterns.md`](./06-agent-loop-patterns.md)
> and the [`../README.md`](../README.md) are **enrichment**, not dependencies: read
> them to go deeper, skip them and this still works.
>
> **Conventions.** "CEO" = the main/orchestrating agent. "The user" = the human owner
> who sets direction and signs off. `CLAUDE.md` is a convention — substitute whatever
> your runtime reads as top-level agent instructions (e.g. `AGENTS.md`).

---

## How to use this skill (one screen)

1. **Read Part 1** so you think in loops, not prompts.
2. **Run the intake interview** (§3) — a few rounds of mostly multiple-choice questions.
3. **Design each pipeline step as a loop** using the inline blueprint (§5c).
4. **Emit the files** per the generation rules (§4), filling the inline templates (§5).
5. **Run the closing self-check** (§6). Ship the skeleton; hand off to the project CEO.

Two — and only two — reasons to stop and ask the human while running: a **missing
credential/authorization**, or a **subjective/business/value judgment**. Everything
else you decide and proceed.

---

## Part 1 · Design loops, not prompts

A prompt produces one output. A **loop** produces a system that keeps producing
outputs — and improves at it. The moment a project needs to *run* rather than
*answer*, you stop writing prompts and design **loops**. Every self-running system,
at every altitude, is one **iteration** built from eight parts:

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

Read it as a sentence: *something triggers an iteration; the right context is injected
into the right role with the right tools; the role acts; the result is verified; on
pass it commits to state and is ready to run again; on fail it retries a bounded number
of times, then escalates to a human.* Eight parts. Miss one and the loop has a hole
production will find.

> **Verification and State are load-bearing — get these two right above all else.**
> **Verification (⑤)** is the only part that can say *no*; it turns repetition into
> improvement instead of decay ("the agent said done" is **not** verification — a real
> gate is something the agent cannot talk its way past). **State (⑥)** is the only part
> that survives the context window; it turns "ran a thousand times" into "learned a
> thousand times." A project strong in both forgives weakness in the other six.

**Loops are fractal.** A project is a loop whose iterations are pipelines; a pipeline
is a loop whose steps are micro-loops. You design the same eight parts at each level
and they nest. That is why one model scaffolds the whole project. Depth on each part:
[`02-loop-anatomy.md`](./02-loop-anatomy.md); the two hard parts:
[`03-verification-and-state.md`](./03-verification-and-state.md); safe failure:
[`04-retry-and-escalation.md`](./04-retry-and-escalation.md).

---

## Part 2 · From loops to a constitution (the CEO architecture)

A self-running project is a **chain of loops** with a boss. The boss is the **CEO**
(the main agent); the loops are run by **sub-agents**. Eight philosophies turn a pile
of loops into a coherent, self-improving project. They are the same machine as the
eight parts, seen from the architecture angle:

1. **The main agent is a CEO, not a worker** — it orchestrates, supervises, verifies,
   and talks to the user; every fixed step is delegated. *(Agent Role ③)*
2. **Everything is a sub-agent; concurrency is the default** — independent loops fan
   out; homogeneous work batches. *(Agent Role ③ + Trigger ①)*
3. **Four-layer architecture** — CEO → sub-agent system prompt → skill → MCP/CLI;
   pointers point **down**, detail never leaks **up**. *(Tools ④)*
4. **Single source of truth + pointers** — `CLAUDE.md` holds principles, the pipeline
   spine, and pointers; everything concrete sinks into `documentation/`, loaded on
   demand. The constitution never bloats. *(Context Injection ②)*
5. **Global vs local capabilities** — reuse shared skills/MCP/agents across projects;
   build only what's unique here. *(Tools ④)*
6. **Reflection is always the last step** — and it is not a step, it is the **outer
   loop that rewrites the inner loop**: each cycle attributes flaws, then a maintainer
   agent upgrades the tools/templates so "decided on the fly" becomes "frozen into a
   function." *(the engine that makes State ⑥ compound)*
7. **Constitution-as-code** — a deterministic `doctor` command checks the docs' claims
   against filesystem reality and exits non-zero on drift. *(Verification ⑤)*
8. **A standard project shape** — root constitution + `documentation/` from birth.

```
① CEO (CLAUDE.md)            — assigns work, sets principles, touches no details
   ↓ dispatch a sub-agent with a self-contained task brief
② Sub-agent (its system prompt)  — role + "which skills this role mainly uses" (pointers)
   ↓ invoke a skill
③ Skill (its SKILL.md)       — how one capability is used; declares its MCP servers + CLI commands
   ↓ execute
④ MCP servers + CLI tools    — executed, never loaded into context
```

> **The bridge in one line.** Each row of the project's pipeline spine is a loop;
> the constitution is the loops, written down. Reflection (philosophy 6) is the outer
> loop that keeps rewriting the inner ones. Full mapping:
> [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md).

---

## Part 3 · The intake interview

Ask in rounds; offer multiple-choice wherever possible to minimize the user's typing.
Echo each answer back, then proceed. The CEO model, the five/six core rules, and the
four-layer architecture are **constants** — do not interview for them; they come
pre-filled (§5a). Batch related questions (~4 at a time); finish in 3–4 rounds.

### Round 0 · Project archetype (selects the pipeline-spine draft)
| Archetype | Typical spine (preloaded draft, then tune) |
|---|---|
| **Content — video/channel** | topic → script → render → publish → engage → reflect |
| **Content — ebook/publishing** | thesis → write → compile → distribute → market → reflect |
| **SEO traffic asset** | topic → build → ship → monetize → SEO/monitor → reflect |
| **Web product/site** | requirements → design → build → test → deploy → monitor → reflect |
| **Casual game** | concept → asset batch → build → test → publish → data-tune → reflect |
| **Other (custom)** | co-design the step-by-step from scratch |

### Round 1 · Mission, North Star, and red lines
- One-sentence mission (becomes the CEO's North Star).
- 3–5 **priority-ordered** non-negotiable constraints (earlier always outranks later).
- The **#1 iron law / compliance red line**, if any ("violate it and everything is wasted").
- **Don'ts in two tiers:** *Forbidden* (red-line actions that void the project) vs
  *Discouraged* (avoid unless justified).

### Round 2 · Brand (→ configuration.json)
- Brand/codename; attribution entity (real name? pen name? company?); per-language?
- Slogan; logo assets (if any).
- **Design language:** primary colors, fonts, tone keywords, visual style, taboos.
- Trust/E-E-A-T anchor; payment/account entity (if monetized).

### Round 3 · Pipeline spine — *every step is a loop*
- From the archetype draft, nail down **each step**: what it does → its output → lead role.
- Mark **which steps fan out** (parallelism points).
- Confirm the last step = reflection & self-iterate.
- **For each step, settle at minimum its loop's** ① Trigger (what hands it work),
  ⑤ Verification gate (how its output is proven), and ⑧ Escalation rule. Use the inline
  **loop blueprint (§5c)** for any step complex enough to deserve its own eight-part design.

### Round 4 · Roles + global/local capabilities
- One sub-agent per fixed step: name + one-line role + which step. Always include a
  `dev-maintainer` (owns all code/SP/skill changes).
- Prefer **one shared maintainer** unless an artifact has genuinely distinct dependencies.
- Auto-survey the host environment for reusable **global** skills/agents/MCP and propose
  a reuse list; list the **local** skills to build/fork — and for each, which MCP + CLI
  it is built from. For each sub-agent, name the skills it should mainly use.

### Round 5 · State (lifeline memory) + single-source docs + credentials
- What **long-term State** does the project need (asset DB / dedup store / progress
  ledger…)? What fields? This is loop part ⑥ for the whole project.
- Which rules are single sources of truth → each becomes a `documentation/<topic>.md`?
- Current phase / ramp-up cadence? Which private credentials must the user provide
  (→ `INITIALIZATION.md`)? Confirm the two-reasons-to-stop boundary (loop part ⑧).

> When the interview is done, **echo back every filled key field** and confirm before
> writing any files.

---

## Part 4 · Generation rules (what to emit)

1. Fill the inline **constitution template (§5a)**; replace every `{{…}}`; delete
   sections that don't apply. **Anything longer than a few lines becomes "one line +
   a pointer to `documentation/<x>.md`."** Keep it lean — it is resident in context
   on every turn.
2. Fill the inline **`configuration.json` template (§5b)**; put **all concrete brand
   values** (pricing/colors/attribution) only there; the constitution references them
   by pointer.
3. Generate `documentation/INITIALIZATION.md` (credentials + one-time deploy) and one
   `documentation/<topic>.md` per single-source rule from Round 5.
4. Create the empty skeletons: `.claude/agents/`, `.claude/skills/`, `reflections/`,
   `reports/`, `HANDOFF.md` (placeholders).
5. Recommend (do not implement) a `doctor` / consistency-lint subcommand for the
   project's future CLI — the machine backstop for philosophy 7 (Verification ⑤).

**Standard project shape emitted:**
```
<project>/
├── CLAUDE.md              ← constitution (resident in context): principles · spine · pointers · map
├── documentation/         ← every single source of truth / config (loaded on demand)
│   ├── configuration.json     brand single source of truth
│   ├── INITIALIZATION.md      one-time first-deploy setup
│   └── <topic>.md             topic playbooks (quality gate / compliance / SOP / …)
├── .claude/agents/        ← sub-agent system prompts (role definitions)
├── .claude/skills/        ← each SKILL.md (capability definitions)
├── reflections/           ← per-cycle reflection reports (permanent State)
├── reports/               ← async analytics, consumed at session start
├── HANDOFF.md             ← cross-session handoff of unfinished work
└── <lifeline>.db          ← the lifeline memory store (gitignored State)
```

---

## Part 5 · Inline templates (self-contained)

### 5a · Constitution skeleton (`CLAUDE.md`)
```markdown
# {{PROJECT_NAME}} — {{ONE_LINE_POSITIONING}}

> **This file = the project constitution (the CEO's charter).** It carries only
> operating principles, the pipeline spine, pointers, and the file map. Details sink
> into documentation/, into each .claude/skills/<name>/SKILL.md, and into sub-agent
> system prompts (loaded on demand).
> ### 📖 Extended docs (read on demand)
> - documentation/configuration.json — brand single source of truth
> - documentation/INITIALIZATION.md — one-time first-deploy setup
> - documentation/{{topic}}.md — {{one line}}

## 🎯 Mission (the CEO's North Star)
> **{{one-sentence mission}}**
Non-negotiable constraints, priority-ordered (earlier outranks later):
1. **{{constraint A}}** — {{why}}
2. **{{constraint B}}** — {{why}}
**Go/no-go test:** {{do it only if ALL pass; stop if ANY fails}}

## 🛡️ #1 Iron Law: {{compliance red line}}   ← if any
{{red-line statement + the hard gate command + pointer to documentation/{{compliance}}.md}}

## 🏛️ Operating model: main agent = CEO (orchestrate · supervise · verify · interact)
### ✅ Rules (always do)
1. **Delegate by default** — except judgment/review/sign-off/talking to the user.
2. **Parallelize over serialize** — fan out independent work; batch homogeneous tasks.
3. **Background over foreground** — long tasks run in the background.
4. **The CEO verifies before reporting** — never trust a sub-agent's "done"; read it / curl it / query the DB. (loop part ⑤)
5. **Fix the code, not the artifact** — change the CLI/template/skill/SP; dispatch dev-maintainer. (loop part ⑦)
6. **Test in dry-run** — never run a real publish/deploy/charge as a test.
### 🚫 Don'ts
**Forbidden (red lines — any one can void the project):** {{forbidden 1}} · {{forbidden 2}}
**Discouraged (avoid unless justified):** {{discouraged 1}} · {{discouraged 2}}
> **Standing authorization:** solve problems yourself; stop and ask only for ① a missing
> credential/authorization, or ② a subjective/business/value judgment. (loop part ⑧)

## {{emoji}} {{PROJECT}} pipeline (the spine — each row is a loop)
| Step | What it does | Lead sub-agent | Output | Trigger ① | Verify ⑤ |
|---|---|---|---|---|---|
| ① {{…}} | {{…}} | `{{agent}}` | {{output}} | {{what hands it work}} | {{the gate}} |
| ⑦ Reflection & self-iterate | review flaws → attribute → dispatch dev-maintainer to upgrade → consolidate. **Always last** (→ documentation/reflection_playbook.md) | CEO | reflections/ | end of cycle | learnings written |
### 🔀 Fan-out points (parallelism): {{which steps fan out — ASCII diagram}}
> **Two reflection loops:** Loop A = per-cycle → reflections/ (permanent). Loop B =
> cross-session: scheduled reports land in reports/; the startup ritual scans, adopts/
> discards, archives. Unfinished work rides in HANDOFF.md.

## 🧱 Architecture (four layers: CEO → Sub-agent SP → Skill → MCP/CLI)
> **Propagation-consistency rule:** before any leaf change (CLI/skill/SP/template) is
> done, confirm each upper layer's contract still holds. Machine backstop = `{{core}} doctor`.
### Skill index — Local (built here) | Global (reused, call directly)
### Sub-agent registry (.claude/agents/<name>.md)
| subagent_type | role | step | mainly uses |
|---|---|---|---|
| `{{name}}` | {{role}} | {{①}} | {{X, Y}} |
| `dev-maintainer` | all code/SP/skill changes | as needed | — |

## 📊 Lifeline memory (State ⑥): {{DB name}}
{{one line per store: what it is + key fields + when read/written. Gitignored.}}

## 🏷️ Brand → documentation/configuration.json (read before any work)
## 📂 File / directory map  ({{ASCII tree, mark gitignored + to-be-built}})
## 📝 Maintenance: session startup ritual (read HANDOFF.md, scan reports/, check the DB) ·
   handoff on loose ends · reload after adding an agent/skill · CHANGELOG per change ·
   doctor is the machine backstop · update this file when structure changes.
```

### 5b · Brand single source of truth (`configuration.json`)
```json
{
  "project": { "name": "", "codename": "", "type": "", "created": "", "phase": "" },
  "brand": {
    "display_name": "",
    "author_attribution": { "default": "", "by_language": {} },
    "slogan": "",
    "logo_assets": "documentation/assets/",
    "design_language": {
      "primary_colors": [], "fonts": [], "tone_keywords": [],
      "visual_style": "", "do": [], "dont": []
    },
    "eeat": { "trust_anchor_url": "", "identity": "" }
  },
  "commercial": { "payment_entity": "", "pricing": {}, "monetization": "" },
  "languages": [],
  "channels_or_imprints": [],
  "notes": "ALL concrete values referenced in more than one place (pricing/colors/attribution) live ONLY here; CLAUDE.md references them by pointer."
}
```

### 5c · Loop blueprint (design one pipeline step as a loop)
```markdown
# Loop: {{LOOP_NAME}}   — purpose: {{what it keeps producing}}   — one iteration = {{one item? batch? session?}}
① Trigger          — first: {{kickoff}} · recurring: {{schedule|event|completion|self-rearm|queue}} · overlap guard: {{…}}
② Context Injection — brief: {{self-contained per-iteration instruction}} · state slice: {{rows from ⑥, summarized}} · kept OUT: {{anti-bloat}}
③ Agent Role        — `{{role}}` — {{single responsibility}} · orchestrator|worker · concurrent siblings: {{…}}
④ Tools             — {{tool}} ({{frozen CLI|MCP|live}}, dry-run: {{how to test safely}}) · narrow: {{tools NOT to reach for}}
⑤ Verification      — gate: {{cheapest deterministic check, exits non-zero}} · inspects the ARTIFACT not the claim · pass = {{bar}}
⑥ State             — durable: {{DB/ledger + key fields}} · reflections: {{where learnings go}} · read-back: {{how ① pulls it in}}
⑦ Retry             — strategy: {{re-attempt+backoff|repair-and-retry|degrade|fix-the-cause}} · max attempts: {{finite}} · changes each try: {{…}}
⑧ Escalation        — stop & ask only for: ① {{missing credential}} ② {{subjective judgment}} ③ {{retry exhausted / irreversible}} · channel: {{…}}
# Closes when: pass → State, fail → Retry → (bounded) → Escalation; context bounded; something compounds in State.
```

> Want the full, commented versions? See [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md),
> [`../templates/verification-checklist.md`](../templates/verification-checklist.md) (gate ⑤),
> [`../templates/memory-state-template.md`](../templates/memory-state-template.md) (state ⑥),
> and the constitution/config templates beside the interview skill. They are richer —
> but the blocks above are enough to scaffold a project from this file alone.

---

## Part 6 · Closing self-check (must pass before delivery)

- [ ] Every pipeline step is designed as a **loop** — at least its Trigger ①,
      Verification ⑤, and Escalation ⑧ are settled (blueprint §5c).
- [ ] **Verification is real** — each step's gate inspects the artifact, not the agent's
      self-report; "done" is unfakeable.
- [ ] **State compounds** — the project has a lifeline store (⑥) and a reflection path
      that writes learnings back so iteration N+1 beats N.
- [ ] **Context stays bounded** — `CLAUDE.md` holds only principles/spine/pointers;
      concrete detail sank into `documentation/` (loaded on demand).
- [ ] The pipeline's **last step = reflection & self-iterate** (the outer loop).
- [ ] Constitution has **Rules** *and* a parallel **Don'ts** (Forbidden vs Discouraged).
- [ ] Each sub-agent names the skills it mainly uses; concurrency captured at fan-out points.
- [ ] Each local skill states which MCP/CLI it is built from; global reuse listed.
- [ ] Concrete brand values live only in `configuration.json`; the constitution is pointers.
- [ ] A `doctor` / consistency-lint command is recommended (Verification ⑤ backstop).
- [ ] Standard shape emitted; the file map matches what was generated; every `{{…}}` replaced.

After delivery: output a **manifest** (files created + one line each), restate the key
decisions, tell the user how to **reload** if the runtime hot-loads agents/skills, and
point to the next step — the project CEO dispatches the maintainer agent to build the
registered local skills / sub-agents / CLI, one loop at a time.

---

## Going deeper (enrichment — optional, not required)

- The full curriculum: [`01-loop-thinking.md`](./01-loop-thinking.md) ·
  [`02-loop-anatomy.md`](./02-loop-anatomy.md) ·
  [`03-verification-and-state.md`](./03-verification-and-state.md) ·
  [`04-retry-and-escalation.md`](./04-retry-and-escalation.md) ·
  [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md) ·
  [`06-agent-loop-patterns.md`](./06-agent-loop-patterns.md).
- Worked loop designs: [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md) ·
  [`../examples/content-production-loop.md`](../examples/content-production-loop.md) ·
  [`../examples/customer-acquisition-loop.md`](../examples/customer-acquisition-loop.md).
- The interview-driven sibling skill (deeper intake, modular templates):
  [`../skills/workflow-design-bible/SKILL.md`](../skills/workflow-design-bible/SKILL.md).
- Project overview: [`../README.md`](../README.md).
