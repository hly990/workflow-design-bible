# 04 · Retry & Escalation — what happens when an iteration fails

> **Failure is the normal case, not the exception.** A loop that only works when
> every iteration passes on the first try is a demo, not a system. The interesting
> design is what happens *after* Verification (⑤) says **no**. Two parts own that
> moment: **⑦ Retry** decides what the loop tries before giving up, and **⑧ Escalation**
> decides when — and *only* when — it stops to pull in a human. Get this pair right
> and your loop is autonomous; get it wrong and it either hangs forever, pings you
> every five minutes, or — worst of all — *fabricates* its way past a real blocker.

This document is the deep dive on parts ⑦ and ⑧ from
[`02-loop-anatomy.md`](./02-loop-anatomy.md). It assumes you already have a real
Verification gate — if you don't, retry has nothing to react to. Fix that first
in [`03-verification-and-state.md`](./03-verification-and-state.md).

---

## First, classify the failure

You cannot retry well until you know *which kind* of failure you're holding. Reading
the verifier's complaint and sorting it into one of four buckets is the entire skill.
The bucket dictates the response.

| # | Failure class | What it looks like | Correct response |
|---|---|---|---|
| (a) | **Transient** | Network blip, 429/503, timeout, lock contention | **Re-attempt with backoff** — same call, wait, try again |
| (b) | **Deterministic defect (this artifact)** | Lint error, failing assertion, schema mismatch, broken link | **Repair-and-retry** — feed the complaint back, fix *that* defect |
| (c) | **Recurring same-failure** | The identical defect comes back after a repair, every time | **Fix the cause** — the bug is in the tool/template/skill, not the output |
| (d) | **Hard blocker** | Missing secret, irreversible call, a judgment only the owner can make | **Escalate** — ⑧, with a crisp ask |

> **The misclassification tax.** Treat a deterministic defect (b) as transient (a)
> and you re-run the identical failing call N times, burning money to watch the same
> red. Treat a recurring failure (c) as a one-off (b) and you hand-edit output after
> output, faking passes while the real bug ships again next iteration. The class is
> the decision. Name it before you act.

---

## ⑦ Retry — bounded, and *different* each time

Two non-negotiable rules govern every retry strategy:

1. **Bounded.** Always pick a finite `max_attempts`. Unbounded retry is not
   resilience — it is a new way to hang while silently spending. "Keep trying until
   it works" is a wish, not a design.
2. **Something must change each attempt.** An identical re-call against a
   deterministic defect produces an identical failure. Either the *input* changes
   (you fed the complaint back), the *environment* changes (you waited out a
   transient), or the *strategy* changes (you degraded). A retry that changes nothing
   is just the same iteration wearing a hat.

### The four strategies

- **Re-attempt with backoff** *(class a)* — for transient errors. Same call, but wait
  between tries: exponential backoff (1s → 2s → 4s) plus a little jitter so parallel
  workers don't synchronize their retries. 3–4 attempts is plenty; if it's still
  failing, it isn't transient — reclassify.
- **Repair-and-retry** *(class b)* — the workhorse. Inject the verifier's *specific*
  complaint into the next iteration's context and instruct the role to fix exactly
  that defect. `tests::auth_test failed: expected 200 got 401` is a far better retry
  prompt than "try again." The artifact changes because the brief changed.
- **Fix the cause, not the artifact** *(class c)* — see below. The most important rule
  in this document.
- **Degrade gracefully** *(fallback)* — when full success is out of reach, a *simpler
  valid* result beats nothing. Fewer scenes but a publishable video; a draft flagged
  "needs human polish" instead of a hard stop; the cached value instead of the
  freshly computed one. Degrade is a *legitimate retry outcome* — it closes the loop
  with a smaller-but-real artifact and lets the next iteration carry on.

### The failure → retry → escalation flow

```
        ⑤ VERIFICATION
              │ fail
              ▼
        ┌───────────────┐   attempts < max ?  ── no ──▶ ⑧ ESCALATE
        │ classify fail │                                 (budget exhausted)
        └──────┬────────┘   yes
               │
   ┌───────────┼───────────────┬─────────────────┐
 (a)│        (b)│             (c)│               (d)│
   ▼           ▼               ▼                  ▼
re-attempt   repair &      same defect ≥2×?   ⑧ ESCALATE
+ backoff    retry          FIX THE CAUSE      (hard blocker —
   │         (feed back     (patch tool/        no retry burns
   │          the complaint) template/skill)    a real attempt)
   └─────┬───────┴───────────────┘
         ▼
   re-run ⑤ VERIFICATION  ──[pass]──▶ ⑥ STATE
         │
      (loop until pass OR attempts == max)
```

Read it as a sentence: **the gate fails; you classify; transient errors back off and
retry, defects get repaired with the complaint fed back, a repeat defect flips you to
fixing the tool, and a hard blocker escalates immediately. Each repaired attempt
re-enters Verification. When attempts run out, you escalate.**

### Fix the code, not the artifact

This is the rule that separates a loop that *compounds* from one that *rots*. When the
same failure recurs (class c), the defect is no longer in the output — it is in the
thing that *produces* the output. The tool, the template, the prompt, the skill.

> **Never hand-edit one output to fake a pass.** Patching a single artifact so the
> gate goes green is the most seductive anti-pattern in agent work: it feels like
> progress, it ships *this* item, and it guarantees the *next* item fails the same
> way. You have spent effort making the bug invisible. Fix the generator and every
> future iteration passes for free; fix the artifact and you've signed up to do it
> again forever.

And the safety corollary that makes retry survivable at all:

> **Never run a real publish / deploy / charge as a test.** Retries happen *in
> dry-run*. You verify against a staged artifact, a `--dry-run` flag, a sandbox, a
> test DB — not the live channel, not the production deploy, not a real card. The
> live action fires exactly **once**, after the gate is green. A loop that retries
> against production is a loop that double-posts, double-charges, and double-deploys
> on its way to "done." Every tool needs a test mode (see Tools, ④).

---

## ⑧ Escalation — the standing-authorization model

A truly autonomous loop is **not** one that never asks a human. It is one that asks
**only** when it genuinely must, and otherwise solves its own problems. The whole
craft is drawing that line *sharply*. The default posture is a standing
authorization: **you are pre-approved to solve problems yourself — stop only for the
short list below.**

**The few, sharp reasons to stop and ask:**

| # | Reason | Why a human (and not the loop) | Example |
|---|---|---|---|
| ① | **Missing private credential / authorization** | The loop *cannot* obtain a secret only the owner holds | API key absent; OAuth needs the owner's account |
| ② | **Subjective / business / value judgment** | A direction call the owner must *own* | "Which of these two strategies?" "Is this on-brand?" |
| ③ | **Retry budget exhausted** | ⑦ tried its bounded attempts; the gate still fails | The fix isn't landing and you can't diagnose why |
| ④ | **Irreversible + high-stakes** | Hard to undo, expensive to get wrong | First-time production deploy; a large real charge |

Everything else — transient errors, defects you can repair, tools you can patch,
data you can look up, decisions inside the owner's stated direction — **the loop
handles itself.** If you find yourself escalating something outside these four, you
have found a gap in your tools or your standing authorization, not a reason to ask.

### The escalation payload

An escalation is a *handoff to a human*, so it obeys handoff discipline: crisp, and
self-contained. Three parts, always:

```
ASK:    one sentence — the single decision or secret you need.
        "Need the STRIPE_API_KEY for the prod account to publish."
CONTEXT: where the loop is, why this is blocking, what's at stake.
        "Iteration 14, publish step. 13 items live. This one is held."
TRIED:  what you already did so the human isn't re-debugging from zero.
        "Confirmed key absent in env + vault; dry-run publish passes."
```

> **A good escalation can be answered in one reply.** If the human has to ask you a
> clarifying question, your payload was incomplete. The `TRIED` line is what proves
> the loop *exhausted its own autonomy* before reaching out — it is the difference
> between "I'm stuck, help" and "here is the one atom only you can supply."

**Channel** matches stakes and latency: a non-blocking item parks a note in the
`HANDOFF` / a queue; an urgent blocker pings the owner directly (DM, the channel they
watch). Don't page someone for a parkable item; don't bury an irreversible-action
approval in a log nobody reads.

### While escalated: block, park, or degrade

One blocked item should rarely stop the whole loop. Pick per item:

- **Block** — stop this item and wait. Correct only when nothing else can proceed
  without the answer (a missing credential the *whole* loop needs).
- **Park and continue others** — set this item aside, keep processing the work-list.
  The default for a batch loop: one missing asset shouldn't freeze the other forty.
- **Degrade** — ship the simpler valid result now, flag it for human review later.
  Best when *some* output beats *no* output and the gap is non-critical.

---

## The two opposite failure modes

Escalation has two ways to be wrong, and they pull in opposite directions. A healthy
loop lives in the narrow band between them.

| | **Over-eager escalation** | **Too-reluctant escalation** |
|---|---|---|
| Symptom | Pings the human on every bump | Invents its way past a real blocker |
| Feels like | "Being careful" | "Being resourceful" |
| Actually is | Not autonomous — you're back to babysitting | **Corrupting State with fabrication** |
| Tell-tale | Questions the loop could've answered itself | Fake credentials, hallucinated approval, made-up data |
| Fix | Widen standing authorization; trust the gate | Tighten the four reasons; **forbid fabrication** |

> **Anti-fabrication is the cardinal rule.** Of the two, the reluctant failure is far
> more dangerous, because it is *silent*. An over-eager loop annoys you; a reluctant
> loop that hallucinates an API key, fakes an approval, or fills a required field with
> plausible-looking garbage writes that lie into **State (⑥)** — and every future
> iteration now builds on a corrupted foundation. **When the loop lacks a real input
> it cannot obtain, the only correct move is to escalate or stop. Never invent it.**
> A loop that escalates honestly is recoverable; a loop that fabricates is poison.

---

## Two worked mini-examples

**A flaky integration test (class a → resolve without a human).** Iteration runs the
suite; one test fails with `connection reset`. The loop classifies it as transient,
backs off 2s, re-runs *only the failing test* — green. It commits to State and moves
on. No human, no fuss. (Had it failed three times running, the loop would reclassify:
this isn't flakiness, it's a real defect — switch to repair-and-retry.)

**A missing API key (class d → escalate honestly).** Publish step needs
`STRIPE_API_KEY`; it's absent from env and vault. There is *no* retry that conjures a
secret — burning attempts here is pure waste, so the loop escalates **immediately**
without spending the budget. It posts: *ASK:* "Need STRIPE_API_KEY for prod to
publish item #14." *CONTEXT:* "13 items already live; this one parked." *TRIED:*
"Confirmed absent in env + vault; dry-run publish passes, so only the live key is
missing." It **parks** item #14, **continues** items #15+, and waits. What it does
*not* do: fabricate a key, fake the publish, or write "published" into State. One
honest block beats one silent lie.

---

## Anti-patterns

| Anti-pattern | Why it rots the loop | Do instead |
|---|---|---|
| Unbounded retry | Hangs forever, spends silently on an unfixable input | Finite `max_attempts`, then escalate |
| Identical re-call | Same input → same failure; pure waste | Change input/env/strategy each attempt |
| Hand-edit the artifact to pass | Hides the bug; it ships again next iteration | Fix the tool/template/skill (class c) |
| Retry against production | Double-posts, double-charges, double-deploys | Retry in dry-run; live action fires once |
| Escalate every bump | Not autonomous; you're babysitting again | Widen standing authorization |
| Fabricate the missing input | Corrupts State; every future iteration inherits the lie | Escalate honestly or stop — never invent |
| Vague escalation ("it's broken") | Human re-debugs from zero; round-trips | Crisp ASK · CONTEXT · TRIED payload |
| Burn the whole budget on a hard blocker | Wastes attempts on something retry can't solve | Class (d) escalates immediately |

---

## Next

- See how retry & escalation map onto the CEO philosophies → [`./05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md)
- Steal a loop shape that already wires these up → [`./06-agent-loop-patterns.md`](./06-agent-loop-patterns.md)
- Revisit the gate that triggers all of this → [`./03-verification-and-state.md`](./03-verification-and-state.md)
- Design ⑦ and ⑧ for your own loop → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- Worked failure handling in practice → [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md)
