# 02 · Loop Anatomy — the eight parts of every agent loop

> **The canonical reference.** Every agent loop, at every altitude, is built from
> eight parts. This document defines them, in order, with the design questions you
> must answer and the failure mode you get when you skip each one. The
> [`loop-blueprint.md`](../templates/loop-blueprint.md) template is just these eight
> parts with blanks to fill. Memorize the order — it *is* the iteration:

```
① TRIGGER → ② CONTEXT INJECTION → ③ AGENT ROLE → ④ TOOLS
                                                      │
   ⑥ STATE ◀──[pass]── ⑤ VERIFICATION ──[fail]──▶ ⑦ RETRY ──▶ ⑧ ESCALATION
```

Five parts set the iteration up (① → ⑤). Three parts decide what happens to the
result (⑥ commit, ⑦ retry, ⑧ escalate). A loop with all eight closes; a loop missing
one has a hole.

---

## ① Trigger — what starts an iteration

The trigger is the *clock* of the loop. It answers: **what wakes this iteration, and
how does the next one begin?** A loop with no defined trigger is a script you have
to babysit.

**Kinds of trigger**
- **Scheduled** — cron / interval ("every morning", "every 10 min"). Best for heartbeat work.
- **Event** — a webhook, a file landing, a queue message, an inbound DM.
- **Completion** — the previous step's output is this step's trigger (pipeline hand-off).
- **Human** — the owner says "go" / sets new direction (often the *first* trigger only).
- **Self / continuation** — the loop re-arms itself: it schedules its own next wake-up, or processes the next item off a work queue it maintains.

**Design questions**
- What is the *first* trigger vs. the *recurring* trigger? (They are usually different.)
- Is one iteration one *item* or one *batch*? Where does the work-list come from?
- What stops the loop from re-firing while a previous iteration is still running?
- When the loop has nothing to do, does it idle cheaply or spin?

**Failure mode if skipped:** the "loop" only runs when you personally poke it — you
*are* the trigger, which means there is no loop, only a prompt you keep re-sending.

> **In the CEO architecture:** the session **startup ritual** is a trigger; scheduled
> **report jobs** are triggers; each pipeline step **hand-off** is a completion trigger.

## ② Context Injection — what the iteration knows

Context injection answers: **what information enters the agent's window this
iteration, and what is deliberately kept out?** This is the most under-designed part
and the one that silently destroys long-running loops. The window is a budget, not a
warehouse.

**What gets injected**
- The **task brief** for *this* iteration (self-contained — see Agent Role).
- The relevant slice of **State** (⑥): not the whole DB, the rows that matter now.
- **Pointers** to single sources of truth, loaded *on demand* — not pasted resident.
- The **inputs**: the item, the upstream artifact, the user's latest direction.

**Design questions**
- What is the *minimum* context for a correct result? Inject that, nothing more.
- What is a **pointer** (load if needed) vs. **resident** (always present)? Default to pointer.
- How does State get summarized down to a window-sized slice each iteration?
- Does context *grow unboundedly* across iterations? If yes, you have a leak — fix it now.

**Failure mode if skipped:** context bloat. The window fills with stale history,
reasoning degrades, cost climbs every turn, and the agent starts contradicting its
own earlier iterations because it can no longer see them clearly.

> **In the CEO architecture:** "single source of truth + pointers; the constitution
> never bloats" *is* context-injection discipline. `CLAUDE.md` stays lean precisely
> because it is resident on every turn; everything else is a pointer loaded on demand.

## ③ Agent Role — who runs the iteration

The role answers: **whose system prompt is executing this iteration?** A loop runs as
*someone* — a defined role with a clear job, not a generic assistant. One iteration,
one role; the orchestrator is itself a role (the CEO) whose job is to dispatch the others.

**Design questions**
- What is this role's *one* responsibility? (If it has two, it is two roles.)
- Is this the **orchestrator** (decides, delegates, verifies, talks to the human) or a **worker** (takes a self-contained brief, does one thing, returns)?
- Does the brief carry *full* context so the worker needs no back-and-forth?
- Which roles can run **concurrently**? (Independent roles fan out; that is free parallelism.)
- Are you about to spawn a new role where a shared one would do? (Resist proliferation.)

**Failure mode if skipped:** a do-everything agent with no role boundary — it holds
the whole project in one window, cannot parallelize, and cannot be reasoned about,
verified, or improved part by part.

> **In the CEO architecture:** "the main agent is a CEO, not a worker" and "everything
> is a sub-agent" are role design. The CEO is the orchestrator role; every fixed step
> is a worker role.

## ④ Tools — what the iteration can do

Tools answer: **what actions can this role take to affect the world?** A role with no
tools can only talk. Tools are tiered, and the tiering is the whole point: the LLM
*decides*, deterministic code *executes*.

**The tool stack (top points down, never up)**
```
role  ──uses──▶  skill (SKILL.md: how a capability is used)
                   ──built from──▶  MCP servers + CLI commands (executed, never in context)
```

**Design questions**
- Which actions are **frozen into deterministic CLI/MCP** (zero-token, repeatable) vs. decided live?
- For each tool, what is its **dry-run / test mode**? (No real publish/deploy/charge as a test.)
- Does the role's brief **narrow** which tools to use? (Seeing a tool ≠ should-use it.)
- Is a recurring mechanical step still being "decided on the fly"? Freeze it into a tool.

**Failure mode if skipped:** the agent re-derives the same mechanical action every
iteration in fragile prose, burning tokens and producing inconsistent results, instead
of calling one deterministic command that always behaves the same.

> **In the CEO architecture:** this is the four-layer stack (skill → MCP/CLI) and the
> rule "freeze recurring mechanical steps into CLI subcommands." Turn a frozen step
> into a capability with [`skill-template.md`](../templates/skill-template.md).

## ⑤ Verification — how you know it actually worked

Verification answers: **what gate must the output pass before it counts as done?**
This is the part that separates a loop that compounds quality from a loop that
compounds slop. **"The agent said it's done" is not verification.** A real gate is
something the agent *cannot talk its way past*.

**Kinds of gate (cheapest first)**
- **Deterministic check** — a command exits non-zero on failure (tests, linter, schema validator, `doctor`, link checker). The gold standard: machine-judged, unfakeable.
- **Self-check against a rubric** — the role re-reads its output against an explicit checklist before claiming done.
- **Independent verifier** — a *different* role (with no stake in the output) judges it. Adversarial by design.
- **Human sign-off** — reserved for subjective/irreversible calls; the most expensive gate, used sparingly.

**Design questions**
- What is the *cheapest deterministic* check that would catch the most common failure? Build that first.
- Does the verifier see the **artifact itself** (read the file, curl the URL, query the DB) or just the worker's claim?
- Who verifies — the actor (weak), a separate role (strong), or a command (strongest)?
- On pass, what commits to State? On fail, what triggers Retry? (⑤ is the fork in the loop.)

**Failure mode if skipped:** silent drift. Every iteration looks fine and is slightly
worse, because nothing ever says *no*. By the time a human notices, the State is full
of accepted garbage. **Verification is the single highest-leverage part of the loop** —
see [`03-verification-and-state.md`](./03-verification-and-state.md).

> **In the CEO architecture:** "the CEO verifies before reporting" (read it / curl it /
> query the DB — never trust a sub-agent's 'done') and the `doctor` consistency-lint
> are verification gates.

## ⑥ State — what survives between iterations

State answers: **what does the loop remember after the window is gone?** The window is
erased every iteration; State is what makes iteration N+1 smarter than iteration N
instead of identical to it. No State, no compounding — just an amnesiac repeating itself.

**Layers of state**
- **Ledger / DB** — the durable record of what exists and what's been done (the lifeline). Queryable, the source of truth for "what's the situation."
- **Reflections** — what was learned, written permanently (never in a build dir that gets cleaned). The compounding layer.
- **Handoff** — the concise "where we left off / what's next" note that bridges sessions.
- **Reports** — async analytics the next iteration consumes at startup, then archives.

**Design questions**
- What is the *minimum* that must persist for iteration N+1 to not repeat iteration N's work or mistakes?
- What is **durable** (survives forever) vs. **scratch** (safe to delete each run)?
- How does State get *read back in* at the next Trigger and *summarized* for Context Injection (②)?
- Where do learnings go so they actually change future behavior, not just get logged?

**Failure mode if skipped:** Groundhog Day. Every iteration starts from zero,
re-decides settled questions, re-makes fixed mistakes, and the project never gets
better no matter how many times it runs.

> **In the CEO architecture:** the lifeline **DB**, the `reflections/` folder, and
> `HANDOFF.md` are the State layers. Design yours with
> [`memory-state-template.md`](../templates/memory-state-template.md).

## ⑦ Retry — what happens when an iteration fails

Retry answers: **when Verification says no, what does the loop do before giving up?**
Most failures are transient or fixable; a loop with no retry strategy throws away the
whole run over a hiccup. But retry must be *bounded* — an unbounded retry is just a
new way to hang.

**Retry strategies**
- **Re-attempt** — try again, often with backoff (transient API/network errors).
- **Repair-and-retry** — feed the verifier's complaint back in and fix the specific defect.
- **Fix the cause, not the artifact** — if the *same* failure recurs, the bug is in the code/template/skill, not this output. Patch the tool, not the result. Never hand-edit one output to fake a pass.
- **Degrade** — fall back to a simpler-but-valid result rather than nothing.

**Design questions**
- How many attempts before escalating? (Bounded, always. Pick the number.)
- Does retry change *something* each attempt, or just repeat the identical failing call?
- What is the backoff between attempts?
- When does a recurring failure flip from "retry this artifact" to "fix the tool"?

**Failure mode if skipped:** either a single transient error kills the whole run, or —
with naive infinite retry — the loop wedges forever on an unfixable input, silently
burning money.

> **In the CEO architecture:** "fix the code, not the artifact" and "test in dry-run"
> are retry discipline. Detail in [`04-retry-and-escalation.md`](./04-retry-and-escalation.md).

## ⑧ Escalation — when a human gets pulled in

Escalation answers: **when does the loop stop and ask a human, and how?** A truly
autonomous loop is not one that *never* asks — it is one that asks **only** when it
genuinely must, and otherwise solves its own problems. The art is drawing that line
sharply so the loop neither halts on every bump nor fabricates past a real blocker.

**When to escalate (and essentially only then)**
- **Missing credential / authorization** — needs a secret, account, or permission only the human holds.
- **Subjective / business / value judgment** — a direction call the owner must own.
- **Retry budget exhausted** — ⑦ tried its bounded attempts and the gate still fails.
- **Irreversible + high-stakes** — going live, spending real money, anything hard to undo.

**Design questions**
- What are *your* two-or-three legitimate reasons to stop? Write them down; everything else the loop handles itself.
- How does an escalation reach the human (channel), and with what (a crisp ask + context + what you tried)?
- While escalated, does the loop *block*, *park the item and continue others*, or *degrade*?
- What is the standing authorization — the explicit "you may do all of this without asking"?

**Failure mode if skipped:** one of two opposite disasters. Too eager → the loop
pings the human constantly and isn't autonomous. Too reluctant → the loop hits a real
blocker and *invents* its way past it (fake credentials, hallucinated approval, made-up
data), corrupting State.

> **In the CEO architecture:** the standing authorization — "solve problems yourself;
> stop and ask in only two cases: a missing private credential, or a subjective/
> business/value judgment" — is the escalation contract.

---

## The eight at a glance

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

## The two forces that hold a loop together

Two parts deserve to be called out as load-bearing, because every other part leans on
them:

- **Verification (⑤) is the quality force.** It is the only part that can say *no*. It
  is what turns repetition into improvement instead of repetition into decay.
- **State (⑥) is the memory force.** It is the only part that survives the window. It
  is what turns "ran a thousand times" into "learned a thousand times."

A loop with strong Verification and strong State is forgiving of weakness elsewhere —
it catches its own mistakes and remembers them. A loop weak in either will rot no
matter how elegant the other six parts are. That is why they get their own document:
[`03-verification-and-state.md`](./03-verification-and-state.md).

## Next

- Go deep on the two hard parts → [`03-verification-and-state.md`](./03-verification-and-state.md)
- Make failure safe → [`04-retry-and-escalation.md`](./04-retry-and-escalation.md)
- See all eight mapped onto the CEO philosophies → [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md)
- Steal a ready-made shape → [`06-agent-loop-patterns.md`](./06-agent-loop-patterns.md)
- Design yours now → [`loop-blueprint.md`](../templates/loop-blueprint.md)
