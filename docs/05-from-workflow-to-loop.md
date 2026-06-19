# 05 · From Workflow to Loop — the eight philosophies *are* the eight loop parts

> **The bridge.** This Bible began life as a *constitution generator* for autonomous,
> agent-run projects, built on eight design philosophies — the CEO-orchestrated
> architecture. The upgrade reframes all of it: **design loops, not just prompts.**
> This document is the seam. Its claim is strong and exact: the eight CEO philosophies
> and the eight loop parts are **the same idea seen from two angles.** The
> philosophies tell you how to *organize an autonomous project*; the loop parts tell
> you how to *design one repeating unit of it.* Loop thinking is the lens that fuses
> the two into one.

---

## Thesis: same machine, two blueprints

The original eight philosophies (they live in
[`../skills/workflow-design-bible/SKILL.md`](../skills/workflow-design-bible/SKILL.md) §A)
were written as *architecture rules* — how a project orchestrates itself. The eight
loop parts ([`02-loop-anatomy.md`](./02-loop-anatomy.md)) were written as *iteration
mechanics* — how one cycle runs and repeats. They were never two systems. The
architecture is **what you get when you stack loops**; the loop is **what one row of
the architecture does when it runs.** Read the philosophies and you are reading a
description of a loop with the labels filed off. This doc puts the labels back on.

## The full mapping

Every loop part is a philosophy (or a mechanism inside one) wearing different clothes:

| Loop part | Appears in the CEO architecture as | Why they are the same thing |
|---|---|---|
| ① **Trigger** | startup ritual · scheduled report jobs · pipeline step hand-offs | Both ask *what wakes this unit of work* — a session ritual, a cron'd report, or a prior step's output landing is exactly a loop's recurring trigger. |
| ② **Context Injection** | single-source-of-truth + pointers · "load on demand, not resident" (Phil. 4) | The constitution stays lean by pointing instead of pasting; that *is* the rule "inject the minimum, keep the rest a pointer." Anti-bloat by another name. |
| ③ **Agent Role** | "main agent is a CEO, not a worker" (Phil. 1) · "everything is a sub-agent" (Phil. 2) | The CEO is the orchestrator role; every fixed step is a worker role. Role design and "who runs this iteration" are one question. |
| ④ **Tools** | four-layer stack: role → skill → MCP/CLI (Phil. 3) · global vs local capability (Phil. 5) | "What can this iteration touch" is answered by the layered tool stack — the LLM decides, deterministic code executes, reused globally or built locally. |
| ⑤ **Verification** | "the CEO verifies before reporting" · the `doctor` consistency-lint (Phil. 7) | Both demand a gate the agent cannot fake: read the artifact / curl it / query the DB, and run `doctor` to make claims == reality. |
| ⑥ **State** | lifeline DB · `reflections/` · `HANDOFF.md` (Phil. 6 & 8) | The standard shape's durable stores *are* the loop's memory layers: ledger, reflections, handoff — what survives the window. |
| ⑦ **Retry** | "fix the code, not the artifact" · dry-run before live (Phil. 3 & 7) | When a failure recurs, patch the tool, not the output — that is the loop's repair-and-retry / fix-the-cause discipline. Dry-run is bounded retry's safe rehearsal. |
| ⑧ **Escalation** | "two reasons to stop and ask" standing authorization | The escalation contract — solve it yourself; stop only for a missing private credential or a subjective/business judgment — is identical in both vocabularies. |

Eight rows, one machine. If you have internalized either column, you already half-know
the other. Notice the shape of the correspondence: the *philosophies* are stated as
**project-wide invariants** ("the constitution never bloats," "the CEO verifies before
reporting"), while the *loop parts* are stated as **per-iteration decisions** ("inject
the minimum this run," "what gate must this output pass"). An invariant is just a
decision you have committed to make the same way every iteration — which is exactly why
the same eight show up at both altitudes. The philosophy is the policy; the loop part is
the policy enacted, once, on this turn.

## The key insight: reflection is the OUTER loop

> **Reflection (Philosophy 6) is not a step at the end of a workflow — it is the
> *outer loop* that rewrites the inner loop.** This is the single idea that lets
> "design the loop" subsume "design the workflow," and it deserves its own section.

The original philosophy says the pipeline's spine **ends** with *Reflection &
Self-Iterate*, and that "no self-iteration review = the loop did not close." Read as a
*workflow*, that sounds like a final chore — the last box before you go home. Read as a
*loop*, it is something far more powerful: it is a **second loop wrapped around the
first**, whose product is not content/code/leads but **an improved version of the inner
loop itself.**

```
  OUTER LOOP  (reflection / self-iteration — Loop B, cross-session)
  ┌──────────────────────────────────────────────────────────────┐
  │  ① review last cycle's flaws  ② attribute root cause          │
  │            │                                                  │
  │            ▼                                                  │
  │   ┌─────────────────  INNER LOOP  ─────────────────┐          │
  │   │ ① Trigger → ② Context → ③ Role → ④ Tools →     │  edits   │
  │   │ ⑤ Verify → ⑥ State → ⑦ Retry → ⑧ Escalate     │ ◀──────┐ │
  │   └─────────────────────────────────────────────────┘        │ │
  │            │ produces work                                   │ │
  │            ▼                                                 │ │
  │  ③ dispatch maintainer → patch a tool/template/prompt ──────┘ │
  └──────────────────────────────────────────────────────────────┘
```

The inner loop **produces** (a video, a PR, an outreach message). The outer loop
**consumes the inner loop's track record and edits the inner loop** — it freezes an
on-the-fly step into a CLI subcommand, tightens a verification gate, rewrites a
sub-agent's system prompt, adds a row to the lifeline schema. The inner loop changes
what it *makes*; the outer loop changes what the inner loop *is*. This is the literal
meaning of **the loop that improves the loop**, and it is why Philosophy 6 names *two*
loops — Loop A (per-cycle reflection → `reflections/`) and Loop B (cross-session review
→ `reports/` → startup adoption). The outer loop is not optional polish. A loop without
a reflection loop wrapped around it cannot compound; it can only repeat.

## A pipeline spine is a list of loops

The constitution's **pipeline spine** — the step-by-step table in `CLAUDE.md` — looks
like a workflow: topic → produce → verify → publish → reflect. Loop thinking reads it
differently: **the spine is a chain of completion-triggered loops.** Each row is one
loop whose ① Trigger is the previous row's output, and whose ⑥ State hand-off becomes
the next row's ② Context. The arrows between steps *are* completion triggers.

That means every spine row is a loop you can design with the
[`loop-blueprint.md`](../templates/loop-blueprint.md) — not a paragraph of prose, but
eight answered parts. To **register a loop as a spine row**, give the row exactly what a
blueprint produces:

| Spine column | Comes from the loop's… |
|---|---|
| Step name | Loop identity (§0) |
| What it does / output | ① Trigger + ③ Role's one responsibility |
| Lead role | ③ Agent Role |
| Hand-off / "done" bar | ⑤ Verification + ⑥ State commit |
| Parallel? (fan-out) | ③ concurrent siblings |

So a spine row is a *one-line summary of a loop*, and the full loop design lives in a
pointer (a blueprint file, a sub-agent SP) — which is just Philosophy 4 again: the
constitution holds the pointer, the detail sinks down. And because every row ends by
committing to State (⑥) and every project spine ends in reflection (Phil. 6), the spine
is not a straight line but a **closed chain**: the last row's reflection feeds the outer
loop, which edits the rows. A workflow is read top-to-bottom and ends; a spine of loops
runs, closes, and re-arms.

## Migration recipe: re-express an existing workflow as loops

Have a pipeline already — a sequence of steps you run by hand or in a brittle script?
Convert it loop by loop. For **each step**, answer all eight parts:

1. **Trigger** — what *currently* makes this step start? A human poke? The prior step
   finishing? Make it explicit and recurring. If the honest answer is "I run it
   myself," that is the gap: you are the trigger, so there is no loop yet.
2. **Context** — what does this step actually need to know? Write the *minimum* brief +
   the State slice it reads. Anything else becomes a pointer, loaded on demand.
3. **Role** — name the single role that owns this step and its one responsibility. If
   the step "does two things," split it into two loops.
4. **Tools** — list what it touches. Mark each as frozen (CLI/MCP) or decided live, and
   give each a dry-run mode. Mechanical steps still done "by hand in prose" are
   freeze-candidates.
5. **Verification** — define the gate: the cheapest deterministic check that inspects
   the *artifact*, not the agent's claim. This is the part workflows almost always lack.
6. **State** — decide what this step commits so the next iteration is smarter: a ledger
   row, a reflection, a handoff line. Name durable vs. scratch.
7. **Retry** — pick a bounded attempt count and a repair strategy. When the same failure
   recurs, the fix is in the tool, not the artifact.
8. **Escalation** — write the *few, sharp* reasons this step stops to ask a human, and
   the standing authorization for everything else.

Then run the **loop closure check** at the bottom of the blueprint, and register the
finished loop as a spine row.

> **The three gaps people always discover.** Doing this honestly, a workflow almost
> always reveals: **(1) no verification gate** — "done" meant "the script didn't
> crash," never "the output is right"; **(2) no state** — each step started from zero
> and re-decided settled things; **(3) unbounded context** — the step inhaled
> everything upstream instead of a minimal brief. Those three holes are why the
> workflow needed babysitting. Closing them is the whole point of the migration.

## How this relates to the constitution-generator skill

The two halves of this repo divide labor cleanly:

- **The SKILL scaffolds the skeleton.**
  [`../skills/workflow-design-bible/SKILL.md`](../skills/workflow-design-bible/SKILL.md)
  runs the intake interview and generates the constitution: a lean `CLAUDE.md`, the
  `documentation/` single-source docs, the empty `.claude/agents` & `.claude/skills`
  registries, and the standard shape. It registers *which* loops should exist — the
  pipeline spine — but writes no loop internals.
- **Loop thinking fills the spine.** Every row the SKILL registered is a loop you now
  design with this Bible: the blueprint for the eight parts, the verification checklist
  for the gate, the memory-state template for ⑥, the skill template to freeze ④. The
  SKILL gives you the building; loop thinking furnishes the rooms.

Generate the constitution with the SKILL; then return here, take each spine row, and
make it a real loop.

## Next

- The two parts where loops live or die → [`03-verification-and-state.md`](./03-verification-and-state.md)
- Make failure safe → [`04-retry-and-escalation.md`](./04-retry-and-escalation.md)
- Steal a ready-made loop shape → [`06-agent-loop-patterns.md`](./06-agent-loop-patterns.md)
- Re-read where it all started → [`01-loop-thinking.md`](./01-loop-thinking.md)
- Design a spine row now → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- See a whole project as nested loops → [`../examples/content-production-loop.md`](../examples/content-production-loop.md)
