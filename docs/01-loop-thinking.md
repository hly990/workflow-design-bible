# 01 · Loop Thinking — design the loop, not just the prompt

> **The thesis of the whole Bible.** A prompt produces *one* output. A loop produces
> a *system that keeps producing* outputs — and gets better at it. The moment you
> want an agent to *run something* instead of *answer something*, you have stopped
> writing prompts and started designing loops. Most people never make that switch;
> they keep sharpening a single prompt and wonder why the agent can't sustain a
> project. This document is the switch.

---

## The paradigm shift in one table

| | **Prompt thinking** | **Loop thinking** |
|---|---|---|
| Unit of design | One instruction → one answer | One iteration that repeats and compounds |
| Time horizon | A single turn | Hours, days, sessions, forever |
| What you optimize | Wording, examples, tone | Trigger, verification, state, retry, escalation |
| Failure mode | "The output was wrong" | "The system stalled / drifted / never closed the loop" |
| Memory | Whatever fits in the window | An explicit **State** store that outlives the window |
| Quality control | You eyeball the result | A **Verification** gate the agent cannot skip |
| When it's done | When the answer is good | When the **loop closes** and is safe to run again |
| Who's in charge | You, every turn | The loop runs itself; you set direction and sign off |

A prompt is a sentence. A loop is a **machine** that you design once and let run.
You can write the world's best prompt and still have nothing that runs itself —
because a self-running system is not a better sentence, it is a different artifact.

## Why a great prompt is not enough

Hand an agent a brilliant prompt and ask it to run a content channel / a coding
project / an acquisition funnel for a week. Watch what breaks:

- **It re-decides settled things.** With no **State**, every session re-litigates
  the same naming, the same strategy, the same "didn't we already try this?"
- **It drifts.** With no **Verification**, "done" means "the agent said done." Quality
  sags one acceptable-looking step at a time until the output is slop.
- **It bloats its own context.** With no **Context Injection** discipline, the window
  fills with history until reasoning degrades and cost explodes.
- **It stalls on the first failure.** With no **Retry**, a transient error ends the
  run. With no **Escalation**, a real blocker either halts forever or — worse — the
  agent fabricates its way past it.
- **It never improves.** With no reflection folded back into **State**, the same
  mistake recurs on iteration 2, 20, and 200.

None of these are prompt problems. You cannot word your way out of them. They are
**loop-architecture** problems, and they are solved by designing the eight parts of
a loop — see [`02-loop-anatomy.md`](./02-loop-anatomy.md).

## What a loop actually is

A loop is one **iteration** wrapped in the machinery that lets it repeat safely:

```
        ┌──────────────────────────────────────────────────────────┐
        │                                                          │
   ① TRIGGER ──▶ ② CONTEXT INJECTION ──▶ ③ AGENT ROLE ──▶ ④ TOOLS ──┤
        ▲                                                    │      │
        │                                                    ▼      │
   ⑧ ESCALATION ◀── ⑦ RETRY ◀──[fail]── ⑤ VERIFICATION ──[pass]──▶ ⑥ STATE
        │                                                           │
        └──────────────── (human sign-off / new direction) ────────┘
```

Read it as a sentence: **something triggers an iteration; the right context is
injected into the right role with the right tools; the role acts; the result is
verified; on pass it commits to state and the loop is ready to run again; on fail
it retries a bounded number of times, then escalates to a human.** Eight parts.
Miss one and the loop has a hole that production will find.

> **The loop is not a `while True:`.** Designing a loop is not writing a literal
> `while` statement. It is deciding, for *one* iteration, what wakes it, what it
> knows, who it is, what it can touch, how you know it worked, what it remembers,
> what happens when it fails, and when it stops to ask you. The repetition is the
> easy part; the eight design decisions are the work.

## Loop thinking is fractal

The same eight parts describe a loop at every altitude — this is what makes the
model worth learning once:

- **Micro loop** — a single sub-agent rendering one scene, verifying the file, retrying
  on a bad frame. Seconds.
- **Pipeline loop** — topic → produce → verify → publish → reflect, once per piece of
  content. Minutes to hours.
- **Project loop** — the whole channel/product running session after session, carrying
  a `HANDOFF` and a lifeline DB between runs. Days to forever.

A project is a loop whose iterations are pipelines; a pipeline is a loop whose steps
are micro-loops. You design the same eight parts at each level, and they nest. The
[patterns catalog](./06-agent-loop-patterns.md) shows the recurring shapes.

## The CEO already thinks in loops

If you know the original eight design philosophies (the CEO-orchestrated
architecture), you already have most of this — loop thinking is the **lens** that
makes them one idea instead of eight. The mapping is exact:

| Loop part | Shows up in the CEO architecture as |
|---|---|
| Trigger | session startup ritual · scheduled report jobs · pipeline step hand-offs |
| Context Injection | single-source-of-truth pointers · "load on demand, not resident" |
| Agent Role | the CEO + role-clear sub-agents (everything is a sub-agent) |
| Tools | the four-layer stack: skill → MCP / CLI |
| Verification | "the CEO verifies before reporting" · the `doctor` consistency check |
| State | the lifeline DB · `reflections/` · `HANDOFF.md` |
| Retry | "fix the code, not the artifact" · dry-run before live |
| Escalation | the two-reasons-to-stop standing authorization |
| (the engine) | **reflection is always the last step** — the loop that improves the loop |

The bridge is spelled out in [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md).
The punchline of that doc: **reflection is not a step at the end of a workflow; it
is the outer loop that rewrites the inner loop.** That is why "design the loop"
subsumes "design the workflow."

## How to use this Bible

1. **Learn the anatomy** — [`02-loop-anatomy.md`](./02-loop-anatomy.md). The eight parts, each with its own design questions and failure modes.
2. **Master the two hard parts** — [`03-verification-and-state.md`](./03-verification-and-state.md). Verification and State are where loops live or die.
3. **Make failure safe** — [`04-retry-and-escalation.md`](./04-retry-and-escalation.md). What happens when an iteration fails, and when a human gets pulled in.
4. **Bridge from workflows** — [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md). Map the CEO philosophies onto loops; migrate an existing workflow.
5. **Steal a pattern** — [`06-agent-loop-patterns.md`](./06-agent-loop-patterns.md). A catalog of proven loop shapes.

In a hurry to scaffold a whole project? The capstone skill
[`00-scaffold-a-self-running-project.md`](./00-scaffold-a-self-running-project.md) is a
self-contained, paste-and-run procedure that applies everything below in one file.

Then **design your loop** with the [`loop-blueprint.md`](../templates/loop-blueprint.md)
template, gate it with the [`verification-checklist.md`](../templates/verification-checklist.md),
give it memory with the [`memory-state-template.md`](../templates/memory-state-template.md),
and turn any frozen step into a capability with the [`skill-template.md`](../templates/skill-template.md).
Three worked end-to-end designs live in [`examples/`](../examples/coding-agent-loop.md).

> **One rule to carry out of this page:** when a request starts with "keep…",
> "every…", "run…", "maintain…", "grow…", or "until done", stop reaching for a
> better prompt. Reach for the eight loop parts. You are not answering — you are
> building a machine that answers, again and again, without you.
