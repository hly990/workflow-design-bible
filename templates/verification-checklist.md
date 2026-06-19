# Verification Checklist — {{LOOP_NAME}}

> **Part-⑤ companion to the blueprint.** This is the gate-design worksheet for the
> Verification part of [`./loop-blueprint.md`](./loop-blueprint.md). Use it to *design*
> the gates a loop's output must pass and to *run* them each iteration. The governing
> idea, from [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md):
> a gate only counts if the agent **cannot talk its way past it.** Build the cheapest
> deterministic check first; climb the ladder only when a command can't express the
> check. Copy this file, replace every `{{…}}`, delete rungs you don't use.

---

## 0 · What this loop verifies

- **Artifact under test:** {{the file / URL / DB rows / binary this iteration produces}}
- **Altitude:** {{micro (one file/call) | pipeline (one item end-to-end) | project (State consistency)}}
- **Most common failure to catch:** {{the bad result you see most often — gate this FIRST}}
- **On pass → ** commit to State (⑥). **On fail → ** Retry (⑦).

---

## 1 · Gate inventory

List every gate this loop runs, cheapest (rung 1) at the top. The agent passes only if
**all** required gates pass.

| Gate | What it inspects (artifact, not claim) | Who runs it | Pass criteria | On-fail action |
|---|---|---|---|---|
| {{deterministic cmd}} | {{file/URL/DB it reads}} | {{command / CI}} | {{exit 0}} | {{→ Retry: repair-and-retry}} |
| {{self-check vs rubric}} | {{checklist items}} | {{actor role}} | {{all boxes true}} | {{→ revise, then re-run rung 1}} |
| {{independent verifier}} | {{the real output}} | {{separate verifier role}} | {{verdict = ship}} | {{→ Retry with verifier's complaint}} |
| {{human sign-off}} | {{the call only a human can make}} | {{owner}} | {{explicit approval}} | {{→ park; Escalation}} |

---

## 2 · The deterministic-first ladder

Fill top-down. Stop climbing once the rungs in play actually catch your failures.

- **Rung 1 — Deterministic command** (build this first): `{{cmd that exits non-zero on failure}}`
  - Catches: {{the common failure from §0}}
  - Why it's unfakeable: {{machine-judged, no agent narration involved}}
- **Rung 2 — Self-check vs rubric** (cheap, weak): {{checklist the actor re-reads its output against}}
  - Known blind spot: {{what the actor will miss because it's grading itself}}
- **Rung 3 — Independent verifier** (adversarial): role `{{verifier}}`, no stake, judges by {{reading the artifact directly}}
  - Independence guarantee: {{different role + inspects artifact, not the worker's summary}}
- **Rung 4 — Human sign-off** (rationed): only for {{subjective / irreversible / high-stakes}}

> **Cheapest-first rule:** if a `{{…}}` on a lower rung is blank because "a command
> can't check this," prove it — most "subjective" checks have a deterministic core
> (length, schema, presence, status code) you can gate for free before spending a
> verifier call or a human's attention.

---

## 3 · The unfakeable test (run this on every gate)

For each gate, answer one question. If **any** answer is "yes," it is **not a gate** —
fix it before shipping the loop.

- [ ] Could the agent write "✅ done" and move on **without this check actually passing**?
- [ ] Does the gate read the agent's **claim/summary** instead of the **real artifact**?
- [ ] Is the **actor** also the **judge** (no independent role, no command)?
- [ ] Can the check be **skipped, mocked, or self-graded** without anyone noticing?

> **If yes anywhere → climb a rung.** Replace a self-report with a command; replace a
> summary-grader with an artifact-reader; replace the actor-as-judge with a separate
> verifier role. A gate the agent can fake is a suggestion, not a gate.

---

## 4 · Sign-off matrix — machine vs human

Decide, per result type, what may pass on machine judgment alone and what needs a human.

| Result type | Machine-only OK? | Needs human sign-off? | Why |
|---|---|---|---|
| {{routine artifact, reversible}} | ✅ | — | {{deterministic gate suffices}} |
| {{subjective / brand / tone call}} | — | ✅ | {{value judgment the owner must own}} |
| {{irreversible / spends money / goes live}} | — | ✅ | {{high-stakes, hard to undo}} |
| {{State-consistency check}} | ✅ (`doctor`) | — | {{ledger reconciles by command}} |

> Ration rung 4. A loop that asks for sign-off on routine, reversible work isn't
> autonomous — push those down to rung 1. See Escalation in [`./loop-blueprint.md`](./loop-blueprint.md).

---

## 5 · Per-gate fill-in block (copy once per gate)

```
GATE: {{name}}
  rung:            {{1 deterministic | 2 self-check | 3 verifier | 4 human}}
  inspects:        {{the artifact itself — file path / URL / DB query / binary}}
  run by:          {{command invocation | verifier role + pointer | human channel}}
  pass criteria:   {{exit 0 | rubric all-true | verifier verdict | approval}}
  unfakeable?:     {{yes/no — if no, why it still can't be talked past}}
  on fail:         {{→ Retry strategy ⑦ | → Escalation ⑧}}
  feeds back:      {{the specific complaint handed to Retry so the fix is targeted}}
```

---

## 6 · Loop-closure verification (the gate around the gates)

Before you call the loop's Verification done, confirm it actually closes the loop:

- [ ] Every gate inspects the **artifact**, not the agent's self-report.
- [ ] The **cheapest deterministic** gate exists and runs **first**.
- [ ] At least one gate is **unfakeable** (rung 1 or an independent rung 3).
- [ ] **Pass → State (⑥):** exactly what commits is named, and only passing results commit.
- [ ] **Fail → Retry (⑦):** the verifier's complaint is fed back, not an identical re-run.
- [ ] Retry is **bounded**; on exhaustion it **escalates (⑧)**, not loops forever.
- [ ] Human sign-off is **rationed** to subjective / irreversible / high-stakes only.
- [ ] Verification runs at the **right altitude** (micro→file, pipeline→item, project→State).

> When this holds, your gate can say *no* — which is the only reason a loop compounds
> quality instead of slop. Pair it with the State design in
> [`./memory-state-template.md`](./memory-state-template.md) and three worked gates in
> [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md),
> [`../examples/content-production-loop.md`](../examples/content-production-loop.md),
> [`../examples/customer-acquisition-loop.md`](../examples/customer-acquisition-loop.md).
