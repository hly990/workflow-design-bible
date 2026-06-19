# Worked Example — Coding Agent Loop

> **The scenario.** A repo has a backlog of tickets — bugs, small features, chores —
> sitting in an issue tracker. You want an agent that *works the backlog unattended*:
> picks the next ticket, implements the change, and only calls it done when the test
> suite, linter, and typechecker all pass — then opens a merge-ready PR and reflects.
> It produces **green, review-ready PRs while you sleep**, not piles of half-finished
> branches. This is the canonical shape of two patterns from
> [`../docs/06-agent-loop-patterns.md`](../docs/06-agent-loop-patterns.md):
> **Queue Worker** (pull the next item off a backlog, one at a time) wrapping a
> **Produce-Verify-Retry** core (make the change, run the gate, repair on red).

This file is a *filled-in* [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md):
the eight canonical parts, answered concretely for one real loop.

```
                  ┌──────────────────────── re-arm: next ticket ───────────────────────┐
                  │                                                                     │
   ① next OPEN ticket ─▶ ② inject ticket + acceptance + only-relevant files ─▶ ③ coding
      (label-filtered)          criteria + CLAUDE.md pointer                     worker
                  ▲                                                                │
                  │                                                                ▼
                  │                                              ④ edit · test · lint · typecheck · git · gh
                  │                                                                │
                  │                                                                ▼
   ⑧ ESCALATE ◀── ⑦ RETRY ◀──[RED: gate exits ≠0]── ⑤ VERIFICATION ──[GREEN]──▶ ⑥ STATE
   (ask: spec /     (repair w/ failure        the deterministic gate:           (tracker: done ·
    creds /          output fed back,         tests+lint+types exit 0           git history ·
    budget out)      bounded ×3)              + diff vs acceptance              reflections · HANDOFF)
                                                      │
                                                  open PR ──▶ ready for human review/merge
```

---

## 0 · Loop identity

- **Name:** `coding-agent-loop`
- **One-sentence purpose:** keep turning open tickets into merge-ready PRs, gated on a green test/lint/type suite, unattended.
- **Altitude:** pipeline (one ticket end-to-end) nested inside a project loop (the backlog, session over session).
- **One iteration =** one ticket → one PR.
- **Cadence / lifetime:** runs continuously while the backlog is non-empty; idles when the queue is drained.

---

## ① Trigger — what starts an iteration

- **First trigger:** a human "go" — you point the orchestrator at the repo and the label filter, and authorize it to start.
- **Recurring trigger:** completion of the prior ticket re-arms the loop; it pulls the **next open ticket** off the tracker automatically.
- **Work-list source:** the issue tracker, filtered to a label (e.g. `agent-ready` AND not `blocked`), ordered by priority then age.
- **Overlap guard:** **one ticket in flight per worktree.** A ticket is moved to `in-progress` (and its worktree locked) before work starts; a second worker cannot claim it. Concurrency comes from *more worktrees*, never from two workers on one ticket.
- **Idle behavior:** queue empty → post a one-line "backlog drained" status and sleep until a new labelled ticket appears (event) or the next scheduled sweep.

> **You are not the trigger.** The first "go" is yours; after that, *completion* arms
> the next iteration. If the loop only moves when you personally assign the next ticket,
> you have a prompt you keep re-sending, not a loop.

## ② Context Injection — what the iteration knows

- **Task brief (per iteration):** the ticket title + body + its **acceptance criteria**, restated as the definition of done for this run.
- **State slice injected:** the ticket's status row, any prior attempt notes on *this* ticket, and the relevant entry from `reflections/` if a matching failure pattern exists.
- **Pointers (load on demand):** `CLAUDE.md` (coding conventions, how to run the suite), and the **relevant files/symbols only** — found by search, opened when touched.
- **Kept OUT on purpose:** the rest of the repo. Do **not** paste the tree, unrelated modules, or the full git log.
- **Anti-bloat rule:** load files on demand via search (grep the symbol, open the file, close the loop) — the window holds the ticket and the few files in play, nothing more. Context stays the size of *this* change, not the size of the repo.

## ③ Agent Role — who runs the iteration

- **Role name + one-line job:** `orchestrator` (CEO) — owns the backlog, dispatches one `coding-worker` per ticket, verifies, and talks to the human.
- **Orchestrator or worker:** the orchestrator *dispatches*; each **`coding-worker`** is a self-contained worker that takes one ticket brief and returns a branch + PR.
- **Concurrent siblings:** multiple `coding-worker`s run in parallel, **each in its own git worktree** — independent tickets fan out for free; an optional `reviewer` role can run after the gate.
- **System prompt / pointer:** `.claude/agents/coding-worker.md` (brief carries full context: ticket, acceptance, conventions pointer — no back-and-forth needed).

> **One worker, one ticket, one worktree.** Resist the do-everything agent that holds
> the whole backlog in one window. Isolation per worktree is what makes the workers both
> parallelizable *and* independently verifiable.

## ④ Tools — what the iteration can do

| Tool / capability | Frozen (CLI/MCP) or live? | Dry-run mode | Notes |
|---|---|---|---|
| editor / file ops | live (the agent decides edits) | edits land in a **worktree branch**, never `main` | the only "creative" tool |
| run test suite | deterministic CLI (`make test` / `pytest`) | runs against the branch; read-only on the world | the gate's core |
| linter | deterministic CLI (`ruff` / `eslint`) | branch-local | exits non-zero on violations |
| typechecker | deterministic CLI (`mypy` / `tsc --noEmit`) | branch-local | exits non-zero on type errors |
| git | deterministic CLI | commit/branch in worktree; **never push `main`** | one worktree per ticket |
| open PR | deterministic CLI (`gh pr create`) | draft PR = the safe default | target a review branch, not a deploy |

- **Tools the role is told NOT to reach for:** anything that pushes to `main`, force-pushes, deploys, or touches prod/CI secrets. Seeing `git` ≠ permission to `git push --force`.
- **Steps still decided on the fly that should be frozen:** "how do I run the suite for this repo" belongs in `CLAUDE.md` / a `verify` CLI subcommand, not re-derived each ticket.

> **Dry-run = work in a branch/worktree.** The whole iteration happens on an isolated
> branch. There is no "test in prod" — opening a (draft) PR *is* the live action, and
> it is reversible. Pushing to `main` is never a way to test.

## ⑤ Verification — how you know it worked

This is the **load-bearing part** of the whole loop. A coding agent that self-reports
"done" is worthless; the gate must be something it *cannot talk past*.

- **Primary gate (cheapest deterministic):** the suite — `test && lint && typecheck` — **must exit zero**. One red check = not done. No exceptions, no "mostly passing."
- **What the gate inspects:** the **artifact** — it actually *runs* the test suite against the branch and reads the exit code. It does not read the agent's summary of the tests.
- **Who verifies:** the deterministic command first (machine-judged, unfakeable); then the worker does a **diff self-review against the acceptance criteria**; optionally an independent `reviewer` role reads the diff with no stake in it.
- **Pass criteria — do it only if ALL hold:**
  1. `test` exits 0 (and a new test covers the change, if the ticket is a bug/feature).
  2. `lint` exits 0.
  3. `typecheck` exits 0.
  4. the diff actually satisfies the acceptance criteria (self-review), with no unrelated changes.
- **On pass →** commit to State (⑥): open the PR, mark the ticket. **On fail →** Retry (⑦).

> **Verify the artifact, not the claim.** "All tests pass" is a hypothesis until the
> runner exits 0 in front of you. The agent **cannot mark a ticket done without a green
> gate** — that single rule is what turns this loop from a slop generator into a
> compounding contributor. Full gate design: [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md).

## ⑥ State — what survives between iterations

- **Durable store(s):**
  - the **issue tracker** — the ledger of what's `done` / `in-progress` / `open` (source of truth for "what's the situation").
  - the **repo + git history** — the durable record of *what was actually changed*; every merged PR is permanent state.
- **Reflections:** `reflections/coding.md` — recurring failure patterns (flaky test, missing fixture, a convention the worker keeps violating). The compounding layer.
- **Handoff:** `HANDOFF.md` — for any ticket still **in flight** at session end: its branch/worktree, what's done, what the gate last said, the next step.
- **Read-back at next trigger:** ① queries the tracker for the next open ticket; ② injects that ticket's row plus any matching reflection.
- **Scratch (safe to delete each run):** the worktree, build artifacts, `__pycache__`, coverage reports — recreated per ticket.

> **State makes N+1 smarter than N.** The tracker stops the loop re-claiming a finished
> ticket; `reflections/` stops it re-making the same mistake on ticket 2, 20, and 200.

## ⑦ Retry — what happens on failure

- **Strategy:** **repair-and-retry** — feed the *exact failure output* (failing test name, lint rule, type error) back into the worker and fix that specific defect. Not an identical re-run.
- **Max attempts before escalating:** **3** on the same ticket. Bounded, always.
- **What changes each attempt:** the verifier's complaint — the new red output — is injected each time, so attempt 2 knows precisely why attempt 1 failed.
- **Flaky test:** if a test fails non-deterministically, **re-run with backoff** (2–3 times); only count a stable red as a real failure.
- **Recurring-failure rule:** if the **same** failure shows up *across tickets* (the harness can't find fixtures, a shared config is wrong, a flake recurs), **fix the shared cause** — the test harness / config / `CLAUDE.md` — **not the one diff**. Never hand-edit a single output to fake a green gate.

## ⑧ Escalation — when a human gets pulled in

- **Legitimate reasons to stop and ask (and only these):**
  1. **Ambiguous or contradictory spec** — the ticket's acceptance criteria conflict or are underspecified; this is a value/direction call the owner must make.
  2. **Missing access / credential** — a CI secret, a private package token, or repo permission the agent doesn't hold.
  3. **Retry budget exhausted** — 3 repair attempts and the gate is still red; escalate **with the failing output attached**.
- **Channel + payload:** a comment on the ticket / a DM — a **crisp ask + context + what was tried** (the diffs, the last red gate, the attempts).
- **While escalated:** **park this ticket** (label `blocked`, drop the worktree lock) and let other workers continue other tickets. Never block the whole backlog on one stuck item.
- **Standing authorization:** branch, edit, run the gate, commit, and open **draft PRs** freely, on any `agent-ready` ticket, without asking. **Never** without explicit sign-off: merging to `main`, force-pushing, deploying, or anything irreversible.

> **Sharp line, both ways.** Too eager and it pings you on every ticket; too reluctant
> and it *invents* a fake credential or hallucinates an approval. Three reasons fire an
> escalation — everything else, the loop solves itself. Detail:
> [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md).

---

## Why this loop closes

Trace it once and the holes are gone. The **green gate (⑤)** is load-bearing: nothing
reaches State without `test && lint && typecheck` exiting zero, so the loop compounds
*working* code instead of plausible-looking diffs. **State (⑥)** — the tracker plus git
history plus `reflections/` — means iteration N+1 never re-claims a finished ticket and
never re-makes a logged mistake. **Retry (⑦)** is bounded at three repair attempts with
the failure fed back each time, so a red test gets fixed, not abandoned, and never hangs
forever. **Escalation (⑧)** has exactly three sharp triggers, so the agent works
unattended yet stops cold at an ambiguous spec, a missing secret, or an exhausted budget —
and never fabricates past a real blocker. Eight parts, all answered, pass → State /
fail → Retry → Escalation. The loop runs the backlog while you review PRs.

## See also

- The blank to fill yourself → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- The two hard parts in depth → [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md)
- The patterns this loop instantiates → [`../docs/06-agent-loop-patterns.md`](../docs/06-agent-loop-patterns.md)
- Sibling worked examples → [`./content-production-loop.md`](./content-production-loop.md) · [`./customer-acquisition-loop.md`](./customer-acquisition-loop.md)
