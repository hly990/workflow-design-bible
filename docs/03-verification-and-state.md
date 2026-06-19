# 03 · Verification & State — the two forces that hold a loop together

> **The two load-bearing parts.** [`02-loop-anatomy.md`](./02-loop-anatomy.md) calls
> Verification (⑤) and State (⑥) "the two forces," and means it literally. Every
> other part leans on these two. **Verification is the quality force** — the only part
> that can say *no*, the thing that turns repetition into improvement instead of decay.
> **State is the memory force** — the only part that survives the window, the thing
> that turns "ran a thousand times" into "learned a thousand times." Get these two
> right and the loop forgives weakness everywhere else. Get either wrong and the loop
> rots no matter how elegant the other six parts are. This document is how you get
> them right.

---

## Why these two are load-bearing

A loop is a wheel that turns on its own. Two forces keep it from flying apart:

```
        no quality force                 no memory force
   ┌────────────────────────┐      ┌────────────────────────┐
   │ every turn looks fine   │      │ every turn starts at 0  │
   │ and is slightly worse   │      │ re-decides settled      │
   │ → slop compounds        │      │ → Groundhog Day         │
   └────────────────────────┘      └────────────────────────┘
        ⑤ Verification stops this    ⑥ State stops this
```

- **Verification gates the wheel.** Without it, nothing ever says *no*, so quality
  sags one acceptable-looking step at a time. By the time a human notices, State is
  full of accepted garbage and you cannot tell which rows are real.
- **State carries the wheel forward.** Without it, the window is erased and iteration
  N+1 is identical to N — an amnesiac repeating itself, never compounding.

They are coupled, not independent: **Verification decides what is allowed to enter
State, and State is what the next Verification checks against.** A pass writes to
State; a fail never does. Design them together or you will design neither.

---

## Part ⑤ — Verification: the gate the agent cannot talk past

The whole job of Verification is to be **unfakeable**: a gate the agent must not be
able to argue, narrate, or assert its way through. "The agent said it's done" is the
null gate — it gates nothing, because the thing being judged is also the judge.

### The gate ladder — cheapest first

Build gates from the bottom up. Reach for the expensive ones only when the cheap ones
cannot express the check.

| Rung | Gate | How it judges | Cost | Fakeable? |
|---|---|---|---|---|
| 1 | **Deterministic command** | exits non-zero on failure (tests, linter, schema validator, `doctor`, link/`curl` check) | ~free, repeatable | **No** — machine-judged |
| 2 | **Self-check vs rubric** | the role re-reads its own output against an explicit checklist | cheap | Weak — same agent, same blind spots |
| 3 | **Independent verifier** | a *different* role, no stake, judges the artifact adversarially | a sub-agent call | Hard — different window, different incentive |
| 4 | **Human sign-off** | the owner looks and approves | slow, scarce | No, but rationed |

> **The unfakeable-gate principle.** A gate only counts if the agent **cannot claim
> success without it actually passing.** If the worker can write "✅ done" and move on
> while the check is skipped, mocked, or self-graded, you don't have a gate — you have
> a suggestion. The test: *could a lazy or confused agent get past this without the
> work being real?* If yes, climb a rung.

### Design the cheapest deterministic check first

Do not start at the top of the ladder. Start at the bottom and ask: **what is the
single cheapest command that catches the most common failure?** Build that one first;
it will catch the majority of bad iterations for almost nothing.

```
most failures ──▶ caught by rung 1 (a command)         ← build FIRST
the subtle ones ─▶ caught by rung 3 (a verifier role)   ← add when rung 1 misses
the irreversible ─▶ caught by rung 4 (a human)          ← reserve, ration
```

A linter that exits non-zero on the failure you keep seeing is worth more than an
elaborate verifier prompt, because it never gets tired, never rationalizes, and costs
no tokens. Freeze the common case into a command; spend judgment only on what a
command cannot express.

### Verifier independence — verify the artifact, not the claim

When you must reach rung 3, the verifier earns its keep only if it is **independent**:

- **A different role.** The actor verifying itself (rung 2) shares the actor's blind
  spots and its motive to call the work done. Spawn a separate verifier role with one
  job — find what's wrong — and **no stake** in the answer being yes.
- **It inspects the artifact, not the report.** The verifier must *read the file*,
  *`curl` the URL*, *query the DB*, *run the binary* — touch the real output. A
  verifier that grades the worker's summary of its own work is rung 2 wearing a costume.

> **The CEO rule, restated:** *the CEO verifies before reporting — read it, curl it,
> query the DB; never trust a sub-agent's "done."* Independence is what makes a
> verifier adversarial instead of collusive.

### Verification across altitudes

The gate ladder is fractal — it runs at every altitude, just with different artifacts:

| Altitude | What's verified | Typical gate |
|---|---|---|
| **Micro** (one task) | one file / one rendered scene / one API response | rung 1: the file parses, the frame isn't black, the call returned 200 |
| **Pipeline** (one item end-to-end) | the whole piece is publishable | rung 1 + rung 3: deterministic checks *then* an adversarial verifier |
| **Project** (session over session) | the system is still consistent and safe to run again | rung 1 `doctor` consistency-lint over State + rung 4 sign-off on direction |

A project loop's verification is mostly **checking State for consistency** — the
`doctor` check that the ledger still reconciles — not re-judging individual artifacts.
Verify at the altitude the failure lives at.

---

## Part ⑥ — State: what survives the window

State is everything the loop remembers after the window is erased. It comes in
**layers**, each with a different durability and a different job. Confuse the layers
and you will either lose what matters or drown in what doesn't.

### The four state layers

| Layer | What it is | Durable? | Read when | Written when |
|---|---|---|---|---|
| **Ledger / DB** | the queryable record of what exists and what's done — *the lifeline* | **durable** (forever) | every Trigger, to load the situation | every pass, to commit the result |
| **Reflections** | permanent learnings — what worked, what didn't, what to never do again | **durable** (forever) | startup, folded into the brief | end of iteration, the compounding step |
| **Handoff** | the concise "where we left off / what's next" note bridging sessions | **durable** (rolling) | first thing next session | last thing this session |
| **Reports** | async analytics the next iteration consumes at startup, then archives | semi (archived) | startup | by a separate scheduled job |

> **Durable vs scratch — the line that saves you.** *Durable* state survives forever
> and must never live in a build dir that gets cleaned: the DB, `reflections/`,
> `HANDOFF.md`. *Scratch* state is everything safe to delete each run: temp render
> dirs, intermediate files, the working window itself. **The single most common State
> bug is writing a learning into scratch** — the reflection gets cleaned away and the
> loop relearns the same lesson forever. If it must change future behavior, it is
> durable. Put it somewhere a `rm -rf build/` cannot reach.

### Read-back & summarization — how State re-enters the loop

State is useless until it flows back in. It re-enters through the front of the loop:

```
⑥ STATE ──(at next ① TRIGGER)──▶ read back ──▶ summarize to window size ──▶ ② CONTEXT INJECTION
```

- **Trigger reads it back.** The recurring Trigger's first move is to load the
  situation: open the ledger, read `HANDOFF.md`, pull yesterday's report.
- **Context Injection summarizes it down.** You do *not* inject the whole DB. You
  inject the **slice that matters now** — the rows relevant to this iteration,
  reflections relevant to this task — summarized to fit a budget, not a warehouse.
  This is the discipline that keeps long loops from bloating: State can grow without
  bound *on disk*; the window slice never does.

The design question is always: *what is the minimum slice of State that makes this
iteration correct?* Inject that. Leave the rest as a pointer the role can query.

### The compounding engine — reflection writes back into State

This is the mechanism that makes a loop *improve* instead of merely *repeat*:

```
   iteration N ──▶ ⑤ verify ──[pass]──▶ ⑥ commit result
                                          │
                                          ▼
                              write REFLECTION (durable)
                                          │
   iteration N+1 ◀── ② inject reflection ◀┘   ← N+1 is now smarter than N
```

Reflection is **always the last step**, and it must write somewhere that **changes
future behavior** — not a log nobody reads, but `reflections/` that gets folded into
the next brief, or a rule that gets promoted into the skill/template itself. A
learning that doesn't re-enter Context Injection is a diary entry, not a loop
improvement. This is the outer loop rewriting the inner loop: the reason "design the
loop" subsumes "design the workflow" (see [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md)).

> **The compounding test.** Ask of every loop: *because of what iteration N learned,
> what will iteration N+1 do differently?* If the answer is "nothing," your reflection
> path is broken — the learning isn't reaching State, or State isn't reaching the
> window. Fix that before anything else; it is the whole point of running a loop more
> than once.

---

## Anti-patterns — the ways these two forces quietly fail

| Anti-pattern | Why it rots the loop | The fix |
|---|---|---|
| **"The agent said done" as verification** | the judge is the actor; gates nothing | climb to rung 1 — a command that exits non-zero |
| **Verifier grades the claim, not the artifact** | rung 2 in disguise; never touches reality | make it read the file / curl / query the DB |
| **Verifier shares the actor's stake** | collusion, not adversarial check | spawn a separate role with no skin in the output |
| **Learnings logged where nothing reads them** | no behavior change; reflection is a diary | write to `reflections/`, fold into the next brief |
| **Reflection saved in a build/scratch dir** | cleaned away; the lesson is relearned forever | durable store only — survive `rm -rf build/` |
| **Whole DB injected every iteration** | unbounded context growth, rising cost, drift | inject the summarized *slice*; keep the rest a pointer |
| **Lifeline DB committed / not backed up** | one bad run corrupts the source of truth | gitignore it; back it up; reconcile with `doctor` |
| **Verifying at the wrong altitude** | re-judging artifacts when State is what drifted | micro→file, pipeline→item, project→State consistency |

---

## Next

- Companion gate designer (part ⑤) → [`../templates/verification-checklist.md`](../templates/verification-checklist.md)
- Companion State designer (part ⑥) → [`../templates/memory-state-template.md`](../templates/memory-state-template.md)
- When the gate says no → [`04-retry-and-escalation.md`](./04-retry-and-escalation.md)
- The eight parts in full → [`02-loop-anatomy.md`](./02-loop-anatomy.md)
- Reflection as the outer loop → [`05-from-workflow-to-loop.md`](./05-from-workflow-to-loop.md)
- Design the whole loop → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)

> **One rule to carry out of this page:** a loop improves only when a gate it cannot
> fake decides what enters a memory that survives the window. Verification without
> State catches mistakes and forgets them; State without Verification remembers
> garbage. You need both, designed together — they are the two forces.
