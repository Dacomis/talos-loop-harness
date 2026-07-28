# talos-loop-harness

Talos, the bronze automaton, walked the same circuit of Crete three times a
day, unattended, until he was stopped.

A minimal, **strictly sequential** loop for working a plan to completion
unattended. One plan item at a time, implemented by a subagent, attacked by an
independent red cell, ruled on by an independent judge, then committed by the
main session.

The previous version of this harness ran plan items in parallel over git
worktrees and needed ~28k lines of JavaScript to make that safe. It still
failed: agents errored, and items reported "implemented all at once"
implemented nothing. v2 abandons parallelism. What is left is markdown.

## Philosophy

- **Sequential.** One item at a time, clean tree between items. No worktrees,
  no lanes, no merge gate, no requeue.
- **Adversarially verified.** The implementer never grades itself. The red cell
  sees only the item text and the diff — never the implementer's account of it.
  The judge always runs, even when the red cell passes.
- **The main session drives.** All plan edits and all commits are made by the
  driving session, never by a subagent. Subagents return text; the driver acts.
- **State is on disk, not in context.** `plan.md` + `decisions.md` + git
  history are the entire state. `/clear` and re-invoke is a free resume.

## Files

| Path | What it is |
|---|---|
| `skills/talos/SKILL.md` | The driver procedure. This is the harness. |
| `skills/talos/FORMAT.md` | `plan.md` / `decisions.md` format spec. |
| `skills/talos/prompts/implementer.md` | Implementer subagent prompt. |
| `skills/talos/prompts/red-cell.md` | Red-cell subagent prompt. |
| `skills/talos/prompts/judge.md` | Judge subagent prompt. |
| `skills/talos-mini/SKILL.md` | Ad-hoc variant: distil a conversation's task list into `mini-plan.md`, then run the same loop. |

## Install into a consumer repo

```bash
mkdir -p <repo>/.claude/skills
cp -r skills/talos      <repo>/.claude/skills/talos
cp -r skills/talos-mini <repo>/.claude/skills/talos-mini
```

Then, in that repo, write a `plan.md` (see `skills/talos/FORMAT.md`) — normally
in an ordinary session with a strong model, since a badly written plan is the
one failure the loop cannot recover from. Commit it, and run `/talos`.

`decisions.md` is created by the loop on first need. For ad-hoc task lists
agreed in conversation, run `/talos-mini` instead — no plan authoring session
required.

## Reading the output in the morning

- `plan.md` — `[x]` done, `[ ]` not reached, `[!]` blocked.
- `decisions.md` — every judge override, plan edit, and block, with reasoning.
- `git log --oneline` — one `feat:`/`fix:` commit per item, each optionally
  followed by its `refactor:` simplification commit.
