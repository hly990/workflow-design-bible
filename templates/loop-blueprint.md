# Loop Blueprint — {{LOOP_NAME}}

> **Fill this in to design one loop.** It is the eight parts of
> [`../docs/02-loop-anatomy.md`](../docs/02-loop-anatomy.md) with blanks. Copy this
> file, replace every `{{…}}`, delete what doesn't apply. When all eight parts are
> answered and the loop *closes* (pass → State, fail → Retry → Escalation), you have a
> design you can build. Keep each answer to a few lines — push detail into a pointer
> (`documentation/<x>.md`, a `SKILL.md`, or a sub-agent system prompt).
>
> Worked examples: [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md) ·
> [`../examples/content-production-loop.md`](../examples/content-production-loop.md) ·
> [`../examples/customer-acquisition-loop.md`](../examples/customer-acquisition-loop.md)

---

## 0 · Loop identity

- **Name:** `{{LOOP_NAME}}`
- **One-sentence purpose:** {{what this loop keeps producing, and why}}
- **Altitude:** {{micro (one task) | pipeline (one item end-to-end) | project (whole thing, session over session)}}
- **One iteration = ** {{one item? one batch? one session?}}
- **Cadence / lifetime:** {{how often it runs · how long it's meant to keep running}}

---

## ① Trigger — what starts an iteration
- **First trigger:** {{what kicks off iteration #1 — usually a human "go"}}
- **Recurring trigger:** {{schedule | event | completion-of-prior | self-re-arm | queue}}
- **Work-list source:** {{where the next item comes from}}
- **Overlap guard:** {{what prevents a new iteration starting while one is in flight}}
- **Idle behavior:** {{what it does when there's nothing to do}}

## ② Context Injection — what the iteration knows
- **Task brief (per iteration):** {{the self-contained instruction for one run}}
- **State slice injected:** {{which rows/notes from ⑥, summarized to window size}}
- **Pointers (load on demand):** {{documentation/<x>.md · config · prior artifact}}
- **Kept OUT on purpose:** {{what you deliberately do NOT inject, to avoid bloat}}
- **Anti-bloat rule:** {{how context stays bounded across many iterations}}

## ③ Agent Role — who runs the iteration
- **Role name + one-line job:** `{{role}}` — {{its single responsibility}}
- **Orchestrator or worker:** {{CEO-style dispatcher | self-contained worker}}
- **Concurrent siblings:** {{which roles run in parallel with this one, if any}}
- **System prompt / pointer:** {{`.claude/agents/<name>.md` or "shared global role X"}}

## ④ Tools — what the iteration can do
| Tool / capability | Frozen (CLI/MCP) or live? | Dry-run mode | Notes |
|---|---|---|---|
| {{tool}} | {{deterministic CLI · MCP · live reasoning}} | {{how to test safely}} | {{which skill wraps it}} |
| … | | | |
- **Tools the role is told NOT to reach for:** {{narrow the surface; seeing ≠ should-use}}
- **Steps still decided on the fly that should be frozen:** {{→ candidates for a new skill/CLI cmd}}

## ⑤ Verification — how you know it worked
- **Primary gate (cheapest deterministic):** {{the command that exits non-zero on failure}}
- **What the gate inspects:** {{the artifact itself — file read / curl / DB query — not the agent's claim}}
- **Who verifies:** {{deterministic command | separate verifier role | human sign-off}}
- **Pass criteria:** {{the explicit bar — do it only if ALL pass}}
- **On pass → ** commit to State (⑥). **On fail → ** Retry (⑦).
> Use the full gate design list: [`verification-checklist.md`](./verification-checklist.md)

## ⑥ State — what survives between iterations
- **Durable store(s):** {{DB/ledger name + the few key fields}}
- **Reflections:** {{where learnings are written permanently}}
- **Handoff:** {{what the cross-session note must contain}}
- **Read-back at next trigger:** {{how ① pulls State in and ② summarizes it}}
- **Scratch (safe to delete each run):** {{build dirs / temp artifacts}}
> Design it with: [`memory-state-template.md`](./memory-state-template.md)

## ⑦ Retry — what happens on failure
- **Strategy:** {{re-attempt w/ backoff | repair-and-retry | degrade | fix-the-cause}}
- **Max attempts before escalating:** {{a finite number — pick it}}
- **What changes each attempt:** {{the verifier's complaint fed back · not an identical re-call}}
- **Recurring-failure rule:** {{when the same failure repeats → fix the code/template/skill, not the artifact}}

## ⑧ Escalation — when a human gets pulled in
- **Legitimate reasons to stop and ask (and only these):**
  1. {{missing credential / authorization}}
  2. {{subjective / business / value judgment}}
  3. {{retry budget exhausted · irreversible high-stakes action}}
- **Channel + payload:** {{how the human is reached + "crisp ask · context · what I tried"}}
- **While escalated:** {{block this item | park it and continue others | degrade}}
- **Standing authorization:** {{the explicit "do all of this without asking" boundary}}

---

## Loop closure check (must hold before you build)
- [ ] Every one of the eight parts above is answered (no blank `{{…}}` that matters).
- [ ] The loop **closes**: pass → State, fail → Retry → (bounded) → Escalation.
- [ ] Verification inspects the **artifact**, not the agent's self-report.
- [ ] Context does **not** grow unboundedly across iterations.
- [ ] Retry is **bounded** and Escalation has **few, sharp** reasons to fire.
- [ ] State makes iteration N+1 smarter than N (something compounds).
- [ ] A reflection/learning path writes back into State (the loop improves the loop).

> When this holds, register the loop in your project constitution (`CLAUDE.md`) as a
> row in the pipeline spine, then build its tools as skills. See
> [`../docs/05-from-workflow-to-loop.md`](../docs/05-from-workflow-to-loop.md).
