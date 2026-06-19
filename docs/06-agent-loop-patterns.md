# 06 · Agent Loop Patterns — proven shapes you instantiate with the blueprint

> **A pattern is a pre-decided loop.** The eight parts of
> [`02-loop-anatomy.md`](./02-loop-anatomy.md) give you the *grammar*; a pattern gives
> you a *sentence* that has already been said a thousand times and works. You don't
> redesign Trigger-through-Escalation from scratch for every loop — you recognize the
> shape, grab the pattern, and fill the [`loop-blueprint.md`](../templates/loop-blueprint.md)
> with its defaults already leaning the right way. Most real systems are two or three
> of these patterns nested.

Each pattern below names: **when to use**, an **8-part sketch** that calls out only the
parts that carry weight for that shape (assume the others take their anatomy defaults), a
tiny **flow**, and a **gotcha** you will hit if you skip the load-bearing part. They are
ordered foundation-first: pattern 1 is the base loop every other one extends.

---

## 1 · Produce → Verify → Retry (the base loop)

**When to use:** always. This is the irreducible loop; every other pattern is this one
with a specialized Trigger, State, or verifier. If you can't draw this, you don't have a
loop yet.

| Part | What it does here |
|---|---|
| ① Trigger | one unit of work is ready (item, brief, request) |
| ⑤ **Verification** | the deterministic gate — exits non-zero on a bad result |
| ⑥ State | on pass, commit the artifact + a one-line "what was learned" |
| ⑦ **Retry** | repair-and-retry, bounded; feed the verifier's complaint back in |
| ⑧ Escalation | retry budget exhausted → hand the human a crisp "tried N, still fails" |

```
produce ─▶ verify ──pass──▶ commit to State ─▶ (next trigger)
              │
            fail ─▶ retry (bounded) ─▶ exhausted ─▶ escalate
```

> **Gotcha:** if Verification only reads the agent's *claim* ("looks good to me") instead
> of the **artifact** (read the file, curl the URL, query the DB), this isn't the base
> loop — it's a prompt with extra steps, and it will compound slop. ⑤ inspects reality.

## 2 · Generate-and-Judge (independent / adversarial verifier)

**When to use:** the output is **subjective** — copy, design, a plan, a summary — so no
single deterministic command can score it, but a *different* role can.

| Part | What it does here |
|---|---|
| ③ Agent Role | a **producer** role and a separate **judge** role — never the same window |
| ⑤ **Verification** | the judge scores against an explicit rubric; or an **N-vote panel** of independent judges and you take the majority/median |
| ⑦ Retry | repair-and-retry using the judge's specific objections as the next brief |
| ⑧ Escalation | judges disagree past a threshold, or rubric ceiling unmet after N → human |

```
producer ─▶ artifact ─▶ judge₁ ┐
                        judge₂ ┼─ vote/score ──pass──▶ State
                        judge₃ ┘        │
                                      fail ─▶ retry w/ objections
```

> **Gotcha:** the judge must have **no stake** in the output and must see the rubric, not
> the producer's reasoning. If the producer judges itself, it grades on a curve every
> time. Adversarial-by-design or it's theater.

## 3 · Queue Worker (drain a work-list)

**When to use:** a backlog of homogeneous items exists and you want to chew through it
one at a time until empty.

| Part | What it does here |
|---|---|
| ① **Trigger** | "next item off the queue" — Trigger *is* the work-list pop |
| ② Context Injection | only **this item** + its State slice; never the whole queue |
| ⑥ **State** | mark item `in-flight → done/failed`; the queue *is* durable state |
| ⑦ Retry | failed item → back to queue with an attempt counter, or to a dead-letter list |

```
┌─▶ pop item ─▶ mark in-flight ─▶ produce/verify ─▶ done ─┐
│                                                          │
└──────────────── queue not empty ◀───────────────────────┘
```

> **Gotcha:** without an **overlap guard** + **idempotency**, two iterations grab the same
> item, or a crash mid-item re-processes it and double-charges/double-posts. Claim the
> item atomically; make the work safe to run twice.

## 4 · Heartbeat / Scheduled Monitor (cron Trigger)

**When to use:** you must *detect and react* to a condition you don't control — a new
mention, a broken deploy, a price change, an inbox — on a clock.

| Part | What it does here |
|---|---|
| ① **Trigger** | cron / interval ("every 10 min"); the loop wakes itself |
| ② Context Injection | the *delta since last run* (a high-water mark from State), not all history |
| ⑤ Verification | did the action actually fire / land? (deterministic post-check) |
| ⑥ State | the watermark / last-seen cursor so the next tick only sees what's new |

```
⏰ tick ─▶ check condition ──nothing──▶ idle cheaply, advance cursor
              │
            something ─▶ act ─▶ verify ─▶ State (new watermark)
```

> **Gotcha:** when there's nothing to do, the loop must **idle cheaply** — no full context
> load, no expensive scan. A heartbeat that does 10K tokens of work to discover "nothing
> changed" every 5 minutes is a money fire, not a monitor.

## 5 · Fan-out / Fan-in (batch concurrency)

**When to use:** many **homogeneous** items can be produced independently and *in parallel*
(60 scenes, 50 product pages, 30 variant images), then must be assembled and checked as a set.

| Part | What it does here |
|---|---|
| ③ **Agent Role** | one worker role, spawned N-wide; a coordinator owns the fan-in |
| ④ Tools | a concurrency cap (e.g. 50 in flight) so you don't exhaust rate limits |
| ⑤ **Verification** | verify **each** item *and* the **assembled set** (count, gaps, consistency) |
| ⑦ Retry | only the failed items re-run, not the whole batch |

```
brief ─┬─▶ worker ─▶ item₁ ─┐
       ├─▶ worker ─▶ item₂ ─┼─▶ fan-in ─▶ verify set ─▶ State
       └─▶ worker ─▶ itemₙ ─┘            (re-run only failures)
```

> **Gotcha:** set-level verification is the part everyone forgets. Every item passes
> individually and the batch is still wrong — 59 of 60 scenes, or two pages with the same
> slug. **Verify the set, not just the members.**

## 6 · Pipeline (completion-triggered chain)

**When to use:** an item must flow through ordered stages — topic → script → render →
publish — where each stage's *output* is the next stage's *input*.

| Part | What it does here |
|---|---|
| ① **Trigger** | each stage's Trigger is the **completion** of the prior stage (hand-off) |
| ② Context Injection | each stage gets a self-contained brief + the upstream artifact only |
| ⑤ Verification | **a gate at every stage boundary** — bad artifacts never flow downstream |
| ⑧ Escalation | a stage that can't pass blocks its item but *not the whole pipeline* |

```
topic ─▶│gate│─▶ script ─▶│gate│─▶ render ─▶│gate│─▶ publish ─▶ State
         (fail → retry/escalate that stage; downstream stays clean)
```

> **Gotcha:** a missing gate between stages lets a defect ride to the end, where it's
> 10× more expensive to catch and may already be published. **Verify at the seam, every
> seam.** Each stage is itself a base loop (pattern 1).

## 7 · Reflect-and-Improve (the outer loop)

**When to use:** always, as the *outer* wrapper — this is the compounding engine. The
inner loop produces; this loop **rewrites the inner loop's tools, templates, and rubrics**
so iteration N+1 is structurally better, not just luckier.

| Part | What it does here |
|---|---|
| ① Trigger | end of a cycle / a recurring review job (completion or scheduled) |
| ② Context Injection | the cycle's failures, retries, and verifier complaints from State |
| ③ **Agent Role** | the **maintainer** role — it edits code/skills/prompts, not artifacts |
| ⑥ **State** | learnings written to `reflections/` (permanent) → fed back as new defaults |
| ⑦ **Retry** | recurring failure ⇒ **fix the cause** (patch the tool), never the artifact |

```
inner loop runs ─▶ reflect on State ─▶ maintainer patches tool/template/rubric
        ▲                                              │
        └──────────── next cycle runs the improved inner loop ◀┘
```

> **Gotcha:** if reflection writes to a log nobody reads, or to a build dir that gets
> cleaned, nothing compounds — you've built a diary, not an engine. The learning must
> **change a default** the inner loop reads next time. See
> [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md): reflection is the outer
> loop that rewrites the inner one.

## 8 · Escalation-Gated / Human-in-the-Loop checkpoint

**When to use:** one step is **irreversible or genuinely subjective** — going live,
spending real money, a brand/value call — and must not proceed on the agent's word alone.

| Part | What it does here |
|---|---|
| ⑤ **Verification** | the gate routes to a **human sign-off** instead of (or after) a machine check |
| ⑦ Retry | revise-and-resubmit on rejection, with the human's reason fed back |
| ⑧ **Escalation** | this *is* an escalation made first-class — a planned stop, not a failure |

```
artifact ─▶ machine pre-check ─▶ HUMAN sign-off ──approve──▶ commit/publish
                                       │
                                    reject ─▶ revise ─▶ resubmit
```

> **Gotcha:** while parked on a human, the loop should **park this item and keep working
> others**, not block the whole system. And the ask must be crisp — "approve A or B,
> here's context, here's what I'd do" — not "what should I do?" A vague checkpoint trains
> the owner to rubber-stamp.

## 9 · Loop-until-Target / Loop-until-Dry

**When to use:** you need to **accumulate to a count** (publish 10 pieces, gather 50 leads)
or **run until exhausted** (process until K consecutive empty rounds), not run forever.

| Part | What it does here |
|---|---|
| ① Trigger | self-re-arm: each iteration schedules the next *if the stop condition is unmet* |
| ⑤ Verification | each produced unit must pass before it counts toward the target |
| ⑥ **State** | the **running tally** / the empty-round counter — the loop's odometer |
| ⑧ **Escalation** | can't reach target after a max-attempts ceiling → stop and report, don't grind |

```
loop: produce ─▶ verify ─▶ tally++ ─┐
        ▲                            │
        └── tally < target AND attempts < ceiling ◀┘   else ─▶ stop + report
```

> **Gotcha:** count **verified** units, not *attempts*, toward the target — or a flaky
> producer "reaches 10" with 3 real pieces and 7 rejects. And always carry a **ceiling**
> (max attempts / max wall-clock) so "until dry" can't become "until bankrupt."

---

## Patterns at a glance

| Pattern | Primary altitude | Part it stresses most |
|---|---|---|
| 1 · Produce → Verify → Retry | micro | ⑤ Verification |
| 2 · Generate-and-Judge | micro / pipeline | ⑤ Verification (independent) |
| 3 · Queue Worker | pipeline | ① Trigger + ⑥ State (idempotency) |
| 4 · Heartbeat / Monitor | project | ① Trigger (cheap idle) |
| 5 · Fan-out / Fan-in | pipeline | ③ Role + ⑤ set-verification |
| 6 · Pipeline | pipeline | ① completion-Trigger + ⑤ seam gates |
| 7 · Reflect-and-Improve | project | ⑥ State + ⑦ fix-the-cause |
| 8 · Escalation-Gated | pipeline / project | ⑧ Escalation (planned) |
| 9 · Loop-until-Target/Dry | project | ⑥ State (tally) + ⑧ ceiling |

> **They nest, they don't compete.** A real project is usually a **Heartbeat** (4) waking
> a **Pipeline** (6) whose stages are **Fan-out** (5) batches of **Produce-Verify-Retry**
> (1) micro-loops, all wrapped in **Reflect-and-Improve** (7), with an **Escalation gate**
> (8) on the publish step. Pick the shapes per altitude; fill one blueprint per loop.

## Next

- Instantiate a chosen pattern → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- Harden the gate the pattern stresses → [`03-verification-and-state.md`](./03-verification-and-state.md) · [`../templates/verification-checklist.md`](../templates/verification-checklist.md)
- Make failure safe → [`04-retry-and-escalation.md`](./04-retry-and-escalation.md)
- See patterns wired end-to-end → [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md) · [`../examples/content-production-loop.md`](../examples/content-production-loop.md) · [`../examples/customer-acquisition-loop.md`](../examples/customer-acquisition-loop.md)
