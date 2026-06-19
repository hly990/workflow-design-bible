# Worked Example — Customer-Acquisition Loop

> **The scenario.** You point an agent at a lead list and walk away. New leads land
> from your forms / CRM / scraped source; the loop **researches and qualifies** each
> one against your ICP, **personalizes** an outbound message, runs it through a
> **compliance + quality gate**, **sends**, and **watches for replies**. The moment a
> lead replies *warm*, the loop stops touching it and hands a qualified, warmed lead
> to a human to close. What it produces is **unattended top-of-funnel**: a steady
> stream of researched, consented, on-brand first-touches and a clean queue of warm
> replies for a salesperson — without you babysitting a sequence or risking your
> domain on a spam complaint.

This loop nests two patterns from
[`../docs/06-agent-loop-patterns.md`](../docs/06-agent-loop-patterns.md): a
**Queue Worker** (3) draining the lead list one record at a time, wrapped around an
**Escalation-Gated / Human-in-the-Loop checkpoint** (8) — because the single most
valuable event in the funnel, a warm reply, is a business judgment that **never**
gets auto-closed. Compliance is the iron law; the human owns the close.

```
   new lead ─▶│rate/overlap guard│
                     │
   ② inject this lead's slice ─▶ ③ researcher/qualifier ─▶ ICP fit?
                     │                                        │
                  not ICP ─▶ mark, skip                    fit ─▶ personalizer
                                                              │
                                              ④ render message to review queue
                                                              │
              ┌────────── ⑤ COMPLIANCE gate (iron law) ──────┘
              │   consent? suppression? rate cap? ToS?
            FAIL ─▶ hard stop / suppress           PASS
              │                                       │
        (no send, ever)                    ⑤ QUALITY gate (on-brand?)
                                            FAIL ─▶ ⑦ revise ─▶ re-gate
                                                       │
                                                     PASS ─▶ send ─▶ ⑥ CRM ledger
                                                                        │
                                                  reply-watcher ◀───────┘
                                                       │
                                          warm reply ─▶ ⑧ HAND TO HUMAN (close)
```

---

## ① Trigger — what starts an iteration

- **First trigger:** a human "go" — you approve the lead source, the ICP, and the
  daily send cap *once*, then the loop runs unattended within those bounds.
- **Recurring trigger:** **event** (a new lead lands in the source list / CRM) *or* a
  **daily batch** popped off the lead queue. One iteration = **one lead**.
- **Work-list source:** the CRM view `status = new` (Queue Worker pop), oldest first.
- **Overlap / rate guard:** claim the lead atomically (`new → in-flight`) so two
  iterations never grab the same record; a **send-cap counter** in State blocks the
  iteration the moment today's cap is hit. **You may never double-contact a lead or
  exceed the cap** — both are guarded in State, not in prose.
- **Idle behavior:** queue empty or cap reached → idle cheaply, re-arm at next window.

## ② Context Injection — what the iteration knows

- **Task brief (per iteration):** "Qualify and, if a fit, draft one compliant,
  personalized first-touch for **this** lead."
- **State slice injected:** the **one lead record** + its **prior-contact history**
  (so you never re-contact wrongly) + its **consent/suppression status**. Nothing else
  from the CRM.
- **Pointers (load on demand):** `documentation/icp.md` (qualification criteria),
  `documentation/brand-voice.md` (messaging), `documentation/compliance.md` (the
  consent / anti-spam / ToS rules). Loaded only when that step needs them.
- **Kept OUT on purpose:** the rest of the CRM, other leads' threads, the full send log.
- **Anti-bloat rule:** inject **this lead's slice, not the warehouse**. The window holds
  one record; everything shared (ICP, voice, rules) is a pointer, so context is flat
  across 10 or 10,000 leads.

## ③ Agent Role — who runs the iteration

- **Role + job:** `acquisition-CEO` — orchestrator; dispatches workers, owns the gate
  and the human hand-off. Decides, never sends on its own claim.
- **Workers it dispatches:** `researcher/qualifier` (enrich + ICP-score),
  `personalizer/copywriter` (draft on-brand), `sender` (deliver), `reply-watcher`
  (poll threads, classify replies). Each gets a self-contained brief.
- **Concurrent siblings:** researcher and reply-watcher can run in parallel across
  different leads; one **shared maintainer** role edits skills/templates/rubrics.
- **System prompt / pointer:** `.claude/agents/acquisition-*.md`.

## ④ Tools — what the iteration can do

| Tool / capability | Frozen or live? | Dry-run mode | Notes |
|---|---|---|---|
| enrichment / research | MCP + live reasoning | cached lookup, read-only | wraps `skills/enrich.md` |
| CRM read/write | deterministic CLI | `--read-only` query | source of truth for status & consent |
| compliance check | **deterministic CLI** | always safe (read-only) | exits non-zero = blocked |
| send channel (email/DM) | API, **live & irreversible** | **draft + render to review queue** | NEVER sends as a test |

- **Tools the role is told NOT to reach for:** no bulk-send, no list import, no CRM
  field overwrite outside the status/consent columns. Narrow surface on purpose.
- **Steps to freeze:** suppression-list lookup and rate-cap accounting are mechanical —
  freeze them into CLI subcommands so they behave identically every iteration.

> **Dry-run never means "send a test email."** The dry-run of an outbound loop is
> **render the message to a review queue** — the exact bytes that *would* ship, plus the
> lead's consent status — so a human or a gate can inspect reality without a real send
> hitting a real inbox. A "test send" is a real send. There is no such thing as a
> harmless one.

## ⑤ Verification — how you know it worked

This loop's gate is **load-bearing on two axes**, and it inspects **artifacts**
(the rendered message + the lead's consent record), never the worker's "looks good."

- **(a) COMPLIANCE gate — the iron law (deterministic).** A command that checks, for
  this exact send: **consent / opt-in present**, lead **not on the suppression list**,
  **rate cap** not exceeded, channel **ToS / anti-spam** satisfied. It **exits
  non-zero** on any failure and that **blocks the send, full stop**. Machine-judged,
  unfakeable.
- **(b) QUALITY gate — independent judge.** A separate `judge` role scores the rendered
  message against a rubric: genuinely **personalized** (not mail-merge), **on-brand**,
  **not slop**, clear ask. The producer never grades itself.
- **Pass criteria:** **green on BOTH gates.** No send without compliance *and* quality
  passing. Compliance is necessary; quality is necessary; neither is sufficient alone.
- **On pass →** send, then commit to State (⑥). **On fail →** Retry (⑦) or hard-stop.

> **Verify the artifact, not the claim.** The gate reads the *actual rendered message*
> and *queries the CRM for this lead's consent status* — it does not trust the
> personalizer saying "this is consented and on-brand." A worker that could talk its way
> past the compliance gate is a worker that will eventually mail your suppression list.

Full gate design list: [`../templates/verification-checklist.md`](../templates/verification-checklist.md).

## ⑥ State — what survives between iterations

- **Durable store(s):**
  - **CRM / lead ledger** — `status` per lead (`new → in-flight → contacted → replied →
    handed-off / disqualified`), `last_contact_at`, `attempts`.
  - **Suppression / do-not-contact list** — durable, **authoritative**. The one list the
    compliance gate consults; a lead here is never contacted again, ever.
- **Reflections:** `reflections/` — what messaging actually converts, which ICP signals
  predict warm replies. Feeds back as new copy/qualification defaults.
- **Handoff:** `HANDOFF.md` — in-flight sequences, today's cap usage, leads parked for a
  human, so the next session resumes without re-contacting or re-counting.
- **Read-back at next trigger:** ① pulls the `new` queue + today's send count; ②
  summarizes the single lead's slice.
- **Reports:** `reports/` — funnel analytics (sent → reply → warm rate), consumed at
  startup, then archived.

> Design it with [`../templates/memory-state-template.md`](../templates/memory-state-template.md).

## ⑦ Retry — what happens on failure

- **Strategy by failure type:**
  - **Transient send failure** (5xx, timeout) → **re-attempt with backoff**.
  - **Bounced / invalid address** → **mark and skip** (degrade — don't hammer a dead
    inbox; flag the record for cleanup).
  - **Failed quality gate** → **repair-and-retry**: feed the judge's specific objection
    back to the personalizer, revise, re-gate.
  - **Failed compliance gate** → **not a retry**. No-consent / suppressed / over-cap is a
    **hard stop**, not something you re-attempt your way past.
- **Max attempts before escalating:** quality revisions bounded (e.g. 2); transient sends
  bounded (e.g. 3 w/ backoff). Pick the number; never unbounded.
- **Recurring-failure rule:** if **deliverability keeps failing** (mass bounces, spam
  folder) or every draft fails the same rubric line, **fix the sending setup / template /
  rubric — not the one message.** Patch the cause via the maintainer; never hand-edit one
  output to fake a pass.

More: [`../docs/04-retry-and-escalation.md`](../docs/04-retry-and-escalation.md).

## ⑧ Escalation — when a human gets pulled in

This is **the defining part of this loop.** The whole point of the funnel is to deliver
warm leads *to a person*, so escalation here is a planned, first-class output — not a
last resort.

- **Legitimate reasons to stop and ask (and only these):**
  1. **A positive / warm reply — ALWAYS.** A warm lead is a subjective, money-on-the-table
     business judgment. The loop **never** auto-replies to close, auto-negotiates, or
     auto-books. It hands the thread to a human with full context. This is the loop's
     primary success path, not an error.
  2. **Compliance ambiguity** — "is this consent actually valid for this channel?" When
     the gate can't say *yes* deterministically, the answer is **ask**, never **assume**.
  3. **Missing channel credential / authorization** — no send key, no send.
  4. **Suppression-list or cap breach attempt** — anything that *would* contact a
     suppressed lead or exceed the cap is a **hard stop**, surfaced to the human.
- **Channel + payload:** the warm thread + lead research + "tried N touches, here's the
  reply, recommend you take it from here" → the human's queue.
- **While escalated:** **park this lead, keep draining the queue** for others. One warm
  reply does not block the funnel.
- **Standing authorization:** research, qualify, draft, gate, and send first-touches to
  **consented, non-suppressed, under-cap** leads — without asking. Everything past a warm
  reply, and every consent doubt, stops.

> **Never fabricate your way past the gate.** The one unforgivable failure here is the
> agent inventing consent, faking a qualification, or hallucinating an opt-in to clear the
> compliance check. The gate is the iron law precisely because the cost of bypassing it —
> a spam complaint, a blacklisted domain, a legal exposure — is irreversible. When in
> doubt about consent, the loop **stops**. It does not improvise.

---

## Why this loop closes

Pass → State (lead marked `contacted`, learnings to `reflections/`), fail → bounded
Retry → Escalation. Four things make it safe to run unattended:

- **Compliance Verification is the iron law.** A deterministic gate that exits non-zero
  blocks the send — and it reads the *artifact* (rendered message + consent record), not
  the worker's claim. No green gate, no send, no exceptions.
- **The suppression list is authoritative State.** One durable do-not-contact source of
  truth that every send consults — so the loop physically cannot mail someone who opted
  out, even across thousands of iterations.
- **Retry is bounded and degrades.** Transient → backoff, bounce → skip, slop → revise,
  recurring → fix the template. A bad inbox or a flaky API never hangs the funnel or
  hammers a dead address.
- **Escalation is sharp and human-owned.** A warm reply *always* goes to a person; a
  consent doubt *always* stops. The agent runs top-of-funnel forever and never pretends to
  own the close.

## See also

- The blueprint this fills in → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- Harden the two-axis gate → [`../templates/verification-checklist.md`](../templates/verification-checklist.md)
- Bound failure, sharpen the hand-off → [`../docs/04-retry-and-escalation.md`](../docs/04-retry-and-escalation.md)
- Sibling examples → [`./coding-agent-loop.md`](./coding-agent-loop.md) · [`./content-production-loop.md`](./content-production-loop.md)
