---
name: {{skill-name}}
description: {{One sentence on the capability this skill wraps — what it does and the deterministic result it guarantees. Then list example trigger phrases so a role knows when to reach for it. Triggers - "{{do the frozen action, e.g. publish the post}}", "{{run the X check}}", "{{deploy/charge/send via Y}}", "{{the recurring mechanical step this freezes}}".}}
version: 0.1.0
license: MIT
---

# {{Skill Name}} — the part-④ companion to one loop

> **What this skill is.** A frozen **Tool** (loop part ④) — one recurring mechanical
> capability, described once so a role *uses* it instead of re-deriving it in fragile
> prose every iteration. It pairs with a loop you designed in
> [`./loop-blueprint.md`](./loop-blueprint.md): the blueprint says *that* this tool
> exists; **this file is where the how-to-use detail lives, so the constitution stays
> lean.** Detail belongs here, not in `CLAUDE.md`.
>
> **What this skill is NOT.** Not the loop, not a role, not business logic in prose. It
> does **not** decide *whether* to act (the role decides); it deterministically *executes*
> when invoked. It wraps MCP/CLI that are **executed, never loaded into context**.

---

## Which loop part it serves

- **Primary:** ④ **Tools** — freezes the recurring step `{{name the step}}` into a
  zero-token, repeatable capability so it behaves identically every iteration.
- **Verification hook for ⑤:** every invocation returns a **deterministic signal**
  (exit code / parseable status) the loop's Verification gate consumes — see
  *Verification hook* below. A tool that can't be checked can't close a loop.
- **Loop it belongs to:** `{{loop name}}` → [`./loop-blueprint.md`](./loop-blueprint.md)

> **Seeing ≠ should-use.** A role may see many skills. The owning sub-agent's system
> prompt must **narrow** to this one for `{{the step}}`. State that pointer in the SP, not here.

## Built from

> The capability *wraps* these — they run as commands, their output never bloats context.

| Layer | What it wraps | Invocation | Dry-run available? |
|---|---|---|---|
| MCP server | `{{server name / tool}}` | `{{how it's called}}` | {{yes/no — test mode}} |
| CLI command | `{{cli subcommand}}` | `{{exact command}}` | {{--dry-run flag}} |
| CLI command | `{{e.g. ./factory.py publish}}` | `{{…}}` | {{…}} |

- **Global vs local:** {{global reuse (shared across projects) | local (built/forked for this project)}}.
- **Scope note:** {{if local — a skill scoped to another dir won't auto-load here; fork or call the global equivalent}}.

## Usage

**A role invokes it like this:**

```
{{exact invocation a sub-agent issues — command + args, or the skill call}}
```

- **Inputs:** {{what the role must supply — item id, file path, payload}}
- **Outputs:** {{the artifact + the machine-readable status the loop reads}}
- **Side effects:** {{what changes in the world — file written, post published, row updated}}
- **Idempotency:** {{safe to run twice? what guards against double-action — see Queue Worker pattern}}

## Dry-run / test mode

> **No real publish / deploy / charge as a test.** Every tool that touches the world must
> have a way to prove itself without side effects. This is how Retry (⑦) and the first
> run stay safe.

```
{{the dry-run / sandbox invocation — e.g. ./factory.py publish --dry-run <item>}}
```

- **What dry-run proves:** {{validates inputs + auth + shape, returns the would-be result, touches nothing real}}
- **How to tell dry-run from live:** {{flag / env / target — make it impossible to confuse}}

## Verification hook (ties to ⑤)

> **The deterministic check that proves it worked.** The loop's Verification gate calls
> this; it inspects the **artifact**, not the agent's claim.

- **Check:** `{{the command that exits non-zero on failure — e.g. curl the published URL, query the DB row, re-read the file}}`
- **Pass criteria:** {{the explicit bar — HTTP 200 + expected body | row exists with status=done | file matches schema}}
- **On pass → ** loop commits to State (⑥). **On fail → ** loop's Retry (⑦) feeds this check's output back as the complaint.
- **Self-test:** {{how a maintainer confirms the skill itself still works — a smoke command}}

## Examples

**Example 1 — happy path**
```
{{invocation}}
→ {{output + verification result}}
```

**Example 2 — dry-run before going live**
```
{{dry-run invocation}}
→ {{would-be result, nothing published}}
```

**Example 3 — failure the verification hook catches**
```
{{invocation that produces a bad result}}
→ {{non-zero exit / status the gate rejects → loop retries}}
```

---

> **Keep it lean and pointer-driven.** Like every SKILL.md, this stays small: *how the
> capability is used* and *how it's verified*, nothing more. Push deeper mechanics into
> the CLI/MCP it wraps; push *when to use it* into the role's system prompt; push *that it
> exists* into [`./loop-blueprint.md`](./loop-blueprint.md). For the role that owns this
> tool and the gate it feeds, see [`../docs/02-loop-anatomy.md`](../docs/02-loop-anatomy.md)
> (parts ④ and ⑤) and the worked uses in
> [`../examples/coding-agent-loop.md`](../examples/coding-agent-loop.md).
