# Worked Example — The Content Production Loop

> **The scenario.** An autonomous content channel — say a daily short-form video or
> article channel — that runs itself, unattended, on a cadence. Each iteration it
> **picks a topic, produces the piece** (script → render/draft), **passes a quality +
> compliance gate, publishes, then reflects.** One iteration = one published piece. It
> wakes once a day, forever, and the only time it interrupts you is when it genuinely
> must. This is the worked, fully-filled version of the
> [`loop-blueprint.md`](../templates/loop-blueprint.md) — every `{{…}}` answered.

**Patterns used** (from [`../docs/06-agent-loop-patterns.md`](../docs/06-agent-loop-patterns.md)):
a **Pipeline** as the spine (topic → produce → gate → publish), **Fan-out/Fan-in** at
the production step (many scenes/assets rendered in parallel, joined into one piece),
and **Reflect-and-Improve** as the outer loop that rewrites the inner one.

```
              ┌───────────────────── once per piece, on a cadence ─────────────────────┐
              │                                                                         │
   ① cron ─▶ ② inject topic+brand ─▶ ③ CEO dispatch ─▶ ④ produce ──▶ ⑤ GATE            │
   (daily)      (dedup vs ledger)     topic-picker      ┌──────────┐    quality+        │
                                      scriptwriter      │ scene 1  │    compliance      │
                                      renderer  ───────▶│ scene 2  │──▶ (judge)         │
                                      publisher         │ scene N  │     │              │
                                                        └ fan-out ─┘     │              │
                                                         (fan-in)        │              │
                              ┌──────[fail]──────────────────────────────┤              │
                              ▼                                       [pass]            │
                        ⑦ RETRY (repair scene / revise to rubric)        ▼              │
                              │                                    ⑥ STATE: publish,    │
                        ⑧ ESCALATE (brand call · missing cred ·     ledger++, reflect ──┘
                           irreversible publish)                   (outer loop)
```

---

## ① Trigger — what starts an iteration

- **First trigger:** a human **"go"** — you green-light the channel once, set the slot
  (e.g. 09:00 daily) and seed the topic queue. After that you never poke it again.
- **Recurring trigger:** a **scheduled cron** fires once per cadence and pulls the next
  topic off the **queue**. Empty pipeline → completion of one piece arms the next slot.
- **Work-list source:** a `topic_queue` (human-seeded + topic-picker-replenished from
  trends). One slot = one piece.
- **Overlap guard:** a **slot lock** — an iteration claims today's slot row before it
  starts; a second wake on the same slot sees the lock and no-ops. Two iterations can
  never race to publish the same slot.
- **Idle behavior:** queue empty + no slot due → log "nothing scheduled" and sleep. It
  idles cheaply; it does not spin generating filler to stay busy.

## ② Context Injection — what the iteration knows

- **Task brief (per iteration):** "Produce one piece on **`<topic>`** for the
  `<channel>` slot at `<time>`, to the brand voice, passing the gate."
- **State slice injected:** the **chosen topic**, the **brand/style slice** (voice,
  format, taboo list) and the last few reflections — *not* the back catalog.
- **Pointers (load on demand):** `configuration.json` (brand + platform config), the
  **quality rubric**, the publish runbook. Loaded only when a worker needs them.
- **Dedup check:** before committing the topic, query the **content ledger** — has this
  (or a near-duplicate) shipped before? If yes, skip it and pick again. Never repeat.
- **Kept OUT on purpose:** every past script, every rendered asset, the full ledger.
- **Anti-bloat rule:** inject the **topic and the brand slice**, not the whole channel
  history. The ledger is a pointer you *query*, never a blob you *paste*. Context stays
  the same size on piece #500 as on piece #1.

## ③ Agent Role — who runs the iteration

- **Orchestrator:** `CEO` — dispatches role-clear workers, verifies the artifact, owns
  the publish decision and the human conversation. It does no production itself.
- **Workers (each one job, full brief, no back-and-forth):**

| Role | One-line job | Concurrency |
|---|---|---|
| `topic-picker` | choose + dedup the next topic | sequential (gates the rest) |
| `scriptwriter` | topic → script/outline to brand voice | sequential |
| `renderer` | script → rendered piece — **fans out**: many scenes/assets in parallel | **parallel** |
| `publisher` | push the green-lit artifact to the platform | sequential (after gate) |
| `dev-maintainer` | one shared role: fix the skills/templates when they break | cross-cutting |

- **System prompt / pointer:** each lives at `.claude/agents/<role>.md`; `dev-maintainer`
  is the shared global maintainer, not a per-piece spawn.

> **Resist proliferation.** One `renderer` that fans out across scenes beats N bespoke
> per-scene roles. The fan-out is *free parallelism* on one role, not a zoo of roles.

## ④ Tools — what the iteration can do

| Tool / capability | Frozen or live? | Dry-run mode | Notes |
|---|---|---|---|
| `gen` image/video/draft generator | MCP (global skill) | render to a local file | the creative engine; one skill wraps it |
| `compile` render/compile CLI | deterministic CLI | `--out preview/` only | joins scenes → one artifact, repeatable |
| `publish` platform API | live (side-effecting) | **render to `preview/`, never publish as a test** | the one irreversible tool |
| `ledger` query/append CLI | deterministic CLI | read-only by default | dedup + performance record |

- **Tools the role is told NOT to reach for:** workers never call `publish` directly —
  only the CEO does, and only after a green gate. Seeing the API ≠ being allowed to fire it.
- **Steps still decided on the fly that should be frozen:** thumbnail crop, caption
  formatting, hashtag assembly → freeze into `compile` subcommands, not re-derived prose.

> **Dry-run is sacred here.** A "test publish" that actually posts is not a test — it is
> a mistake your audience sees. The test mode renders to `preview/` and stops.

## ⑤ Verification — how you know it actually worked

This is the **load-bearing** part of this loop. A content channel with a weak gate does
not fail loudly — it compounds slop into your brand, piece by piece. **Two gates, both
mandatory, no publish without both green:**

- **QUALITY gate** — the artifact is scored against the **rubric** (hook, clarity, pacing,
  payoff). A `scriptwriter`-independent **judge role** watches/reads the *rendered* piece
  and scores it. Self-check first; independent judge for the publish decision.
- **COMPLIANCE / brand-safety gate** — the **#1 iron law**: no slop, no platform-ToS
  violation, no brand taboo (from the taboo list in `configuration.json`). This gate is
  adversarial and non-negotiable.

- **What the gate inspects:** the **actual rendered artifact** — the judge *watches the
  video / reads the draft* — not the worker's "looks good." A claim is not evidence.
- **Pass criteria:** quality ≥ rubric bar **AND** zero compliance flags. Both, or no publish.
- **On pass →** commit to State (⑥) and publish. **On fail →** Retry (⑦).

> **No green gate, no publish.** The publish tool is physically downstream of a passing
> gate. The full gate-design list lives in
> [`../templates/verification-checklist.md`](../templates/verification-checklist.md).

## ⑥ State — what survives between iterations

- **Content ledger (DB):** topics shipped, publish timestamps, and **performance**
  (views/engagement pulled back later). The source of truth for "what have we done."
- **Dedup store:** topic fingerprints — queried by ② so the channel never repeats itself.
- **`reflections/`:** what underperformed and *why*, written permanently (a hook that
  flopped, a format that lagged). The compounding layer — never in a build dir.
- **`HANDOFF.md`:** the cross-session note — slot state, queue depth, open escalations —
  so a fresh session resumes exactly where the last one stopped.
- **`reports/`:** async analytics dropped between runs; consumed at next startup to
  update performance, then archived.
- **Scratch (safe to delete each run):** `preview/`, render temp, intermediate scenes.

> **The ledger is what makes piece N+1 smarter than N.** Dedup stops repeats;
> reflections stop re-flopping the same way. Design it with
> [`../templates/memory-state-template.md`](../templates/memory-state-template.md).

## ⑦ Retry — what happens on failure

- **Failed render / one bad scene →** **repair-and-retry that scene only**, not the whole
  batch. The fan-in waits on the one repaired asset; the other N stay as rendered. Bounded.
- **Failed quality gate →** **revise against the rubric's specific complaint** ("weak hook
  at 0:03") and re-score. Feed the complaint back; never re-call identically.
- **Max attempts:** **2 scene-repairs**, **2 rubric-revisions**, then escalate. Finite, picked.
- **Recurring-failure rule:** if the *same* defect recurs across pieces — every render
  clips the same way, a whole format keeps failing compliance — **fix the
  template/skill** (hand it to `dev-maintainer`), not this one piece. Never hand-edit a
  single output to fake a green gate.

## ⑧ Escalation — when a human gets pulled in

- **Legitimate reasons to stop and ask (and only these):**
  1. **Brand / compliance judgment call** — a topic in a sensitive area, or a piece the
     gate flags as a *judgment* (not a clear rule) call.
  2. **Missing platform credential** — expired token, no account, no permission to post.
  3. **Irreversible + off-brand risk** — anything about to **publicly publish** something
     hard to undo that the gate couldn't fully clear → human sign-off before it goes live.
- **Channel + payload:** a crisp ask + the rendered preview + what the gate flagged + what
  Retry already tried. The human approves, edits, or kills.
- **While escalated:** **park this slot and continue** — don't block the channel; the next
  scheduled topic can proceed while one piece waits on a human.
- **Standing authorization:** *pick, produce, gate, and publish autonomously every day;
  stop only for the three reasons above.* Everything else, the loop handles itself.

---

## Why this loop closes

- **Verification is load-bearing and doubled.** A **quality** gate *and* a
  **compliance/brand-safety** gate, both judging the *rendered artifact*, both required
  before publish. Nothing reaches the audience without a green light it cannot fake — the
  difference between a channel that builds a brand and one that compounds slop.
- **State makes it smarter, not just busier.** The **ledger + dedup store** guarantee it
  never repeats a topic; `reflections/` and `reports/` feed real performance back so
  piece N+1 learns from N.
- **Retry is bounded and surgical.** One bad scene is repaired alone (fan-out keeps the
  rest); a gate complaint drives a targeted revision; a *recurring* failure fixes the
  template, not the piece. Two strikes, then a human.
- **Escalation is sharp.** Three reasons, all of them real — a brand judgment, a missing
  credential, or an irreversible public publish. Otherwise it runs unattended.

And the **Reflect-and-Improve** step is the *outer* loop: each piece's reflection rewrites
the rubric, the brand slice, the topic strategy — so the loop that produces content also
**improves the loop that produces content.** That bridge is the whole point of
[`../docs/05-from-workflow-to-loop.md`](../docs/05-from-workflow-to-loop.md).

## See also

- Fill your own → [`../templates/loop-blueprint.md`](../templates/loop-blueprint.md)
- Design the gate → [`../templates/verification-checklist.md`](../templates/verification-checklist.md)
- The two hard parts → [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md)
- Sibling worked examples → [`./coding-agent-loop.md`](./coding-agent-loop.md) ·
  [`./customer-acquisition-loop.md`](./customer-acquisition-loop.md)
