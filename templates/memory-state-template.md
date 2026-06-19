# Memory & State — {{LOOP_NAME}}

> **Part-⑥ companion to the blueprint.** This is the worksheet for the State part of
> [`./loop-blueprint.md`](./loop-blueprint.md). Use it to design what your loop
> *remembers* after the window is erased — the memory force from
> [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md).
> State comes in **layers** with different durability and different jobs; the cardinal
> rule is **learnings go somewhere durable that changes future behavior**, never a
> scratch dir that gets cleaned. Copy this file, replace every `{{…}}`, delete layers
> you don't use.

---

## 0 · What this loop must remember

- **Minimum to persist:** {{the least that must survive so iteration N+1 doesn't repeat N's work or mistakes}}
- **Altitude:** {{micro | pipeline | project (session over session)}}
- **The compounding promise:** because of what N learns, N+1 will {{do X differently}}.

---

## 1 · State layers

Map each layer this loop uses. The lifeline (DB/ledger) and reflections are almost
always durable; scratch is whatever a `rm -rf build/` may safely erase.

| Layer | Store | Key fields / contents | When read | When written | Durable? |
|---|---|---|---|---|---|
| **Ledger / DB** (lifeline) | {{sqlite/postgres/jsonl path}} | {{id · status · created_at · result_ref · …}} | every Trigger (load situation) | every pass (commit result) | **durable** |
| **Reflections** | {{`reflections/` dir / table}} | {{what worked · what failed · never-do-again}} | startup, into the brief | end of iteration (last step) | **durable** |
| **Handoff** | {{`HANDOFF.md`}} | {{where we left off · what's next · open threads}} | first thing next session | last thing this session | **durable** (rolling) |
| **Reports** | {{`reports/` / analytics table}} | {{metrics consumed then archived}} | startup | by scheduled job | semi (archived) |
| **Scratch** | {{`build/`, tmp dirs}} | {{intermediate artifacts, working files}} | within a run | within a run | **scratch — deletable** |

---

## 2 · Ledger / DB schema (the lifeline)

The source of truth for "what's the situation." Design the table before the loop runs.

```
TABLE {{items}}:
  id            {{pk}}
  status        {{enum: pending | in_progress | verified | failed | escalated}}
  created_at    {{timestamp}}
  updated_at    {{timestamp}}
  input_ref     {{pointer to the input artifact, not the artifact inline}}
  result_ref    {{pointer to the verified output}}
  attempts      {{int — feeds Retry ⑦ bounding}}
  verify_log    {{which gate passed / failed, from part ⑤}}
  {{...}}        {{domain-specific columns}}
```

- **Source-of-truth query:** `{{the one query that answers "what's the state right now?"}}`
- **Overlap guard:** {{how `status = in_progress` prevents a second iteration grabbing the same item}}

---

## 3 · Durable vs scratch — draw the line

> **The single most common State bug is writing a learning into scratch.** It gets
> cleaned away and the loop relearns the same lesson forever. If it must change future
> behavior, it is durable — put it where `rm -rf build/` cannot reach.

- **Durable (survives forever):** {{DB · `reflections/` · `HANDOFF.md` · promoted rules}}
- **Scratch (safe to delete each run):** {{`build/` · render temp · intermediate files · the window itself}}
- **Test for each item:** *does iteration N+1 need this to be smarter?* yes → durable, no → scratch.

---

## 4 · Read-back & summarization (how State re-enters via ① and ②)

State is useless until it flows back into the window. Design the path in:

```
⑥ STATE ──(at next ① TRIGGER)──▶ read back ──▶ summarize to slice ──▶ ② CONTEXT INJECTION
```

- **Trigger read-back (①):** {{the first move each iteration — e.g. "open DB, read HANDOFF.md, pull latest report"}}
- **Slice injected (②):** {{which rows/notes — the ones that matter NOW, not the whole DB}}
- **Summarization rule:** {{how the slice is compressed to window size — counts/digests, not raw dumps}}
- **Kept OUT on purpose:** {{the bulk of State left as a queryable pointer, never resident}}
- **Anti-bloat guarantee:** {{why the injected slice stays bounded even as the DB grows without limit}}

---

## 5 · Reflection writeback (the compounding engine)

Reflection is **always the last step**, and it must land where it **changes future
behavior** — not a log nobody reads.

- **What gets reflected:** {{the learning — pattern, failure cause, better approach}}
- **Where it's written (durable):** {{`reflections/<date>.md` · a rules table · promoted into a SKILL/template}}
- **How it re-enters:** {{folded into the next brief at ② · or promoted so a tool now enforces it}}
- **Behavior-change check:** because this is written, iteration N+1 will {{do X differently}} — if "nothing," the path is broken.

> A learning that doesn't re-enter Context Injection is a diary entry, not a loop
> improvement. The outer loop must rewrite the inner loop — see
> [`../docs/03-verification-and-state.md`](../docs/03-verification-and-state.md).

---

## 6 · Persistence hygiene

- **`.gitignore` the lifeline:** {{the DB lives in `.gitignore`}} — it is runtime state, not source; committing it means one bad run corrupts shared truth.
- **Back it up:** {{snapshot/backup cadence for the DB}}.
- **Reconcile / `doctor`:** {{the consistency check (part ⑤, project altitude) that confirms the ledger still adds up before a new run}}.
- **What's safe to wipe:** {{scratch dirs cleaned between runs — confirm no durable layer lives inside them}}.

---

## 7 · State-design closure check

- [ ] Every layer in §1 has a store, a read-time, a write-time, and a durable/scratch verdict.
- [ ] The **lifeline DB** is `.gitignore`d, backed up, and `doctor`-checkable.
- [ ] **No durable layer lives inside a scratch dir** (no learning in `build/`).
- [ ] Read-back at ① and the summarized slice at ② are both specified; the slice is **bounded**.
- [ ] A **reflection writeback** path exists and a named behavior changes in N+1.
- [ ] Only **verified** results (part ⑤ pass) commit to durable State.

> When this holds, your loop has memory that survives the window — pair it with the
> gate design in [`./verification-checklist.md`](./verification-checklist.md) and see
> it wired up end-to-end in
> [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md),
> [`../examples/content-production-loop.md`](../examples/content-production-loop.md),
> [`../examples/customer-acquisition-loop.md`](../examples/customer-acquisition-loop.md).
